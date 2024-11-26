---
title: "Belajar Bash Scripting: Arguments"
date: 2022-07-30T21:38:09+07:00
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel kali ini, saya akan membahas mengenai shell argumens, yaitu bagaimana agar kita dapat memasukkan parameter tertentu saat memanggil script yang telah kita tulis atau saat memanggul fungsi yang telah kita definisikan.

# Parameter/argumen

Mari kita lihat pada *command* `cp` berikut ini:

```bash
cp -f file1 file2
```

_Command_ diatas bisa diartikan dengan "menyalin secara paksa `file1` sebagai `file2`".

Paksaan yang dimaksud adalah jika `file2` sebelumnya telah ada, maka langsung menimpa `file2` tersebut dengan `file1`tanpa memberitahu user terlebih dahulu. Pada *command* `cp`, struktur dasar penggunaannya adalah:

```bash
cp [OPSI] <FILE SUMBER> <FILE TUJUAN>
```

Sintaks yang berada dalam kurung kotak, merupakan parameter opsional, alias boleh diberi boleh tidak. Kemudian dalam kurung lancip, adalah wajib. Maka pemberian parameter `FILE SUMBER` dan `FILE TUJUAN` adalah wajib.

# Interpretasi shell bash

Jika sintaks diatas kita bedah satu persatu, maka shell bash akan menginterpretasikan masing-masing parameter/argumen yang diinputkan sebagai variabel-variabel yang bisa diakses. Variabel-variabel tersebut dimulai dari variabel `$0` yang itu merupakan nama program/script yang dijalankan, kemudian `$1` yang merupakan argumen pertama, hingga `$n` (dimana n adalah jumlah variabel).

Sehingga, pada contoh diatas, maka bisa didapatkan variabel-variabel berikut:

1. `$0` adalah `cp`
2. `$1` adalah `-f`
3. `$2` adalah `file1`, dan
4. `$3` adalah `file2`

# Percobaan

Sebagai pembuktian, mari dicoba dengan script `coba.sh` sederhana berikut ini:

```bash
#!/usr/bin/env bash

echo "\$0 = $0"
echo "\$1 = $1"
echo "\$2 = $2"
```

Hasil tesnya adalah

```bash
$ ./coba.sh halo bismillah
$0 = ./coba.sh
$1 = halo
$2 = bismillah
```

Ini juga bisa digunakan untuk fungsi:

```bash
#!/usr/bin/env bash

main() {
	echo "\$0 = $0"
	echo "\$1 = $1"
	echo "\$2 = $2"
}

main "uji" "coba"
```

Maka saat dijalankan, akan mengeluarkan output:

```bash
$ ./coba.sh
$0 = ./coba.sh
$1 = uji
$2 = coba
```

{{< callout >}}
Disini perbedaannya adalah, <code>$0</code> akan selalu merujuk pada program/shell script yang dijalankan, bukan fungsi yang dipanggil.
{{< /callout >}}

# Contoh script

```bash
#!/usr/bin/env bash

usage() {
    echo "Usage: $(basename $0) <nama>"
    exit
}

if [[ -z "$1" ]]; then
    echo "Masukkan namamu!"
    usage
fi

for _ in {1..3}; do
    echo "[$_] Namaku $1"
done
```

```bash
$ ./sapa.sh
Masukkan namamu!
Usage: sapa.sh <nama>

$ ./sapa.sh Fulan
[1] Namaku Fulan
[2] Namaku Fulan
[3] Namaku Fulan
```

# Variabel $#, $@ dan $*

## $#

Variabel `$#` digunakan untuk mengetahui jumlah parameter yang diberikan. Misalnya:

```bash
$ coba.sh a b c d
# maka nilai $# adalah 4
```

Ini bisa dimanfaatkan salahsatunya untuk melakukan validasi apakah user memberikan argumen atau tidak:

```bash
#!/usr/env/bin bash

if [[ $# -eq 0 ]]; then
	echo "Harap berikan argumen!"
	exit 1
fi

echo "Jumlah argumen: $#"
```

Maka saat dicoba:

```bash
$ ./coba.sh
Harap berikan argumen!

$ ./coba.sh aa bb cc dd ee
Jumlah argumen: 5
```

## $@ dan $*

Dua variabel ini digunakan untuk mengakses seluruh argumen yang diberikan.

### Contoh 1

```bash
#!/usr/bin/env bash

echo "Percobaan \$@"
echo "Arg: [ $@ ]"

echo "Percobaan \$*"
echo "Arg: [ $* ]"
```

```bash
$ ./coba.sh haha hehe hoho huhu
Percobaan $@
Arg: [ haha hehe hoho huhu ]
Percobaan $*
Arg: [ haha hehe hoho huhu ]
```

Pertanyaannya, jika outputnya sama, mengapa dibedakan? Pada contoh diatas, tidak akan ketara perbedaannya. Mari dilihat pada contoh berikutnya.

### Contoh 2

```bash
#!/usr/bin/env bash

printf '== %s ==========\n' "Menggunakan \$@"
printf 'arg: %s\n' "$@"
echo 
printf '== %s ==========\n' "Menggunakan \$*"
printf 'arg: %s\n' "$*"
```

```bash
$ ./coba.sh 11 22 33 44
== Menggunakan $@ ==========
arg: 11
arg: 22
arg: 33
arg: 44

== Menggunakan $* ==========
arg: 11 22 33 44

$ ./coba.sh "haha" "ha~ha~" "haa haa" "hue hue" "hoo
hoo"
== Menggunakan $@ ==========
arg: haha
arg: ha~ha~
arg: haa haa
arg: hue hue
arg: hoo
hoo

== Menggunakan $* ==========
arg: haha ha~ha~ haa haa hue hue hoo
hoo
```

Maka:

1. Variabel `$@` akan expand dan masing-masingnya menjadi argumen.
2. Variabel `$*` akan expand menjadi satu string utuh.

Nah, untuk bisa memahami kesimpulannya, maka saya coba berikan contoh script sederhana berikut ini.

### Contoh 3

```bash
#!/usr/bin/env bash
# command file berfungsi untuk mengetahui jenis "file"
# command tersebut mendukung multi argumen, file file1 file2 ... fileN

printf '== %s ==========\n' "Menggunakan \$@"
file -- "$@" 

printf '== %s ==========\n' "Menggunakan \$*"
file -- "$*"
```

```bash
# Membuat file file1 s/d file3, pembahasan brace expansion
$ touch file{1..3} 
$ ./coba.sh file1 file2 file3
== Menggunakan $@ ==========
file1: empty
file2: empty
file3: empty
== Menggunakan $* ==========
file1 file2 file3: cannot open `file1 file2 file3' (No such file or directory)
```

Bisa diperhatikan, pada `$@`, sintaks `file -- "$@"`, input `./coba.sh file1 file2 file3` akan terexpand sesuai dengan masing-masing argumennya  `file -- "file1" "file2" "file3"` dengan total argumen sebanyak 4.

Sedangkan pada `$*` input yang sama, akan terekspand menjadi `file -- "file1 file2 file3"`. Jika dipecah kembali, maka akan didapatkan total 2 argumen saja:

1. Argumen 1: `--`
2. Argumen 2: `file1 file2 file3`

Sehingga, secara harfiah, file yang bernama “file1 file2 file3” memang tidak ada pada folder tersebut.

---

Semoga bermanfaat, *barakallahufiikum*.
