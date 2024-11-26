---
title: "Zimbra: Menggunakan SSL Certificate dari LetsEncrypt"
date: 2023-02-14T08:08:21+07:00
tags:
- zimbra
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

LetsEncrypt merupakan salah satu penyedia sertifikat SSL gratis yang banyak sekali digunakan. 

Pemanfaatan sertifikat SSL dari LetsEncrypt pada Zimbra bisa dilakukan tetapi caranya tidak _out-of-the-box_. 
Sehingga pada artikel kali ini saya akan menjelaskan bagaimana cara menggunakan LetsEncrypt sebagai penyedia sertifikat SSL untuk _instance_ zimbra Anda.

Saya menulis artikel ini dengan asumsi bahwa Anda sudah bisa dan terbiasa menggunakan `certbot` untuk merequest sertifikat SSL.

## Garis besar

Secara garis besar, cara untuk menggunakan LE (LetsEncrypt) pada Zimbra adalah sebagai berikut:

1. Menghentikan zimbra-proxy (`zmproxyctl`)
1. Merequest SSL menggunakan `certbot` dengan flag `--nginx` (dan tentunya dengan flag-flag lain yang digunakan untuk merequest sertifikat SSL).
1. Menyalin file-file SSL yang didapat, ke direktori yang bisa diakses oleh user `zimbra`.
1. Mendeploy sertifikat SSL sebagai user `zimbra` dengan `zmcertmgr deploycrt`
1. Menyetop semua instance `nginx` yang bukan dari zimbra.
1. Merestart service-service zimbra dengan `zmcontrol restart`

## Kebutuhan sistem

Paket-paket yang dibutuhkan pada sistem:

1. Tentunya, aplikasi `certbort`
1. Plugin nginx untuk certbot: `python3-certbot-nginx`

## Langkah-Langkah

1. Stop terlebih dahulu, service proxy milik zimbra, yang itu dia menggunakan port 80. Jika tidak distop, maka `certbot` tidak akan bisa menjalankan webserver nginx sebagai media validasi kepemilikan domain.
Stop sebagai user `zimbra` menggunakan perintah: `zmproxyctl stop`
2. Jalankan `certbot` beserta flag-flag yang diperlukan. Misalnya berikut ini:
```shell
certbot --nginx -q -d mail.mydomain.tld
```
3. Jika berhasil, buatlah direktori baru pada */opt/zimbra* (atau ditempat yang lain yang bisa diakses oleh user `zimbra`) misalnya `/opt/zimbra/ssl/mycerts`
4. Salin semua file-file SSL yang dibutuhkan dari folder `/etc/letsencrypt/live/mail.mydomain.tld` ke folder yang barusaja dibuat sebelumnya.
5. Jika diperlukan, backup private key di folder `/opt/zimbra/ssl/commercial/commercial.key` pada suatu folder lain.
6. Timpa file `commercial.key` yang asli tersebut tersebut dengan private key dari LetsEncrypt di `/etc/letsencrypt/live/mail.mydomain.tld/privkey.pem`. Bisa menggunakan _command_ berikut:
```shell
install --user zimbra --group zimbra \
    /etc/letsencrypt/live/mail.mydomain.tld/privkey.pem \
    /opt/zimbra/ssl/commercial/commercial.key
```
7. Setelah itu, jalankan perintah `zmcertmgr` sebagai user `zimbra` berikut:
```shell
zmcertmgr deploycrt comm <certificate> <ca-bundle>
# misal
zmcertmgr deploycrt comm /opt/zimbra/ssl/mycerts/fullchain.pem /opt/zimbra/ssl/mycerts/chain.pem
```
8. Begitu selesai, restart zimbra services:
```
zmcontrol restart
```
9. Selesai.

## Akhir kata

Langkah-langkah diatas bisa Anda buatkan scriptnya agar bisa dijalankan secara praktis via cronjob. 

Atau, Anda bisa menggunakan script buatan saya yang bernama `zimbra-autossl.sh` di sini: https://github.com/annahri/zimbra-autossl

Penjelasan mengenai script tersebut sudah ada di repositori terkait. Pada intinya proses yang ada pada script tersebut, sesuai dengan garis besar yang sudah dijelaskan diatas.

Semoga bermanfaat, _barakallahufiikum
