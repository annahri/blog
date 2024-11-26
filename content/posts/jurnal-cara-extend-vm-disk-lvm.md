---
title: "Jurnal: Cara Extend VM Disk untuk LVM"
date: 2021-11-02T18:58:41Z
draft: false
categories:
- Sysadmin
tags:
- lvm
- extend
- disk
---

![Bismillah](/images/bismillah-2.png#center)

Saya akan menjelaskan cara mudah untuk resize disk suatu VM dengan konfigurasi LVM. Asumsinya, disk sudah diextend dari sisi HVnya (PVE, ESXi, dll).

Pastikan penambahan space sudah terdeteksi dari VM melalui `dmesg` dan Anda akan mendapati informasi seperti ini:

```
[262733.527587] sd 0:0:2:0: [abc] 4096-byte physical blocks
[262733.528263] abc: detected capacity change from 214748364800 to 429496729600
```

Kalau ternyata belum terdeteksi, bisa dicoba untuk menjalankan perintah ini 

```sh
echo 1 > /sys/class/block/<disk>/device/rescan
```

Lalu coba cek kembali di `dmesg`. Kalau sudah muncul bisa langsung jalankan perintah berikut:

```sh
growpart /dev/<disk> <no-partisi> # contoh `growpart /dev/sda 1`
pvresize /dev/<partisi> # contoh `pvresize /dev/sda1`
lvresize --extents +100%FREE --resizefs /dev/xxx/yyy # Value xxx yyy merujuk pada logical volume path
```

Khusus untuk `growpart` jika tidak tersedia pada sistem Anda, maka bisa Anda gunakan `parted`

```sh
parted /dev/<disk>
(parted) resizepart <no-partisi> 100%
(parted) quit
```

Selesai. Semoga bermanfaat, _insyaAllah_
