# Jarkom-Modul-3-T03-2021

## Soal dan Penyelesaian

### 1. Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server


### 2. Dan Foosha sebagai DHCP Relay. 


### 3. Luffy dan Zoro menyusun peta tersebut dengan hati-hati dan teliti. Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu: Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169.


### 4. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50.


### 5. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.


### 6. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit. 

    
### 7. Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69. 

**Jawab:**
Kita tinggal melihat hwaddress dari skypie. Kemudian tinggal memasukkan command berikut ke `/etc/network/interfaces` pada node skypie tergantung nilai hwaddress masing masing
```
hwaddress ether 46:9b:96:38:f1:bc
```
Kemudian memasukkan konfigurasi ini ke file `/etc/dhcp/dhcpd.conf` pada node Jipangu
```
host Skypie {
    hardware ethernet 46:9b:96:38:f1:bc;
    fixed-address 10.43.3.69;
}
```

![image](https://media.discordapp.net/attachments/858956223604850688/908924169511649380/Screenshot_2021-11-13_103621.png)


### 8. Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000. 

**Jawab:**
Kita menginstal DNS di EniesLobby kemudian kita masukkan konfigurasi untuk proxy Water7.
Masukkan konfigurasi berikut pada file `/etc/bind/named.conf.local` pada node EniesLobby
```
zone "jualbelikapal.T03.com" {
        type master;
        file "/etc/bind/kaizoku/jualbelikapal.T03.com";
};
```

Kemudian konfigurasi berikut pada file `/etc/bind/kaizoku/jualbelikapal.T03.com`
```
$TTL    604800
@       IN      SOA     jualbelikapal.T03.com. root.jualbelikapal.T03.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      jualbelikapal.T03.com.
@       IN      A       10.43.2.3; 
```
Setelah itu restart bind9

Kemudian install squid di Water7 dan masukkan konfigurasi berikut pada file `/etc/squid/squid.conf`
```
http_port 8080
visible_hostname Water7
```
Kemudian restart squid, apabila belum bisa mengakses karena masih ada konfigurasi yang akan ditambahkan di soal berikutnya.

Untuk menjalankan proxy di client gunakan perintah
```
export http_proxy="http://jualbelikapal.T03.com:5000"
```
dan untuk mengecek apa proxy sudah berjalan gunakan perintah `env | grep -i proxy`

![image](https://media.discordapp.net/attachments/858956223604850688/908924186741846016/unknown.png)


### 9. Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy. 

**Jawab:**
Tinggal jalankan saja perintah berikut, parameter `c` adalah untuk membuat username password baru sehingga yang kedua tidak perlu parameter `c`, `b` untuk memasukkan parameter password bersamaan dengan username, dan `m` adalah untuk enkripsi MD5
```
htpasswd -cbm /etc/squid/passwd luffybelikapalt03 luffy_t03
```
```
htpasswd -bm /etc/squid/passwd zorobelikapalt03 zoro_t03
```
Kemudian tambahkan konfigurasi pada file `/etc/squid/squid.conf`
```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS AVAILABLE_WORKING
http_access deny all
```
![image](https://user-images.githubusercontent.com/61416036/141604613-47d30692-23f7-4245-8167-a99b13faeaab.png)
![image](https://user-images.githubusercontent.com/61416036/141604629-c4d6b0b2-baf1-4e64-aa43-801e5871942d.png)
Terjadi error karena waktu belum di set


### 10. Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00).

**Jawab:**
Tambahkan konfigurasi berikut pada file `/etc/squid/acl.conf` 
```
acl AVAILABLE_WORKING time MTWH 07:00-11:00
acl AVAILABLE_WORKING time TWHF 17:00-23:59
acl AVAILABLE_WORKING time WHFA 00:00-03:00
```
Dan juga tambahkan konfigurasi berikut pada file `/etc/squid/squid.conf`
```
include /etc/squid/acl.conf
```
![image](https://user-images.githubusercontent.com/61416036/141604638-cbc4ef99-07b0-4e06-a2c7-f4603d3bc159.png)
![image](https://user-images.githubusercontent.com/61416036/141604642-c14d6821-8e46-4ee3-b8bf-a6bf04a6350d.png)
Dapat mengakses dengan syarat syarat yang ada


### 11. Setiap mengakses `google.com`, akan diredirect menuju `super.franky.yyy.com`

1. Menambahkan peraturan proxy di `/etc/squid/squid.conf` pada Water7

```
acl myNetwork src 10.43.3.0/24 10.43.1.0/24
acl blksites dstdomain .google.com
deny_info http://super.franky.T03.com all

http_reply_access deny blksites all

http_access allow myNetwork AVAILABLE_WORKING
```

2. Restart squid `service squid restart`


3. `Lynx google.com` pada proxy client yaitu Loguetown

saat `lynx google.com` akan langsung ter-redirect ke `super.franky.T03.com`

![11-1](https://user-images.githubusercontent.com/73921231/141609505-87b38a4e-695b-4f49-be1a-e70d1e66ec46.jpg)

isi dari `super.franky.T03.com` sama dengan yang ada pada modul 2

![11-2](https://user-images.githubusercontent.com/73921231/141609519-a592a50b-2fbf-4000-bb2c-f3212c2f3044.jpg)

### 12. Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Bandwidth luffy dibatasi menjadi kecepatan 10 kbps.

1. Menambahkan peraturan proxy di `/etc/squid/squid.conf` pada Water7

```
acl user1 proxy_auth luffybelikapalT03
acl user2 proxy_auth zorobelikapalT03

acl download url_regex -i \.jpg234$ !\.jpg$ !\.png$
acl downloadd url_regex -i \.jpg$ \.png$

delay_pools 1
delay_class 1 1
delay_parameters 1 10000/50000
delay_access 1 allow user1

http_reply_access deny download user1
http_reply_access deny downloadd user2
```

2. Restart squid `service squid restart`

3. Test akses file dan download speed

**Luffy**

Jika mencoba mengakses file selain `.png` dan `.jpg`, file tersebut menjadi forbidden/access denied

![12-1](https://user-images.githubusercontent.com/73921231/141609671-e32f2d2c-429f-4b0f-98f3-d21145cdb78c.jpg)

Download speed luffy menjadi 10kbps saat mendownload file `.png` atau `.jpg`

![12-2](https://user-images.githubusercontent.com/73921231/141609789-80565006-9157-4470-a9c8-81e07dcf9c53.jpg)

**Zoro**

Jika mencoba mengakses file `.png` dan `.jpg`, file tersebut menjadi forbidden/access denied

![12-3](https://user-images.githubusercontent.com/73921231/141609850-cd5b0fe6-6a0e-41e8-b65d-64ce6c5f5c62.jpg)

### 13. Kecepatan Zoro tidak dibatasi

Mengakses file selain `.png` dan `.jpg`

![13](https://user-images.githubusercontent.com/73921231/141609975-1dcc076c-d4b1-4060-9f4a-749480cc1f52.jpg)

file tersebut akan terload secara instant.

hal itu membuktikan kecepatan zoro tidak dibatasi, tidak seperti luffy.


## Kendala
