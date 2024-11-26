---
title: "Trik Bash Shell #1"
date: 2021-11-04T23:28:21Z
draft: false
slug: trik-bash-shell-1
series: ["Bash Shell Tricks"]
tags:
- bash
- shell
- tips-n-trik
categories:
- Bash
---

![Bismillah](/images/bismillah-2.png#center)

Saya akan menjelaskan beberapa tips dan trik dalam menggunakan _shell_ **bash** yang biasa saya gunakan sehari-hari. 

## Reverse Search

Sebagian orang biasa menggunakan `history` untuk melihat/mencari _command-command_ sebelumnya yang sudah pernah diinputkan. Atau bahkan menggunakan _arrow key_ atas untuk mencari _command_ yang diinginkan. 

Sebenarnya hampir kebanyakan _shell_ memiliki fitur _reverse search_. Sesuai namanya, fitur ini bisa melakukan pencarian terhadap _command_ yang sebelumya pernah diinputkan. 

Bagaimana caranya? Cukup menekan <kbd><kbd>CTRL</kbd>+<kbd>R</kbd></kbd> pada terminal. Dan ketikkan kata kunci untuk _command_ yang sedang dicari.

```sh
(reverse-i-search)`apt': sudo apt install hugo
                    ^    ^
                    |    |- history terakhir yang paling relevan
                    |- keyword
```

## Sintaks Spesial !!, !$ dan !n

### Sintaks !!

Pada bash shell (atau shell yang lain juga, mungkin) menrujuk kepada command terakhir yang diinputkan.

Jadi, jika sebelumnya kita menginputkan, misal, perintah `echo abc`. Lalu pada input berikutnya jika kita memasukkan `!!` maka, shell akan mengeksekusi kembali perintah `echo abc`.

```sh
user@host:~$ echo abc
abc
user@host:~$ !!
echo abc    <--- terminal tidak benar-benar mengeluarkan output ini, hanya sebagai penjelasan ekspansi shell saja
abc
```

Ini sangat berguna ketika kita lupa hendak menambahkan `sudo` pada suatu command yang panjang. Maka, pada input berikutnya, kita tinggal ketikkan `sudo !!` tanpa perlu mengetik ulang command sebelumnya.

```sh
user@host:~$ apt install -y package1 package2 package3 package4 package5
E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
user@host:~$ sudo !!
sudo apt install -y package1 package2 package3 package4 package5
...
```

### Sintaks !$

Sintaks ini merujuk pada _positional argument_ pada _command_ terakhir. Sama seperti contoh sebelumnya, jika `echo abc` maka ekspansi dari `!$` adalah `abc`.

Kasus penggunaan sintaks ini biasanya saat melakukan editing file. Seperti di bawah ini:

```sh
user@host:~$ cat /path/panjang/menuju/file
isi file
user@host:~$ vim !$
vim /path/panjang/menuju/file
...
```

Jadi tidak perlu lagi mengetik ulang path panjang file yang dituju, tapi cukup gunakan sintaks `!$` saja.

### Sintaks !n

Huruf `n` merujuk kepada angka pada history shell. Jika kita menjalankan perintah `history` maka tiap-tiap baris history terdapat angka disebelahnya (kecual jika konfigurasi untuk history sudah tidak default). Nah, dengan menggunakan `!n` maka shell akan mengulangi command yang sama sesuai dengan nomor yang tertera.

Contoh:

```sh
user@host:~$ history
    1  ll
    2  ls
    3  lsblk
    4  history
user@host:~$ !1
ll
total 20
drwxr-xr-x 1 user user  110 Nov  5 07:23 ./
drwxr-xr-x 1 root root   42 Nov  5 07:22 ../
-rw-r--r-- 1 user user  220 Nov  5 07:22 .bash_logout
-rw-r--r-- 1 user user 3771 Nov  5 07:22 .bashrc
drwx------ 1 user user    0 Nov  5 07:23 .cache/
drwxr-xr-x 1 user user   40 Nov  5 07:22 .config/
-rw-r--r-- 1 user user    5 Nov  5 07:22 .hidden
-rw-r--r-- 1 user user   87 Nov  5 07:22 .inputrc
-rw-r--r-- 1 user user  807 Nov  5 07:22 .profile
```

## Pintasan Clear Screen `clear`

Command `clear` digunakan untuk membersihkan sesi layar shell. Agar lebih cepat, cukup tekan kombinasi tombol <kbd><kbd>CTRL</kbd>+<kbd>L</kbd></kbd>. 

---

Bersambung, _insyaAllah_