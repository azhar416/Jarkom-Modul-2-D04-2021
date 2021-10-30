# Jarkom-Modul-2-D04-2021

1. Daffa Muhamad Azhar [05111940000037]
2. Taqarra Rayhan I [05111940000121]
3. Iwan Dwi Prakoso [05111940000229]


# Modul 2
## Nomor 1 
EniesLobby sebagai DNS Master, Water7 sebagai DNS Slave, Skypie sebagai webserver. Terdapat 2 Client, Loguetown dan Alabasta. Semua Node terhubung pada Foosha sehingga dapat mengakses internet.

## Jawab



pertama mengatur settingan di foosha agar bisa mengakses server. Untuk melakukan itu menjalankan command berikut 

```sh

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.23.0.0/16

cat /etc/resolv.conf
```
lalu akan muncul ip nameserver

kemudian menjalankan command berikut di loguetown dan alabasta

```sh
echo nameserver 192.168.122.1 > /etc/resolv.conf

```
agar bisa mengakses internet

## Nomor 2
Membuat website utama dengan mengakses **franky.d04.com** / **www.franky.d04.com** pada folder kaizoku.

## Jawab
untuk nomer 2 penyelesainnya adalah dengan mengatur konfigurasi pada `/etc/bind/named.conf.local` seperti berikut

```sh
zone "franky.d04.com" {
        type master;
        file "/etc/bind/kaizoku/franky.d04.com";
};

```
kemudian dibuat file direktori dengan nama kaizoku pada folder `/etc/bind/` dengan perintah `mkdir /etc/bind/kaizoku`
setelah itu mengatur file franky.d04.com dengan perintah seperti berikut 

```sh
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.d04.com. root.franky.d04.com. (
                              2         ; Serial 
                         604800         ; Refresh
                          86400         ; Retry 
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;   
@       IN      NS      franky.d04.com.
@       IN      A       10.23.2.4
www     IN      CNAME   franky.d04.com.
```

kemudian dapat di testing dengah ngeping franky.d04.com dan akan muncul gambar berikut

![jarkom-2](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/jarkom-2.JPG?raw=true )

## Nomor 3
Membuat subdomain **super.franky.d04.com** / **www.super.franky.d04.com** yang diatur DNS-nyadi EniesLobby dan mengarah ke Skypie.

## Jawab

mirip dengan nomer 2 kita dapat menambahkan beberapa konfigurasi terlebih terdahulu pada `/etc/bind/kaizoku/franky.d04.com`

```sh

echo '

ns1    IN    A    10.23.2.4
super    IN    NS    ns1

' >> /etc/bind/kaizoku/franky.d04.com


```

kemudian menambahkan super.franky.d04.com di `/etc/bind/named.conf.local`


```sh
echo 'zone "super.franky.d04.com"{
    type master;
    file "/etc/bind/kaizoku/super.franky.d04.com";
};' >> /etc/bind/named.conf.local

```
lalu menjalankan perintah berikut

```sh

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     super.franky.d04.com. root.super.franky.d04.com. (
                              2         ; Serial
                        604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      super.franky.d04.com.
@       IN      A       10.23.2.4
www     IN      CNAME   super.franky.d04.com.'
 > /etc/bind/kaizoku/super.franky.d04.com

```

dapat di test dengan ngeping super.franky.d04.com


![jarkom-3](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/jarkom-3.JPG?raw=true)


## Nomor 4
Membuat reverse domain pada domain utama.

## Jawab
untuk melalukan reverse domain diperlukan command berikut untuk djalankan

```sh

echo -e 'zone "2.23.10.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.23.10.in-addr.arpa";
};' >> /etc/bind/named.conf.local

```

lalu untuk `2.23.10.in-addr.arpa` dapat diatur seperti berikut 


```sh

echo  ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.d04.com. root.franky.d04.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
2.23.10.in-addr.arpa.   IN      NS      franky.d04.com.
2       IN      PTR franky.d04.com.' > /etc/bind/kaizoku/2.23.10.in-addr.arpa

```

kemudian dapat ditesting dengan host -t PTR 10.23.2.2 menjadi seperti berikut


![jarkom-4](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/jarkom-4.JPG?raw=true)

## Nomor 5
Membuat Water7 sebagai DNS Slave untuk domain utama.

