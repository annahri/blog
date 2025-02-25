---
title: "Cara Search Replace dengan Mengubah Huruf Besar-Kecil di VIM"
description: "Pelajari cara mudah melakukan search-replace dan mengubah gaya huruf (case) di VIM dengan cepat. Tips praktis untuk pengguna VIM!"
date: 2025-02-24T23:18:39+07:00
tags:
- vim
- search-replace
- letter-case
categories:
- Menggunakan VIM
---

![Cover](cover.png#center)

Dalam suatu kasus, saya perlu untuk mengganti beberapa baris _substring_, menambahkan sesuatu dan kemudian menjadikan kata setelahnya menjadi kapital.

Hal tersebut terjadi saat saya sedang mengupdate konfigurasi template _Zabbix_ yang saya buat. Beberapa nama item perlu saya bubuhi "Zimbra:" di awalnya. Masalahnya, beberapa item sudah memiliki awalan "Zabbix" diikuti dengan nama itemnya tetapi berhuruf kecil.
Sehingga, saya perlu menghubah huruf tersebut menjadi kapital agar konsisten dan rapi.

Contoh inputnya seperti ini:

```
...
    name: 'Zimbra: Zimbra domains discovery'
...
```

Output yang diharapkan adalah:

```
...
    name: 'Zimbra: Domains discovery'
...
```

Maka, sintaks VIM yang saya gunakan adalah:

```vim
:%s/Zimbra: Zimbra \(\w\+\)\(.*\)/Zimbra: \u\1\2/gc
```

Yang penjelasannya adalah:

- `%`, eksekusi untuk semua baris pada file ini.
- `s`, perintah untuk melakukan substitusi berdasarkan dengan pola `s/POLA/PENGGANTI/flag`, dimana `flag` ini mengatur sifat substitusi tersebut.
- `Zimbra: Zimbra \(\w\+\)\(.*\)`, mencari baris yang terdapat kata "Zimbra: Zimbra " diikuti dengan sembarang kata (alfabet) yang disimpan pada grup 1 dan diikuti dengan apapun yang disimpan pada grup 2.
- `Zimbra: \u\1\2`, diganti dengan teks "Zimbra: " diikuti dengan grup 1 yang huruf pertamanya dijadikan kapital (`\u`, satu huruf), dan kemudian diikuti apapun pada grup 2.
- `gc`, lakukan substitusi tidak hanya pada temuan pertama pada baris terkait (`g`) dan tanyakan konfirmasi sebelum melakukan substitusi (`c`).

---

Selain `\u`, ada juga modifier lain:

- `\U`, mengganti seluruh karakter hingga akhir menjadi huruf kapital, atau hingga bertemu modifier `\e`.
- `\l`, mengganti huruf setelahnya menjadi huruf kecil.
- `\L`, mengganti seluruh karakter hingga akhir menjadi huruf kecil, atau hingga bertemu modifier `\e`.
- `\e`, penanda akhir untuk modifier `\U` dan `\L`

Contoh penerapan:

```
# input
foo bar

# sintaks
s/foo/\u&/

# hasil
Foo bar
```

```
# input
foo BAR

# sintaks
s/\(foo\) \(bar\)/\U\1\e \L\2\e/

# hasil
FOO bar
```

---

Sekian, semoga bermanfaat _barakallahu fiik_.
