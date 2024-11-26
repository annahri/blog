---
title: "Belajar Bash Scripting: Membaca File dan Membaca Tiap Barisnya"
date: 2023-07-28T19:40:22+07:00
draft: false
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Dalam menulis script, terkadang kita ingin menjalankan sekumpulan perintah berdasarkan data yang ada pada suatu file (baik raw file, csv, dsb). Kemudian juga terkadang, kita ingin menuliskan suatu script yang menghasilkan suatu file berupa log, atau data terstruktur seperti csv, dan sebagainya.

Maka, pada seri tulisan "Belajar Bash Scripting" kali ini, saya akan membahas mengenai bagaimana cara membaca dan menghasilkan suatu file pada `bash`.

# Membaca File

Secara umum, suatu file teks bisa dibaca menggunakan perintah `cat`:

```
$ cat file.txt
pertama
kedua
ketiga
keempat
kelima
```

{{% callout %}}
Sebenarnya, perintah `cat` ini berfungsi untuk menggabungkan file menjadi 1 dan menampilkannya pada `stdout`.

Karena fungsinya demikian, ketika argumen yang diberikan hanyalah 1 saja, maka tentu yang muncul hanya isi dari file tersebut. Dengan demikian, hal tersebut bisa dimanfaatkan untuk keperluan "membaca isi file".
{{% /callout %}}

Biasanya `cat` digunakan untuk membaca/membuka file yang kemudian akan diproses menggunakan perintah lain agar mudah dibaca. Beberapa perintah mendukung input langsung melalui `stdin` sehingga `cat` tidak diperlukan.

## Menyimpan isi file kedalam variabel

Menggunakan _process substitution_:

```bash
var1="$(cat file.txt)"
var2="$(cat file.txt | ...)"
```

Menggunakan `printf`:

```bash
printf -v var1 '%s\n' "$(cat file.txt)"
```

## Membaca file baris-per-baris

Ada beberapa cara untuk melakukan hal ini, tergantung dengan tujuannya. Namun, seluruhnya menggunakan loop, baik `for` atau `while`

1. Jika hanya melakukan suatu proses **tanpa adanya penyematan variabel**, maka bisa menggunakan pipe. Hanya bisa dengan `while` loop saja.
1. Jika melakukan suatu proses **dengan adanya penyematan variabel** didalamnya, maka pipe tidak bisa digunakan. Bisa menggunakan `for` atau `while` loop.

{{% callout %}}
Semua proses yang berjalan didalam pipeline, itu berjalan pada subshell. Pada subshell, semua penyematan variabel hanya berdampak pada lingkup subshell itu sendiri dan tidak terekspos ke shell diatasnya.
{{% /callout %}}

### Loop menggunakan pipe

Menggunakan kombinasi `while read` dengan inputan dari pipe:

```bash
cat file.txt | while read -r line; do
    echo "Baris: $line"
    ...
    ...
done
```

Pada contoh diatas, tiap baris yang ada pada file, bisa diakses menggunakan variabel `$line` (bisa diganti sesuai keinginan, `while read -r baris` misalnya) dan berurutan. Dalam bentuk seperti ini, semua penyematan variabel tidak akan tersimpan.

### Loop tanpa pipe

Masih menggunakan kombinasi `while read` dengan inputan dari `<`:

```bash
while read -r line; do
    echo "Baris: $line"
    ...
    ...
done < file.txt
```

Metode yang ini digunakan apabile jika operasi yang ingin dilakukan terdapat penyematan variabel berdasarkan data yang ada pada baris yang dibaca.

Loop `for` juga bisa digunakan, namun secara asal (_default_), akan bermasalah jika baris yang ada di dalam file itu lebih dari 1 kata (dipisahkan dengan spasi):

```bash
for line in $(cat file.txt); do
    echo "Baris: $line"
    ...
    ...
done
```

Untuk mengatasi limitasi diatas, perlu mengatur variabel internal `IFS` (_Internal Field Separator_) untuk memberitahu `for`, pada momen apa suatu baris itu berganti.