## Jawab
pertama kita mengatur franky.d04.com seperti berikut

```sh

zone "franky.d04.com" {
        type master;
        notify yes;
        also-notify { 10.23.2.3; };
        allow-transfer { 10.23.2.3; };
        file "/etc/bind/kaizoku/franky.d04.com";
};



```


lalu mengatur dns slave pada water 7


```sh

echo -e '
	zone "franky.d04.com" {
		type slave;
		masters { 10.23.2.2; };
		file "/var/lib/bind/franky.d04.com";};
' > /etc/bind/named.conf.local
```

kemudian untuk ngetesting pada eniesloby menjalankan command `service bind9 stop` untuk mematikan eniesloby kemudian pada loguetown dapat ngeping franky.d04.com
 

## Nomor 6
Membuat subdomain **mecha.franky.d04.com** / **www.mecha.franky.d04.com** didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo.

## Jawab

pertama kita mengatur franky.do4.com terlebih dahulu dengan menambahkan line berikut 

```sh

echo  'ns1      IN      A       10.23.2.3
mecha   IN      NS      ns1' >> /etc/bind/kaizoku/franky.d04.com

```
lalu mengatur `/etc/bind/named.conf.options`


```sh

echo  'options {
        directory "/var/cache/bind";
        #dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;
        listen-on-v6 { any; };
        };' > /etc/bind/named.conf.options




echo  '
zone "franky.d04.com" {
        type master;
        notify yes;
        also-notify { 10.23.2.3; };
        allow-transfer { 10.23.2.3; };
        file "/etc/bind/kaizoku/franky.d04.com";
};

zone "2.23.10.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.23.10.in-addr.arpa";
};

zone "super.franky.d04.com"{
    type master;
    file "/etc/bind/kaizoku/super.franky.d04.com";
};

' > /etc/bind/named.conf.local
```
kemudian pada water7 dapat mengatur options seperti berikut 


```sh
echo -e 'options {\n\tdirectory "/var/cache/bind";\n\t#dnssec-validation auto;\nn
\tallow-query{any;};\n\n\tauth-nxdomain no;\n\tlisten-on-v6 { any; };\n};' > /ett
c/bind/named.conf.options

```

dapat di testing pada loguetown dengan ping

![jarkom-6](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/jarkom-6.JPG?raw=true)

## Nomor 7
Membuat subdomain melalui Water7 dengan nama **general.mecha.franky.d04.com** / **www.general.mecha.franky.d04.com** yg mengarah ke Skypie.

## Jawab
pertama kita mengatur zone baru sperti berikut

```sh
echo 'zone "general.mecha.franky.d04.com"{
    type master;
    file "/etc/bind/sunnygo/general.mecha.franky.d04.com";
};' >> /etc/bind/named.conf.local

```

lalu menambahkan command berikut pada mecha.franky.d04.com dan mengatur filenya juga

```sh
echo 'ns1    IN    A    10.23.2.4
general.mecha    IN    NS    ns1' >> /etc/bind/sunnygo/mecha.franky.d04.com

```

```sh


echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     general.mecha.franky.d04.com. root.general.mecha.franky..
d04.com. (
                              2         ; Serial
                        604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      general.mecha.franky.d04.com.
@       IN      A       10.23.2.4

```

kemudian dapat ditesting pada loguetoen

![jarkom-7](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/jarkom-7.JPG?raw=true)
## Nomor 8
Membuat webserver **www.franky.d04.com** membutuhkan webserver dengan DocumentRoot pada **/var/www/franky.d04.com**.

## Jawab
Untuk menyiapkan webserver, perlu diinstall beberapa aplikasi dengan cara berikut
```sh
apt-get install apache2 -y
apt-get install php -y  
apt-get install libapache2-mod-php7.0 -y
apt-get install wget -y
apt-get install unzip -y
```

Download file-file yang dibutuhkan pada soal-soal berikutnya dan extract.

```sh
wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/general.mecha.franky.zip
wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip
wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/franky.zip

unzip general.mecha.franky.zip
unzip super.franky.zip
unzip franky.zip
```

Membuat DocumentRoot dilakukan dengan cara memindahkan folder hasil extract dari franky.zip ke folder /var/www
```sh
mv /root/franky /var/www
mv /var/www/franky /var/www/franky.d04.com
```

