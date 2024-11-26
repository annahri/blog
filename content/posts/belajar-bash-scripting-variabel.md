---
title: "Belajar Bash Scripting: Variabel (Update)"
date: 2022-02-28T13:21:57+07:00
draft: false
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel ini, saya akan menjelaskan bagaimana cara membuat variabel, array, dan lain-lain yang berkaitan, pada shell Bash.

Seperti yang sudah pernah dibahas pada artikel Belajar Bash Scripting lain, bahwa tentunya Bash Scripting yang dibahas ini tidaklah POSIX-compliant atau istilahnya _Bashism_.

## Aturan penamaan

Dalam membuat variabel atau array, ketentuan dalam pemberian nama adalah sebagai berikut:

1. Tidak diawali dengan angka.
1. Tidak mengandung tanda pentung, tanda @, tanda pagar # dan asterisk (\*).
1. Variabel dengan huruf kapital seluruhnya, dikhususkan untuk variabel internal shell dan varibel _environment_.

Contoh:

```bash
1variabel # salah
!variabel # salah
@variabel # salah
vari*abel # salah
#variabel # salah
vari#abel # salah
vari!abel # salah
vari@abel # salah

SHELL     # sudah digunakan sebagai environment variable
HOSTNAME  # sudah digunakan sebagai environment variable
REPLY     # sudah digunakan dalam BASH internal variable

variabel  # benar
Variabel  # benar
vAriabel  # benar
vari_abel # benar
_variabel # benar
vari4bel  # benar
variabel1 # benar
```

## Variabel

### Membuat variabel

Pada Bash shell, variabel bisa dibuat dengan cara seperti ini:

```bash
# Tanpa petik
var=string

# Petik
var="string"

# Petik tunggal
var='string'
```

Perbedaan mengenai penggunaan tanda petik dan petik tunggal akan dijelaskan nanti.

Perlu diperhatikan bahwa untuk membuat variabel, tidak boleh ada spasi diantara tanda `=`, misalnya:

```bash
# Salah
var = "string"
var= "string"
var ="string"
```

### Melihat isi variabel

Isi variabel bisa dilihat dengan _command_ `echo` atau `printf`:

```bash
$ var="Bismillah"
$ echo "$var"
Bismillah
$ printf '%s\n' "$var"
Bismillah
```

### Mengisi variabel dari _output_ suatu _command_

Suatu variabel bisa diisi dengan _output_ dari suatu _command_ tertentu dengan menggunakan 2 cara:

1. Menggunakan _backticks_ \`...\`
2. Menggunakan _command substitution_ `$(...)`

Contoh:

```bash
var=`uname -r`
var=$(uname -r)
```

Cara kedua lebih disarankan karena 1) _backticks_ sudah dianggap telah _deprecated_ (ditinggalkan), 2) _command substitution_ memungkinkan untuk digunakan secara berlapis. Misal:

```bash
var=$(ls -1 $(pwd))
```

### Menambah isi variabel (_append_)

Menambah isi variabel (yang berupa _string_) bisa dilakukan seperti membuat variabel tersebut namun tidak sekedar dengan tanda `=` saja, tetapi dengan `+=`, contoh:

```bash
$ var=Hello
$ echo "$var"
Hello
$ var+=", World!"
$ echo "$var"
Hello, World!
```

### Kalkulasi aritmatis pada variabel (_integer_)

Membuat variabel berupa hasil kalkulasi 2 bilangan bulat (atau lebih), bisa dilakukan dengan dua tanda kurung `$((...))`

```bash
$ bil1=5
$ bil2=9
$ hasil=$((bil1 + bil2))
$ echo $hasil
14
$ kali=$((bil1 * bil2))
$ echo $kali
45
```

### Variabel _readonly_

Variabel _readonly_ adalah variabel yang tidak bisa berubah setelah ditetapkan isinya, sama dengan konsep variabel konstan pada keumuman bahasa pemrograman.

```bash
# cara 1
$ readonly var=string

# cara 2
$ declare -r var=string

