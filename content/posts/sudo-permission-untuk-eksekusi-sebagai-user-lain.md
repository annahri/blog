---
title: "Membuat Entri Sudoers untuk Eksekusi Sebagai User Lain"
description: "Belajar bagaimana membuat entri sudoers untuk memungkinkan suatu pengguna untuk mengeksekusi perintah sebagai pengguna lain. Panduan step-by-step untuk System Adminstrator/DevOps."
date: 2024-11-25T19:21:16+07:00
tags:
- sudo
- sudoers
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Umumnya, entri sudoers yang dibahas pada tutorial-tutorial, seringnya hanya membahas penambahan entri sudoers biasa, alias memungkinkan user untuk mengeksekusi semua perintah sebagai root.

Pada artikel kali ini, saya akan menjelaskan bagaimana membuat entri sudoers untuk memungkinkan user terkait untuk bisa mengeksekusi perintah yang ditentukan sebagai user lain. Contoh kasus dimana hal ini sangat bermanfaat adalah ketika ada _monitoring agent_ seperti Zabbix yang biasanya berjalan sebagai user tertentu (zabbix), yang memerlukan izin untuk menjalankan suatu perintah sebagai user lain tetapi tidak sampai diberi akses super user. 

## Sintaks Dasar File Sudoers

Pada dasarnya, file sudoers memiliki berbagai "spesifikasi" untuk berbagai tujuan seputar sudo (bisa dilihat via `man sudoers`). Tetapi yang akan dibahas disini hanya seputar __User specification__, yaitu bagian inti yang menentukan tentang siapa yang bisa menjalankan apa.

```sudoer
user  HOST=(user:group) LIST

# untuk grup, awali dengan %
%group HOST=(user:group) LIST
```

Ada 3 bagian inti: 

1. penentuan user/group
2. host dan user dan/atau group target
3. list perintah

Poin no.1 adalah user/group yang hendak menjalankan perintah. Kemudian no.2 adalah user/group dimana perintah tersebut akan dijalankan. Dan no.3 tentu daftar perintah yang ingin diizinkan, dipisahkan dengan koma.

Mengenai bagian HOST, pada umumnya cukup diset ke `ALL`, alias diizinkan dijalankan ke semua host. Kemudian pada bagian user/group target, bisa sekedar menuliskan usernya saja, groupnya saja atau keduanya.

Jika ingin menjalankan perintah terkait tanpa perlu memasukkan password, maka sebelum LIST perintah, berikan `NOPASSWD:`.

Contoh:

```sudoers
user ALL=(target) NOPASSWD: LIST
```

Yang terakhir, pada bagian LIST, perintah yang dimasukkan perlu ditulis secara lengkap _full path_-nya. Misal, `cp` harus `/usr/bin/cp`; `rm` harus `/usr/bin/rm` dan seterusnya. Sehingga, contoh sebelumnya menjadi:

```sudoers
user ALL=(target) NOPASSWD: /usr/bin/cp, /usr/bin/rm
```

## Contoh kasus

Mari kita gunakan contoh skenario berikut: kita ingin memonitor sesuatu pada instance Zimbra kita menggunakan Zabix active agent. Perintah yang perlu dijalankan adalah dibawah ini:

```bash
zmsoap -z --json GetServiceStatusRequest
```

Perintah diatas hanya bisa dijalankan sebagai user `zimbra` saja dan _monitoring agent_ yang digunakan, berjalan sebagai user `zabbix`. Sehingga, user `zabbix` perlu untuk bisa menjalankan perintah tersebut sebagai user `zimbra`. Maka, perintah yang akan dijalankan user `zabbix` menjadi:

```bash
sudo -inu zimbra zmsoap -z --json GetServiceStatusRequest

# -i   run login shell as the target user;
# -n   non-interactive mode, no prompts are used
# -u   run command (or edit file) as specified username or ID
```

Dengan spesifikasi diatas, maka entri sudoers yang diperlukan adalah sebagai berikut:

```
zabbix ALL=(zimbra) NOPASSWD: /bin/bash -c zmsoap -z --json GetServiceStatusRequest
```

__Kenapa ada `/bin/bash -c` ?__

Hal tersebut disebabkan karena ketika suatu perintah dijalankan melalui `sudo`, maka perintah tersebut akan dijalankan dari `bash -c` (atau shell lain). Pada kasus ini, shell dari user `zimbra` adalah `bash`.

Kemudian, entri tersebut disimpan pada file `/etc/sudoers.d/zabbix`. Untuk memverifikasinya, jalankan:

```bash
visudo -c -f /etc/sudoers.d/zabbix
```

Jika menghasilkan output: `sudoers: parsed OK` maka tidak ada masalah pada file sudoers tersebut.

{{% callout %}}
Disarankan juga untuk mengedit file sudoers tersebut menggunakan perintah: `visudo -f /etc/sudoers.d/zabbix`

Dengan demikian, jika terjadi kekeliruan sintaks, maka semua modifikasi akan dibatalkan agar tidak membuat fungsi sudo rusak.
{{% /callout %}}

---

Semoga bermanfaat, _barakallahufiikum_.