Setelah DocumentRoot sudah siap, dibuatkan configurasi apache webserver dan dimasukkan konfigurasi DocumentRoot sesuai yang baru saja disiapkan. Lalu dilakukan aktivasi configurasi website yang baru saja dibuat dan apache direstart untuk mengaplikasikan perubahan.
Pada nomor-nomor berikutnya, setelah dilakukan perubahan pada configurasi, selalu dilakukan apache2 restart. Sehingga pada penjelasan langkah pada soal berikutnya tidak akan diulangi untuk penjelasan terkait.
```sh
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/franky.d04.com.conf

echo '<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    ServerName franky.d04.com
    ServerAlias www.franky.d04.com
    DocumentRoot /var/www/franky.d04.com

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/franky.d04.com.conf

a2ensite franky.d04.com.conf

service apache2 restart
```


![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/926.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/925.png)

## Nomor 9
URL **www.franky.d04.com/index.php/home** dapat menjadi **www.franky.d04.com/home**.

## Jawab
Untuk mengubah route agar url www.franky.d04.com/index.php/home dapat menjadi menjadi www.franky.yyy.com/home perlu dilakukan aktivasi module rewite.
```sh
a2enmod rewrite
service apache2 restart
```
Setelah itu ditambahkan konfigurasi dan rewrite rule pada file .htaccess serta merestart apache2  
```sh
echo 'RewriteEngine On
    RewriteRule ^home$ index.php/home' > /var/www/franky.d04.com/.htaccess

echo '<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    ServerName franky.d04.com
    ServerAlias www.franky.d04.com
    DocumentRoot /var/www/franky.d04.com

    <Directory /var/www/franky.d04.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/franky.d04.com.conf

service apache2 restart
```

![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/927.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/928.png)

## Nomor 10
Pada subdomain **www.super.franky.d04.com** dibutuhkan penyimpanan aset yang memiliki DocumentRoot pada **/var/www/super.franky.d04.com**.

## Jawab
Langkah nomer 10 ini dilakukan seperti pada nomor 8 hanya saja dengan domain dan path yang berbeda.
```sh
mv /root/super.franky /var/www
mv /var/www/super.franky /var/www/super.franky.d04.com

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/super.franky.d04.com.conf

echo '<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    ServerName super.franky.d04.com
    ServerAlias www.super.franky.d04.com
    DocumentRoot /var/www/super.franky.d04.com

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/super.franky.d04.com.conf

a2ensite super.franky.d04.com.conf

service apache2 restart
```

![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/929.png)

## Nomor 11
Pada folder **/public** hanya dapat melakukan listing.

## Jawab
Untuk merestrict akses pada folder public hanya dapat melakukan listing ditambahkan configurasi sebagai berikut
```sh
echo '<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    ServerName super.franky.d04.com
    ServerAlias www.super.franky.d04.com
    DocumentRoot /var/www/super.franky.d04.com

    <Directory /var/www/super.franky.d04.com/public>
        Options +Indexes
    </Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/super.franky.d04.com.conf

a2ensite super.franky.d04.com.conf

service apache2 restart
```
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/931.png)

## Nomor 12
Menyiapkan error file **404.html** pada folder **/error** untuk menggantu error code pada apache.

## Jawab
Untuk memberikan halaman error misal pada saat user mengakses url path yang tidak tersedia, ditambahkan configurasi ```ErrorDocument 404 /error/404.html```.
```sh
echo '<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    ServerName super.franky.d04.com
    ServerAlias www.super.franky.d04.com
    DocumentRoot /var/www/super.franky.d04.com

    ErrorDocument 404 /error/404.html

    <Directory /var/www/super.franky.d04.com/public>
        Options +Indexes
    </Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/super.franky.d04.com.conf

a2ensite super.franky.d04.com.conf

service apache2 restart
```
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/933.png)

## Nomor 13
Membuat konfigurasi virtual host untuk mengakses file aset **www.super.franky.d04.com/public/js** menjadi **www.super.franky.d04.com/js**.

