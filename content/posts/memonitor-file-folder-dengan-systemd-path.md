---
title: "Memonitor File/Folder dengan Systemd Path Unit"
date: 2023-08-12T16:31:05+07:00
draft: false
tags:
- systemd
- systemd-path
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel kali ini, saya akan membahas mengenai salah satu fitur Systemd yang memungkinkan user untuk menjalankan suatu _command_/_script_ atau bahkan mengeksekusi Systemd Service berdasarkan aktifitas suatu file/folder yang dimonitor. Baik ketika file terkait telah dimodifikasi, folder terkait terdapat file baru yang dibuat, dan sebagainya.

Komponen systemd yang dimaksud adalah _Systemd **Path** unit_.

## Ringkasan Systemd unit Path

Unit `Path` pada systemd ini memiliki file dengan akhiran `.path` yang didalamnya berisikan instruksi terkait file/folder apa yang dipantau dan apa yang akan dieksekusi ketika kondisi pemantauan terpenuhi.

Mengenai apa yang dieksekusi, hal tersebut harus diatur menggunakan Systemd unit `Service` sehingga paling tidak harus membuat 2 file untuk keperluan ini, yaitu:

1. File konfigurasi `.path`
2. File konfigurasi `.service`

Kedua file konfigurasi tersebut, bisa diletakkan di `/etc/systemd/system/` untuk lingkup global, atau `/home/$USER/.config/systemd/user/` untuk lingkup user tertentu.

## Membuat unit Path

Misal, kita ingin memonitor apakah suatu file (anggap saja `/var/www/index.php`) mengalami perubahan isi, dan ketika hal itu terjadi akan mengirimkan alert berupa email.

Maka, kita buat file `/etc/systemd/system/monitor-index.path` dengan isi:

```systemd
[Unit]
Description="Monitor file /var/www/index.php"

[Path]
PathModified=/var/www/index.php
Unit=monitor-index.service

[Install]
WantedBy=multi-user.target
```

kemudian, isi dari file `/etc/systemd/system/monitor-index.service`:
```systemd
[Unit]
Description="Alert changes on /var/www/index.php by email"

[Service]
Type=oneshot
ExecStart=bash -c '\
    printf "From: %s\nTo: %s\nSubject: %s\n\n%s\n" \
    "root@system" \
    "target@domain.tld" \
    "Change alert" \
    "File /var/www/index.php has been changed. Please check immediately!!!" | \
    sendmail -f root@system target@domain.tld'
```

Setelah itu, tinggal jalankan perintah dibawah ini agar `systemd` mengenali kedua file unit diatas:
```
systemctl daemon-reload
```

{{% callout %}}
Setiap kali ada pengubahan unit apapun pada `systemd` maka perlu melakukan `systemctl daemon-reload`
{{% /callout %}}

## Mengaktifkan unit Path

Kemudian meng-_enable_ unit path `monitor-index.path` tersebut dengan:

```
systemctl enable --now monitor-index.service
```

Ketika dicek statusnya dengan `systemctl status monitor-index.path`, maka akan muncul seperti ini:
```
● monitor-index.path - "Monitor file /var/www/index.php"
     Loaded: loaded (/etc/systemd/system/monitor-index.path; enabled; preset: enabled)
     Active: active (waiting) since Sat 2023-08-12 05:32:42 WIB; 1m ago
   Triggers: ● monitor-index.service

Aug 12 05:32:42 localhost systemd[532]: Started "Monitor file /var/www/index.php".
```

Bisa diperhatikan dari output diatas, bahwa unit `Path` tersebut akan men-_trigger_ unit `monitor-index.service` jika kondisi pada unit `Path` ini terpenuhi.

## Menonaktifkan unit Path

Untuk menonaktifkan unit `Path`, jalankan perintah berikut ini:

```
systemctl disable --now monitor-index.path
```

## Penutup

Selain opsi `PathModified`, terdapat opsi-opsi lain yang bisa digunakan pada bagian `[Path]`, diantaranya adalah:

```
PathExists=, PathExistsGlob=, PathChanged=, PathModified=, DirectoryNotEmpty=
```

yang seluruhnya bisa dibaca secara detail pada manpages yang bisa diakses melalui `man systemd.path`.

Perlu diketahui pula, sebenarnya unit `Path` ini memanfaatkan API dari `inotify` sehingga unit ini mewarisi semua limitasi yang ada pada `inotify`.

Kemudian, untuk keperluan file monitoring yang lebih kompleks, unit ini kurang cocok untuk dimanfaatkan karena tidak terlalu fleksibel.


Semoga bermanfaat. _Barakallahufiikum_
