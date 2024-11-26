---
title: "Belajar Bash Scripting: Loop"
date: 2022-05-08T14:21:00+07:00
draft: false
tags:
- bash
- scripting
categories:
- Belajar
---

![Cover](/images/bss-loop.jpg)

![Bismillah](/images/bismillah-2.png#center)

Artikel ini akan menjelaskan mengenai bagaimana cara untuk melakukan LOOP alias menjalankan sekumpulan _command_ secara berulang-ulang dengan kondisi tertentu.

## FOR loop

FOR loop pada shell Bash sifatnya sama seperti _foreach_ pada keumuman bahasa pemrograman. Yaitu melakukan perulangan tanpa adanya _counter_, alias suatu perulangan tidaklah diketahui ke-berapa perulangan tersebut sedang terjadi. Atau sederhananya tidak ada indeks pada setiap perulangan.

Namun dengan pengecualian jika FOR loop tersebut dideklarasikan menggunakan indeks. Akan tiba contohnya pada penjelasan di bawah.

### Struktur dasar

Variabel `$var` dibawah ini adalah variabel yang berisi _string_-_string_ yang dipisahkan dengan spasi.

```bash
$ var="a b c d"
$ for VARIABEL in $var; do
      echo "Huruf $VARIABEL"
  done
Huruf a
Huruf b
Huruf c
Huruf d
```

Sebenarnya, sintaks `in VARIABEL` adalah opsional. Jika sintaks tersebut tidak disebutkan, maka Bash akan mengambil dari _Positional Argument_.

```bash
$ set -- 1 2 3 4
$ for angka; do
      echo "Angka ke-$angka"
  done
Angka ke-1
Angka ke-2
Angka ke-3
Angka ke-4
```

Pada bagian `in` juga bisa diberikan suatu command yang outputnya akan digunakan untuk _looping_.

```bash
$ for x in $(seq 10); do
      echo "Urutan ke $x"
  done
Urutan ke 1
Urutan ke 2
...
Urutan ke 10
```

### Model bahasa C (_C-Style FOR loop_)

_C-style loop_ sesuai dengan namanya, yaitu model FOR loop menggunakan model bahasa pemrograman C.

```bash
$ for ((i = 0; i < 10; i++)); do
      echo "Angka $i"
  done
Angka 0
Angka 1
Angka 2
...
Angka 9
```

## WHILE loop

Sama seperti keumuman bahasa pemrograman, WHILE loop ini akan terus diulang-ulang selama kondisi yang diujikan masih benar. 

WHILE loop ini memberikan fleksibilitas yang lebih daripada FOR loop. Ketika FOR loop hanya terbatas pada tiap yang argumen diberikan, WHILE loop bisa digunakan lebih dari itu.

### Struktur dasar

Agak mirip secara struktur, yaitu dimulai dari `while kondisi; do ...; done`. Setiap loop pada bash, pasti ada sintaks `do .. done`.

```bash
while [[ $x -lt 10 ]]; do
    command ...
    ((x++))
done
```

### _Looping_ terus-menerus

Karena sifat WHILE loop ini yang akan terus melakukan perulangan ketika kondisi yang diujikan bernilai benar, maka jika ia diberikan sintaks yang selalu bernilai benar seperti **true** misalnya, maka akan dihasilkan _infinite_ loop.

```bash
while true; do
    command ...
    if kondisi; then
        break
    fi
done

# atau
while :; do
    command ...
    if kondisi; then
        break
    fi
done
```

### Mengulang command terus-menerus

Sebagaimana yang telah dicontohkan pada bagian sebelumnya (looping terus-menerus), sebenarnya sintaks **true** dan **:** merupakan sebuah command yang valid jika dijalankan pada shell secara langsung. Sehingga, WHILE loop bisa diletakkan padanya suatu command yang memiliki exit status. Jika exit status command tersebut adalah _non-zero_ alias tidak bermasalah, maka loop akan berlanjut.

```bash
while sleep 1; do
    command
    command
done

# atau

while ssh -T HOST uptime; do
    sleep 1
done
```

### Membaca file, variabel atau output command dengan WHILE loop

WHILE loop bisa dimanfaatkan untuk membaca baris tiap baris dari sebuah file atau output suatu command atau pipeline. Ini berguna jika kita hendak memproses tiap baris pada output tersebut.

#### Loop dari pipe

Melakukan piping semacam dibawah ini bisa dilakukan selama didalam loop tidak ada proses penyematan variabel. Karena, pipeline berlangsung didalam subshell sehingga, variabel yang sama, yang dipanggil dari dalam loop ini, adalah variabel yang berbeda.
Namun, jika aktifitas loop tidak ada kegiatan mengisi isi variabel (yang sudah dideklarasi sebelum pipe ini) dan sebagainya, maka loop ini bisa dipakai.

```bash
find /dir -type f -name 'pattern' | while read line; do
    file=$(basename "$line")
    new_name="new-${file}"
    echo "Renaming $file to $new_name"
    mv "$file" "$new_name"
done 
```

#### Loop dengan input dari file, variabel atau _process substitution_

_Process Substitution_ memungkinkan kita untuk menginputkan serangkaian _command_ (_pipeline_).

Dalam kasus loop ini, WHILE mendapat input tidak dari pipe sehingga semua yang terjadi didalamnya, tidak berada di dalam subshell. Yang artinya, semua penyematan variabel di dalam loop, akan benar-benar tersematkan.

```bash
while read line; do
    ...
done < file

# atau
while read line; do
    ...
done <<< "$var"

# atau
while read line; do
    ...
done < <(command)
```

## UNTIL loop

UNTIL loop adalah kebalikan dari WHILE loop. Yaitu melakukan perulangan selama kondisi yang diberikan tetap bernilai salah.

### Struktur dasar

Sama dengan WHILE loop, cukup menggantinya dengan UNTIL

```bash
until kondisi; do
    command
done

# misal
until [[ $# -eq 0 ]]; do
    : proses loop
done
```

Contoh penggunaan dalam script:

```bash
# Loop akan exit jika input file yang diberikan adalah valid
until [[ -f "$file" ]]; do
    read -p "Masukkan input file > " file
done
```

## Sintaks _break_ dan _continue_

Sintaks _break_ dan _continue_ tidak terbatas hanya pada WHILE loop saja, tetapi juga bisa digunakan pada UNTIL dan FOR loop juga. Kedua sintaks ini berfungsi untuk mengatur alur suatu loop, baik untuk menghentikan loop (keluar dari loop) atau meloncati suatu iterasi.

### Break

Dalam _infinite_ loop, perlu ada kondisi yang dimana dia akan membuat loop tersebut berhenti. Untuk dapat keluar dari loop, gunakan sintaks **break**. Contoh untuk ini sudah dijelaskan pada bagian WHILE loop sebelumnya. Atau misal pada kasus script interaktif seperti contoh berikut:

```bash
while read -p "Masukkan angka 1-10 > " angka; do
    if [[ $angka -gt 10 || $angka -lt 1 || $angka =~ [^0-9]+ ]]; then
        echo "Input tidak valid!"
    else
        echo "Angka terpilih: $angka"
        break
    fi
done
```

### Continue

Sedangkan jika dalam suatu loop ada sebuah kondisi yang jika kondisi tersebut terpenuhi, maka perputaran saat itu diloncati, maka hal tersebut bisa dilakukan dengan sintaks **continue**.

```bash
for ((i = 1; i < 10; i++)); do
    if ((i % 2 != 0)); then
        continue
    fi
    ...
done
```

## Loop non-standar

Pipeline jika dirangkai sedemikian rupa juga akan bisa melakukan loop. Contohnya adalah gabungan dari `seq` atau Bash _brace expansion_ yang dipipe kepada `xargs` seperti contoh berikut ini:

```bash
seq 10 | xargs -n1 -I{} echo "Putaran ke-{}"
# atau serupa dengan
for i in {1..10}; do
    echo "Putaran ke-$i"
done
```

Mencari bilangan genap dari 1 sampai 100 :
```bash
echo {1..100} | xargs -n1 -I{} bash -c 'if (( {} % 2 == 0 )); then echo {}; fi'
```

---

Semoga bermanfaat, _insyaAllah_.
