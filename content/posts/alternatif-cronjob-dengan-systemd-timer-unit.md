---
title: "Alternatif Cronjob Menggunakan Systemd Timer Unit"
date: 2023-08-12T17:00:37+07:00
draft: false
tags:
- systemd
- systemd-timer
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Systemd tidaklah sebatas software yang berfungsi sebagai init system saja tetapi memiliki sejumlah tool-set lain yang bermanfaat. Salah satunya adalah Systemd unit `Timer`.

Unit `Timer` berfungsi untuk memicu (_trigger_) suatu unit systemd lain berdasarkan fungsi waktu. Secara sederhana bisa dipahami sebagai alternatif dari `cron`. Fungsi waktu yang dimaksud bisa berupa, namun tidak terbatas pada, interval waktu (tiap menit, tiap jam atau tiap sekian waktu), penjadwalan eksekusi, dan sebagainya.

Sama seperti kebanyakan unit systemd yang lainnya, unit `Timer` ini juga memerlukan unit lain seperti unit `Service` untuk mengeksekusi sesuatu berdasarkan konfigurasi waktu yang ditentukan. Ketika sistem memasuki waktu yang ditentukan, maka unit `Timer` ini perlu dan akan mentrigger unit lain untuk menjalakan seperangkat perintah yang ditentukan.

## Membuat Unit Timer

Seperti pada kebanyakan unit systemd, unit ini bisa disimpan secara global di `/etc/systemd/system` atau secara terbatas pada lingkup user terkait di `~/.config/systemd/user`.

Dimisalkan ada keperluan untuk menjalankan perintah `certbot renew` setiap pekannya menggunakan unit `Timer` ini. Maka file yang perlu dibuat adalah:

**certbot-renew.timer**

```systemd
[Unit]
Description="Triggers certbot-renew.service unit"

[Timer]
OnCalendar=Sun *-*-* 00:01:00

[Install]
WantedBy=multi-user.target
```

Dengan konfigurasi diatas, maka unit tersebut akan mentrigger unit lain dengan nama yang sama tetapi dengan ekstensi `.service`.

{{% callout %}}
Jika ingin memanggil unit lain, definiskan hal tersebut menggunakan `Unit=nama-unit.service` pada bagian `[Timer]`
{{% /callout %}}

**certbot-renew.service**

```systemd
[Unit]
Description="Executes certbot renew"

[Service]
Type=oneshot
ExecStart=/usr/bin/certbot renew
```

## Mengaktifkan Unit Timer

Agar semua penambahan atau modifikasi pada unit systemd apapun bisa digunakan, perlu melakukan perintah berikut:

```shell
systemctl daemon-reload

# User unit menggunakan --user
systemctl --user daemon-reload
```

Kemudian untuk menjalankannya sekaligus membuatnya aktif secara otomatis setiap kali reboot (aktif bukan dalam artian langsung tereksekusi), menggunakan perintah berikut:

```shell
systemctl enable --now certbot-renew.timer
```

Cukup melakukan _enable_ untuk unit `Timer` saja, karena unit `certbot-renew.service` akan ditrigger oleh timer tersebut, sehingga tidak perlu di-_enable_ secara tersendiri. 

Output yang akan muncul ketika diverifikasi dengan `systemctl status certbot-renew.timer` adalah seperti berikut ini:

```
● certbot-renew.timer - "Triggers certbot-renew.service unit"
     Loaded: loaded (/etc/systemd/system/certbot-renew.timer; enabled; preset: enabled)
     Active: active (waiting) since Sat 2023-08-12 17:30:13 WIB; 2s ago
    Trigger: Sun 2023-08-13 00:01:00 WIB; 6h left
   Triggers: ● certbot-renew.service

Aug 12 17:30:13 localhost systemd[532]: Started "Triggers certbot-renew.service unit".
```

## Melihat Daftar Timer yang Aktif

Jalankan perintah dibawah ini untuk melihat daftar timer apa saja yang sedang aktif:

```
systemctl list-timers
```

Contoh:

```
NEXT                        LEFT       LAST                        PASSED    UNIT                ACTIVATES
Sat 2023-08-12 18:00:00 WIB 23min left Sat 2023-08-12 17:00:00 WIB 36min ago hourly-play.timer   hourly-play.service
Sun 2023-08-13 00:01:00 WIB 6h left    -                           -         certbot-renew.timer certbot-renew.service

2 timers listed.

```

## Menonaktifkan Timer

Sama seperti unit systemd yang lain, unit `Timer` bisa dinonaktifkan dengan cara yang sama pula:

```
systemctl disable --now certbot-renew.timer
```

## Timer Monotonik

Maksudnya adalah, jenis timer yang tidak berkaitan dengan satuan waktu di dunia nyata (hari, pekan, bulan, tanggal tertentu, dsb). Namun, relatif kepada momen tertentu pada sistem. Seperti, waktu setelah sistem boot, setelah unit aktif, dsb.

Opsi yang bisa digunakan untuk mengatur timer monotonik ini adalah:

```
OnActiveSec=, OnBootSec=, OnStartupSec=, OnUnitActiveSec=, OnUnitInactiveSec=
```

Masing-masing rinciannya bisa dilihat melalui `man systemd.timer`.

Contohnya untuk opsi `OnBootSec=`, misal diatur menjadi `OnBootSec=50s`, maka unit ini akan melakukan trigger 50 detik setelah booting dilakukan.

## Penutup

Unit `Timer` systemd ini bisa dimanfaatkan sebagai alternatif dari `cron`. Salah satunya memiliki kelebihan dan kekurangan dari yang lainnya.

---

Semoga bermanfaat, _barakallahufiikum_.