```bash
OIFS=$IFS  # Menyimpan value awal IFS
IFS=$'\n'  # Mengganti isi IFS menjadi newline (ganti baris)
for line in $(cat file.txt); do
    echo "Baris: $line"
    ...
    ...
done
IFS=$OIFS # Mengembalikan value IFS menjadi seperti awal
```

{{% callout %}}
Secara asal, variabel `IFS` berisikan 1 spasi. Sehingga `for` (dan perintah lain) akan memaknai ganti "kolom" ketika bertemu spasi.
{{% /callout %}}

Contoh tanpa mengganti `IFS`:
```bash
$ cat file.txt
baris satu
baris dua
baris tiga
baris empat

$ for line in $(cat file.txt); do
> echo "$((++i)): $line"
> done
1: baris
2: satu
3: baris
4: dua
5: baris
6: tiga
7: baris
8: empat
```

Hasilnya adalah setiap barisnya akan terpecah (_split_) pada spasi, sehingga menjadi tidak sesuai harapan.

Contoh dengan mengganti `IFS`:
```bash
$ cat file.txt
baris satu
baris dua
baris tiga
baris empat

$ OIFS=$IFS
$ IFS=$'\n'
$ for line in $(cat file.txt); do
> echo "$((++i)): $line"
> done
1: baris satu
2: baris dua
3: baris tiga
4: baris empat
```

## Membaca file csv

File `csv` adalah suatu file yang tiap barisnya berisikan kolom-kolom yang dipisahkan (_delimiter_ atau _separator_) dengan tanda koma `,` (atau _separator_ yang lain, seperti spasi, tab, dsb).

Contoh file csv:

```bash
$ cat file.csv
Fulan,Surabya,24,Swasta
Joko,Pacitan,30,PNS
Ahmad,Tangerang,29,Swasta
Abbas,Malang,40,Wiraswasta
```

Misal, kita ingin membuat script untuk meng-_generate_ alamat email berdasarkan data diatas. Maka, kita bisa menggunakan kombinasi `while read` loop dan IFS untuk memisahkan masing-masing kolomnya.

```bash
while IFS=, read -r nama kota umur pekerjaan; do
    echo "Email: ${nama,,}.${kota,,}@mail.co.id"
done < file.csv
```

{{% callout %}}
Sintaks `${variabel,,}` merupakan salah satu dari fungsi _Parameter Expansion_ dari Bash untuk mengubah isi variabel menjadi huruf kecil seluruhnya. Untuk menjadikan seluruhnya huruf kapital, gunakan `${variabel^^}`. _InsyaAllah_, akan datang pembahasan tersendiri mengenai _Parameter Expansion_ Bash.
{{% /callout %}}

{{% callout %}}
Sintaks `read -r nama kota ...` diatas, akan menghasilkan variabel dengan nama-nama tersebut, sesuai dengan urutan di dalam file inputnya.

Misal, baris-baris dalam file input tersebut memiliki kolom-kolom berikut:
```
No Nama Email Password
```

maka, pada sintaks `read` ke-empat kolom tersebut bisa langsung disematkan kepada variabel dengan cara menyebutkan nama variabel sasuai dengan urutannya:

```bash
read -r no nama email password
```

Variabel bisa diberi nama secara bebas, seperti:
```bash
read -r kol1 kol2 kol3 kol4
```
{{% /callout %}}

Contoh diatas akan menghasilkan output berikut:

```bash
$ while IFS=, read -r nama kota umur pekerjaan; do
>   echo "Email: ${nama,,}.${kota,,}@mail.co.id"
> done < file.csv
Email: fulan.surabya@mail.co.id
Email: joko.pacitan@mail.co.id
Email: ahmad.tangerang@mail.co.id
Email: abbas.malang@mail.co.id
```

Kemudian, jika file csv-nya memiliki _delimiter_ berupa tab. Maka `IFS` bisa diset menjadi `IFS=$'\t'`.

# Kesimpulan

Pada Bash, melakukan pembacaan baris-per-baris dari suatu file, bisa menggunakan `for` dan `while` loop. Untuk `while` loop, bisa melakukan looping pada pipe atau langsung menggunakan `stdin` (`<`). 

Namun jika menggunakan pipe, maka penyematan variabel akan diabaikan.

Sekian artikel kali ini, semoga bermanfaat _barakallahufiikum_.
