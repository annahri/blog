---
title: "Membenahi Kendala Boot Macet Pada Ubuntu"
date: 2023-09-13T17:07:35+07:00
draft: false
tags:
- ubuntu
- troubleshooting
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel kali ini, saya akan membagikan mengenai bagaimana cara membenahi VM Ubuntu yang macet pada saat booting. Salah satu diantara penyebab gagalnya booting ini adalah `PARTUUID` yang berbeda antara `PARTUUID` pada partisi terkait (root) dan `PARTUUID` yang dikenali oleh `grub`.

Tampilan boot yang macet tersebut adalah seperti dibawah ini: 

![Boot Macet](./screenshot.jpg#center)

Sistem tidak mampu meneruskan proses boot karena `PARTUUID` yang hendak di-_boot_ tidak ditemukan.

## Pemecahan Masalah

Boot sistem tersebut menggunakan media _recovery_ apapun seperti SysrescCd, Ubuntu Live, dan lain-lain, yang terpenting bisa melakukan `chroot` kedalam sistem.

Saat pada sesi _recovery_, lakukan persiapan `chroot` dengan perintah-perintah berikut:

```bash
mount /dev/sdX /mnt
mount --bind /dev /mnt/dev
mount --bind /sys /mnt/sys
mount --bind /proc /mnt/proc
```
{{% callout %}}
Dimana `X` pada `/dev/sdX`, adalah partisi `root`
{{% /callout %}}

Jika sistem terkait menggunakan partisi `boot` terpisah, silakan di-`mount` juga.

Kemudian, masuk ke sesi `chroot`:

```bash
chroot /mnt
```

Setelah itu, biasanya sesi `chroot` tidak masuk sebagai _login shell_ sehingga beberapa `env` variabel tidak termuat, seperti `PATH`. Untuk itu lakukan:

```bash
su -
# atau
bash -i
```

Kemudian bandingkan `uuid` yang terkonfigurasi pada file `/etc/default/40-force-partuuid.cfg` dengan output dari command:

```
blkid -s PARTUUID -o value /dev/sdX
```

Dengan adanya kendala ini, dipastikan kedua valuenya berbeda. 

Untuk itu, _replace_ value variabel `GRUB_FORCE_PARTUUID` pada file `/etc/default/40-force-partuuid.cfg` dengan value pada perintah `blkid` diatas.

Setelah itu, lakukan _rebuild_ grub:

```bash
update-grub
```

Jika sudah selesai, silakan `reboot`. Sistem akan berhasil booting seperti sedia kala.

## Penutup

Penyebab dari berubahnya `PARTUUID` tersebut masih belum saya ketahui secara pasti. Kemungkinan, `uuid` tersebut berubah karena hasil dari clone VM yang pada prosesnya terdapat semacam proses "randomize" `uuid`-s.

Diantara referensi yang saya dapat, ada yang mengindikasikan adanya bug pada template ubuntu:
- [initramfs does not get loaded](https://bugs.launchpad.net/ubuntu/+source/livecd-rootfs/+bug/1870189)

---

Semoga bermanfaat, _Barakallahufiikum_.
