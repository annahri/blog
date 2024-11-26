---
categories:
- Sysadmin
- Infrastruktur
date: 2021-05-27
draft: false
linktitle: Instal LEMP Stack pada Ubuntu 20.04
title: Instalasi LEMP Stack (Linux, Nginx, MariaDB, PHP) pada Ubuntu 20.04
slug: 'lemp-stack-ubuntu20-04-focal'
tags: 
- webserver
- ubuntu-focal
- nginx
- php-fpm
- stack
---

![Bismillah](/images/bismillah-2.png#center)

LEMP stack adalah software stack yang terdiri dari (L)inux, (E)gin-x alias Nginx,
(M)ysql atau Mariadb, dan (P)HP.

Di sini, saya akan menjelaskan bagaimana cara instalasi LEMP stack pada Ubuntu 20.04.
Untuk OS berbasis RPM menyusul.

### Konfigurasi Awal

Pastikan repositori Anda sudah pada kondisi terbaru dan package pada sistem anda terupgrade.

```sh
$ sudo apt update && sudo apt upgrade -y
```

Jika menggunakan firewall, pastikan port 80 (HTTP) dan 443 (HTTPS) terbuka.
Kalau anda menggunakan `ufw` maka perintahnya adalah:

```sh
$ sudo ufw allow proto tcp from any to any port 80,443
```

Gunakan perintah dibawah ini kalau Anda menggunakan `iptables`:

```sh
$ sudo iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT
# agar rule diatas menjadi 'permanen'
$ iptables-save | sudo tee /etc/iptables/rules.v4
```

### Instalasi Stack

Selanjutnya instal nginx, mariadb, dan php-fpm :

```sh
$ sudo apt install -y \
    nginx \
    mariadb-server mariadb-common \
    php7.4-fpm php7.4 php7.4-common php7.4-mysql
```

Pastikan semua service berjalan dengan baik :

```sh
$ sudo systemctl status nginx
$ sudo systemctl status mariadb
$ sudo systemctl status php7.4-fpm
```

Jika belum berjalan, maka jalankan service-service tersebut dengan perintah:

```sh
$ sudo systemctl enable nginx --now
$ sudo systemctl enable mariadb --now
$ sudo systemctl enable php7.4-fpm --now
```

Kemudian cek juga apakah port 80 sudah listen atau belum.

```sh
$ sudo ss -tulpn | grep ':80'
```

Untuk port 443 masih belum. Karena setelah instalasi baru, secara asal port 80 yang listen.

### Tes Web Server

Untuk itu, silahkan coba buka browser menggunakan alamat IP server. Atau juga bisa menggunakan domain.
Jika memang sudah diatur A recordnya.

Atau, pengecekan bisa dilakukan dengan menggunakan `curl` :

```sh
$ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Tes Halaman PHP

Buat file php dengan isi berikut, simpan di `/var/www/html/phpinfo.php`:

```php
<?php
    phpinfo();
?>
```

Kemudian pada browser / `curl`, silahkan coba akses `http://IP SERVER/phpinfo.php`:

```sh
$ curl localhost/phpinfo.php
<?php
   phpinfo();
?>
```

Kok tidak berhasil? Ya, karena `nginx` belum diatur untuk memproses halaman `php`.

Edit file `/etc/nginx/sites-available/default.conf` kemudian cari baris berikut, dan hilangkan pagar-pagar didepannya (alias uncomment) :

```conf
        location ~ \.php$ {
               include snippets/fastcgi-php.conf;
        #       # With php-fpm (or other unix sockets):
               fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        }
```

Sederhananya, blok diatas menginstrusikan `nginx` untuk memproses seluruh file yang berakhiran `.php` kepada `php-fpm`.
Bisa melalui socket unix (`unix:/path/to/socket.sock`) atau socket tcp (`IP:port`).

Setelah itu, reload konfigurasi nginx `sudo nginx -s reload` dan tes kembali menggunakan `curl` atau browser.

```sh
$ curl localhost/phpinfo.php
```

Jika menghasilkan halaman info php, maka konfigurasi sudah benar.

### Konfigurasi dan Tes Konektivitas Database

Biasakan untuk selalu menjalankan `mysql_secure_installation` setiap kali menginstall `mariadb-server` atau `mysql-server`.
Karena sesuai dengan namanya, perintah tersebut memastikan instalasi mysql itu aman alias secure.

```sh
$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
Y   
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Jawab setiap promptnya dengan `Y`.

Kemudian kita buat satu database percobaan dan satu database user.

```sh
$ sudo mysql
```

```mysql
CREATE DATABASE tesdb;
CREATE USER dbuser@localhost IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON tesdb.* TO dbuser@localhost;
FLUSH PRIVILEGES;
```

Buat satu file untuk uji coba konek php ke database. Simpan di `/var/www/html/tesdb.php`

```php
<?php
    $con = mysqli_connect("localhost","dbuser","password","tesdb");

    if (mysqli_connect_errno()) {
        echo "Failed to connect to MySQL: " . mysqli_connect_error();
        exit();
    } else {
        echo "Koneksi Database Berhasil.";
    }
?>
```

Tes kembali menggunakan browser atau `curl`, buka `http://IP SERVER/testdb.php`

```sh
$ curl localhost/testdb.php
Koneksi Database Berhasil.
```

Selesai!

---

### Kesimpulan

Begitu mudahnya untuk melakukan instalasi LEMP stack pada Ubuntu 20.04.
Cukup menginstal `nginx` + `mysql`/`mariadb` + `php-fpm`, dan sedikit konfigurasi pada nginx.

Tentunya akan memerlukan konfigurasi lebih lanjut jika hendak menginstal webapp semacam wordpress, dan semisalnya.

Semoga bermanfaat, insyaAllah.
