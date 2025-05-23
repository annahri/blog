---
title: "NFS: Squash atau ID Mapping"
description: "Pelajari berbagai opsi NFS squash (ID mapping), fungsinya dan penggunaannya."
date: 2025-03-15T08:27:40+07:00
tags:
- NFS
- idmap
categories:
- Sysadmin
---

![Cover](cover.png#center)

NFS squashing adalah metode pemetaan UID dan GID terkait file/folder yang diexport oleh NFS.
Misalnya ketika folder yang diexport dimiliki oleh UID 1000 dan GID 1000, lalu di sisi _client_, apa _ownership_ yang akan dihasilkan ketika membuat file/folder baru? Nah, hal tersebut diatur oleh opsi _squashing_ pada suatu NFS export.

Terdapat 3 jenis metode mapping:

1. `no_root_squash`
2. `root_squash`
3. `all_squash`

Mari kita bahas satu-persatu.

## Opsi 1: `no_root_squash`

![no_root_squash](no_root_squash.png#center)

Opsi ini tidak melakukan pemetaan ID sama-sekali, dalam artian, UID 0 (__root__) pada _client_ akan menghasilkan file ownership dengan UID 0 pula.
Dan root akan diizinkan untuk memodifikasi semua file/folder yang ada pada suatu NFS export, dan aktivitas itu teranggap sebagai aktivitas dari root NFS server itu sendiri.

User lain juga bisa melakukan segala operasi file dengan UID dan GID yang sesuai dengan user tersebut. Jika di sisi _client_ dan _server_ terdapat user dangan UID yang sama tetapi dengan nama yang berbeda, maka file yang tadi akan muncul sesuai dengan sudut pandang masing-masing mesin. Misal di sisi _client_, user `fulan` memiliki UID 1000, dan di sisi `server`, UID tersebut dimiliki oleh user `bob`, maka di sisi client akan muncul sebagai `fulan` dan di server sebagai `bob`.

Dengan demikian, UID yang satu tidak bisa memodifikasi file yang dimiliki oleh UID lain.

## Opsi 2: `root_squash`

![root_squash](root_squash.png#center)

Opsi ini akan memetakan UID 0 kepada UID 65534 (alias user __nobody__) atau ke UID dan GID lain ditentukan dengan parameter `anonuid` dan `anongid`. Sehingga semua operasi file yang dilakukan oleh UID 0, maka akan menghasilkan ownership sebagai UID 65534 (atau sesuai `anonuid`).

Di sisi `client`, user dengan UID yang sama dengan spesifikasi `anonuid` diperbolehkan melakukan segala operasi file, dari membuat file baru atau memodifikasi yang sudah ada.

Adapun untuk user dengan UID selain itu, maka tidak diizinkan untuk melakukan apapun, atau paling tidak menyesuaikan dengan _permission bits_ file terkait. Jika `others` boleh melakukan _write_ atau _read_, maka hal tersebut bisa dilakukan.

## Opsi 3: `all_squash`

![all_squash](all_squash.png#center)

Opsi ini akan memetakan semua UID kepada UID 65534 atau yang ditentukan oleh `anonuid` dan `anongid`. Hampir sama dengan `root_squash`, tetapi user lain, meskipun memiliki UID yang berbeda dari `anonuid`, diizinkan melakukan operasi dan teranggap sebagai UID `anonuid`.

## Contoh penerapan

### Kasus `all_squash`

Katakanlah dalam suatu tim developer, terdapat beberapa person. Tiap personnya memiliki hak akses ke server untuk mengelola file website/aplikasi tertentu (misal pada development server). Akses tiap person menggunakan username masing-masing, dan di sisi server, file-file aplikasi dimiliki oleh user system `www-data` yang disediakan oleh suatu NFS server.

Jika di sisi server, folder tersebut diexport menggunakan `no_root_squash` maka yang terjadi adalah akan ada tumpang tindih file ownership. Developer A membuat file sebagai ownership miliknya sendiri, dan juga developer lain. Kemudian di sisi webserver akan terjadi berbagai kendala seperti 403 forbidden atau yang lain yang berkaitan dengan _file permission_.

Pada kondisi seperti ini, biasanya, tim infra yang kurang berpengalaman akan memberi permission bits 777 agar mengatasi kendala ini. Ibarat staff bank yang kesulitan mengakses brankas yang kemudian solusinya adalah dengan membuka brankas 24 jam kepada semua orang. Tentu sangat konyol dan berbahaya.

Untuk mengatasi problem diatas, maka opsi _squashing_ yang tepat adalah dengan `all_squash` dengan `anonuid` sesuai dengan UID user `www-data`. Sehingga, siapapun yang memiliki akses ke server dan folder tersebut, akan bisa mengelola file tanpa ada tumpang-tindih _ownership_.

![sample1-all_squash](sample1-all_squash.png#center)

Konfigurasi export:

```
/nfs/www/dev_app  hostname(rw,async,all_squash,anonuid=33,anongid=33,no_subtree_check)
```

### Kasus `root_squash`

Misal dalam suatu aplikasi dengan infra 3 tier (loadbalancer, webserver, database) dimana tiap aplikasi dimiliki oleh usernya masing-masing.

Anggap saja ada app1, app2 dan app3 masing-masing tidak di-_own_ oleh `www-data` tetapi `www-app1`, `www-app2` dan `www-app3`. Ini dilakukan agar tiap aplikasi "terisolasi" dari yang lainnya. Di sisi NFS server, opsi squashing yang digunakan adalah `root_squash`:

```
/nfs/www/app1   webservers(rw,async,root_squash,anonuid=2001,anongid=2001)
/nfs/www/app2   webservers(rw,async,root_squash,anonuid=2002,anongid=2002)
/nfs/www/app3   webservers(rw,async,root_squash,anonuid=2003,anongid=2003)
```

Dengan demikian, user ID 2002 tidak memiliki izin untuk mengubah file milik user ID 2001 dan sebagainya. Dari sisi user root, semua operasi file akan teranggap sebagai user `anonuid`.

![sample2-root_squash](sample2-root_squash.png#center)

{{% callout noemoji=true %}}
**Mengapa tidak pakai `no_root_squash` saja? Bukankah sama saja?**

Memang ada kemiripan, tetapi ingat, bahwa pada `root_squash` user UID 0 alias `root` juga terpetakan sebagai user `anonuid`. Dengan demikian, ini akan menghindari munculnya file-file baru atau penggantian file ownership sebagai `root`.
{{% /callout %}}

---

Sekian, jika ada pertanyaan terkait, silakan dikirimkan ke ahfas\[at\]annahri\[dot\]com, _Allahumma Bārik_.