$ var=strong
bash: var: readonly variable
```

### Variabel lokal

Variabel lokal adalah variabel yang ruang lingkupnya hanya pada suatu fungsi saja. Ketika dipanggil diluar fungsi tersebut, maka variabel yang dipanggil tersebut adalah variabel yang berbeda.

```bash
$ var=xyz
$ fungsi() {
  local var=abc # atau declare var=abc
  echo "$var"
}
$ echo "$var"
xyz
$ fungsi
abc
```

### Membuat variabel dengan `printf`

_Command_ `printf` adalah salah satu _tool builtin_ pada shell Bash. Selain berfungsi seperti `echo`, `printf` juga bisa digunakan untuk menetapkan suatu variabel.

```bash
$ printf -v var '%s\n' string
$ echo "$var"
string
```

### Membuat variabel berdasarkan input pengguna

Yang dimaksud disini adalah membuat variabel dengan cara interaktif, yaitu membutuhkan inputan dari pengguna shell. Seperti "Siapa namamu?", kemudian pengguna mengisikan namanya.

```bash
$ read -r -p "Siapa namamu? " nama
Siapa namamu? Fulan bin Fulan
$ echo "Namaku: $nama"
Namaku: Fulan bin Fulan
```

Flag `-r` diatas berfungsi untuk menghindari _backslash_ dari melakukan _escaping_ karakter apapun. Sedangkan `-p` berfungsi untuk mem-print serangkaian kata yang sudah ditentukan sebelum melakukan `read`. Ini guna menghemat dari penggunaan `echo` sebelum `read`.

Kasus penggunaan tool `read` ini diantaranya adalah seperti melakukan konfirmasi _Yes/No_ sebelum menjalankan _command_ tertentu; atau membuat script yang interaktif, dan sebagainya.

## Array

Diantara shell-shell yang ada, diantaranya ada yang mendukung array dan ada yang tidak. Bash adalah salah satu yang mendukung array. Jenis array yang bisa dibuat pada shell bash adalah *Associative Array* dan *Indexed Array*

### Array terindeks (_indexed_ array)

Yang dimaksud dengan _indexed_ array adalah array yang tiap anggotanya memiliki nomor indeks masing-masing yang dimulai dari nol (_zero indexed_). Cara membuatnya adalah seperti berikut:

```bash
# cara 1
array=(satu dua tiga)

# cara 2
declare -a array=(satu dua tiga)
```

### Array asosiatif (_associative_ array)

_Associative_ array, adalah jenis array yang memiliki _key_ dan _value_. Cara membuat:

```bash
$ declare -A array=([nama]=Fulan [kota]=Surabaya [pendidikan]=S1)
$ array[status]="Lajang"
$ array[pekerjaan]="Swasta"
```

### Melihat isi array

Untuk melihat seluruh isi array:

#### _Indexed array_

```bash
$ echo "${array[@]}"
satu dua tiga
```

Untuk mengakses salah satu anggota array:

```bash
$ echo "${array[1]}"
dua
```

#### _Associative array_

Hampir sama dengan array terindeks, untuk melihat _value_ tiap anggota array asosiatif:

```bash
$ echo "${array[@]}"
Fulan Surabaya S1 Lajang Swasta
```

Untuk melihat seluruh _key_ dari tiap anggota array:

```bash
$ echo "${!array[@]}"
nama kota pendidikan status pekerjaan
```

Untuk melihat _value_ dari _key_ tertentu:

```bash
$ echo "${array[status]}"
Lajang
```

#### Mengakses array menggunakan _loop_

Mengakses array menggunakan _loop_ memungkinkan untuk melakukan sesuatu terhadap tiap-tiap isi array, baik itu array asosiatif atau terindeks.

```bash
# _Indexed_ array
$ arr=(satu dua tiga empat)
$ for angka in ${arr[@]}; do echo "Putaran ke-$angka"; done
Putaran ke-satu
Putaran ke-dua
Putaran ke-tiga
Putaran ke-empat

# Associative array
$ declare -A arr=([nama]=Alan [kota]=Surabaya [asal]=Jakarta)
$ for key in ${!arr[@]}; do echo "Data $key => ${arr[$key]}"; done
Data nama => Alan
Data kota => Surabaya
Data asal => Jakarta
```

### Menambah isi array

Cara menambah isi array adalah seperti berikut:

```bash
$ array+=(empat lima)
$ echo "${array[@]}"
satu dua tiga empat lima

# Untuk array asosiatif juga sama
$ array+=([key]=value [key2]=value2)
```

### Mengubah string menjadi array

```bash
$ var="satu dua tiga empat lima"
$ array=( $var )
# atau
$ declare -a array <<< $var
```

Penjelasan mengenai `<<<` ada pada artikel mendatang, _insyaAllah_.

### Menkonversi file yang berisi list menjadi array

Jika ada suatu file yang berisi suatu list (bisa berupa daftar _string_, dsb) dan hendak dikonversi menjadi array, maka bisa menggunakan `mapfile`:

```bash
$ cat file
aaaa
bbbb
cccc
dddd
$ mapfile list < file
$ echo ${list[3]}
dddd
```

### Kasus penggunaan array

Pada sebuah script, menentukan sekumpulan _options_ dari suatu command tertentu. Misalnya, script tersebut menggunakan `rsync` dan ada sekumpulan _options_ dan _flag_ yang sudah ditentukan untuk setiap panggilannya.

```bash
#!/usr/bin/env bash

rsync_options=(--archive --append --bwlimit=512k --progress ...)

rsync() {
    rsync ${rsync_options[@]} ....
}
```

---

*Referensi:*

- https://www.geeksforgeeks.org/bash-script-define-bash-variables-and-its-types
- https://www.shell-tips.com/bash/math-arithmetic-calculation

Semoga bermanfaat, _insyaAllah_.
