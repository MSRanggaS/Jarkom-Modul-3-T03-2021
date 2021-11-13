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


### 11. Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie.


### 12. Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps. 


### 13. Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya.



## Kendala