## Jawab
Pada soal nomor ditambahkan alias untuk route ```"/js"``` akan mengarah ke path ```"/var/www/super.franky.d04.com/public/js"```
```sh
echo '<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    ServerName super.franky.d04.com
    ServerAlias www.super.franky.d04.com
    DocumentRoot /var/www/super.franky.d04.com

    ErrorDocument 404 /error/404.html
    Alias "/js" "/var/www/super.franky.d04.com/public/js"

    <Directory /var/www/super.franky.d04.com/public>
        Options +Indexes
    </Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/super.franky.d04.com.conf

a2ensite super.franky.d04.com.conf

service apache2 restart
```
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/934.png)

## Nomor 14
web **www.general.mecha.franky.d04.com** hanya dapat diakses dengan port 15000 dan port 15500.

## Jawab
Pada soal ini langkah-langkah pengerjaan dilakukan seperti nomor 8 dan nomor 10. Hanya saja pada konfigurasi baris pertama, port disetting menjadi 15000 dan 15500 dimana secara default adalah 80.
```sh
mv /root/general.mecha.franky /var/www
mv /var/www/general.mecha.franky /var/www/general.mecha.franky.d04.com

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/general.mecha.franky.d04.com.conf

echo '<VirtualHost *:15000 *:15500>
    ServerAdmin webmaster@localhost
    ServerName general.mecha.franky.d04.com
    ServerAlias www.general.mecha.franky.d04.com
    DocumentRoot /var/www/general.mecha.franky.d04.com

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/general.mecha.franky.d04.com.conf

echo 'Listen 15000
    Listen 15500' >> /etc/apache2/ports.conf

a2ensite general.mecha.franky.d04.com.conf

service apache2 restart
```
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/938.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/936.png)

## Nomor 15
Membuat autentikasi dengan `username: luffy` dan `password: onepiece` dan file di **/var/www/general.mecha.franky.d04.com**.

## Jawab
Untuk membuat authentikasi perlu ditambahkan configurasi seperti dibawah.
```sh
htpasswd -c /etc/apache2/.htpasswd luffy

echo '<VirtualHost *:15000 *:15500>
    ServerAdmin webmaster@localhost
    ServerName general.mecha.franky.d04.com
    ServerAlias www.general.mecha.franky.d04.com
    DocumentRoot /var/www/general.mecha.franky.d04.com
    
    <Directory /var/www/general.mecha.franky.d04.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
	
    ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/general.mecha.franky.d04.com.conf
```

Setelah itu pada file .htaccess ditambahkan seperti di bawah
```sh
echo 'AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user' > /var/www/general.mecha.franky.d04.com/.htaccess

service apache2 restart
```
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/940.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/942.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/945.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/947.png)

## Nomor 16
setiap mengakses IP Skypie akan langsung diarahkan ke **www.franky.d04.com**.

## Jawab
Karena tidak bisa mengakses IP EniesLobby, maka IP yang kami alihkan adalah ketika mengakses IP Skypie
Edit file `/var/www/html/.htaccess` dengan menambahkan:

```bash
    RewriteEngine On
    RewriteBase /
    RewriteCond %{HTTP_HOST} ^10\.23\.2\.4$
    RewriteRule ^(.*)$ http://www.franky.d04.com [L,R=301]
```

Kemudian, edit file /etc/apache2/sites-available/000-default.conf dengan menambahkan:

```bash
    <Directory /var/www/html>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
```

## Nomor 17
mengganti request gambar yang memiliki substring "franky" akan diarahkan ke **franky.png**.

## Jawab
Seperti pada nomor 9, ditambhakn rewrite rule yang makna jika user mengakses file dengan substring "franky" akan diarahkan ke franky.png
```sh
echo '<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    ServerName super.franky.d04.com
    ServerAlias www.super.franky.d04.com
    DocumentRoot /var/www/super.franky.d04.com

    ErrorDocument 404 /error/404.html
    Alias "/js" "/var/www/super.franky.d04.com/public/js"

    <Directory /var/www/super.franky.d04.com/public>
        Options +Indexes
    </Directory>

    <Directory /var/www/super.franky.d05.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/super.franky.d04.com.conf

echo 'RewriteEngine On
    RewriteRule ^(.*)franky(.*)\.(jpg|gif|png)$ http://super.franky.d05.com/public/images/franky.png [L,R]' >> /var/www/super.franky.d04.com/.htaccess

a2ensite super.franky.d04.com.conf

service apache2 restart 
```
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/947.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/948.png)
![img](https://github.com/azhar416/Jarkom-Modul-2-D04-2021/blob/main/img/951.png)
