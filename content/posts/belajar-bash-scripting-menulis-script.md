---
title: "Belajar Bash Scripting: Menulis Script"
date: 2022-05-28T16:25:00+07:00
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Setelah mempelajari mengenai elemen-elemen dasar pada suatu _script_, kali ini saya akan menjelaskan bagaimana membuat file _script_.

Jika Anda ingin mempelajari ulang mengenai elemen-elemen yang dimaksud, bisa Anda pelajari melalui tautan-tautan dibawah ini:

1. [Konsep dasar _Shell Variables_](https://blog.annahri.com/posts/belajar-bash-scripting-variabel/)
1. [Konsep dasar _Shell Conditionals_](https://blog.annahri.com/posts/belajar-bash-scripting-conditional/)
1. [Konsep dasar _Shell Loops_](https://blog.annahri.com/posts/belajar-bash-scripting-loop/)
1. [Konsep dasar _Shell Functions_](https://blog.annahri.com/posts/belajar-bash-scripting-functions/)

Pada dasarnya, shell script merupakan file yang berisi serangkaian instruksi shell yang disusun sedemikian rupa untuk tujuan tertentu. 

Diantara tujuan tersebut dapat berupa instruksi otomasi proses, CLI (_command line interface_) _tool_, program _wrapper_ sebagai ekstensi suatu program lain yang sudah ada, dan semisalnya.

## Struktur Dasar

### Ekstensi file

Pada umumnya, file script diberi akhiran `.sh` namun sebenarnya ini tidak wajib. Hanya saja ini merupakan praktik yang umum dilakukan.

Namun, ini akan memudahkan _user_ untuk bisa mengetahui bahwa suatu file adalah file script atau bukan, bisa dilihat dari ekstensi `.sh`-nya.

### Shebang / Hashbang

Suatu file script selalu diawali dengan yang namanya _SHEBANG_ atau _HASHBANG_. Contohnya adalah seperti berikut ini:

```bash
#!/usr/bin/bash
```

Fungsi dari _shebang_ ini adalah untuk memberitahu _shell_ bahwa untuk menjalankan _script_ tersebut, program yang digunakan adalah bash (atau yang lainnya).

Maka, _shebang_ ini ditentukan di baris paling pertama, diawali dengan karakter pagar dan pentung `#!`. Kemudian diikuti dengan _path_ menuju file _binary_ program/shell yang ingin ditulis scriptnya.

Misalnya, jika hendak menulis script untuk Bourne Shell (sh), maka:

```sh
#!/bin/sh
```

atau Z Shell:

```zsh
#!/bin/zsh
```

atau Python 3.10:

```python
#!/usr/bin/python3.10
```

atau awk:

```awk
#!/usr/bin/awk -f
```

dan sebagainya.

Bagaimana jika _path_ untuk shell Bash (misalnya), lokasinya tidak menentu? Bisa jadi ada di `/bin/bash` atau `/usr/bin/bash` atau `/usr/local/bin/bash` ?

Praktik yang disarankan adalah menggunakan program `env`, contoh:

```bash
#!/usr/bin/env bash
```

Dengan demikian, `env` akan mencari _path_ yang valid untuk shell Bash tanpa harus menentukannya secara _hard-coded_.

### Memiliki izin eksekusi (_executable_)

Suatu file script perlu memliki izin eksekusi (_executable_) agar bisa dieksekusi langsung tanpa harus memanggil _shell_ yang terkait.

``` bash
chmod +x file.sh
```

Jika tidak demikian, ketika hendak dieksekusi, maka shell akan mengeluarkan error "Permission denied".

```bash
$ ./script.sh
bash: ./script.sh: Permission denied
```

Atau, perlu memanggil program/shell yang akan digunakan terlebih dahulu sebelum mengeksekusi. Contohnya:

```bash
$ bash script.sh
# atau
$ python script.py
...
```

## Contoh file script

Script `hello-world.sh`:

```bash
#!/usr/bin/env bash

echo "Hello, World!"
```

Script `oraganize.sh`, berfungsi untuk menata file berdasarkan ekstensi file:
```bash
#!/usr/bin/env bash

if [[ -z "$1" ]]; then
    echo "Penggunaan: $(basename $0) /path/to/dir"
    exit
fi

if [[ ! -d "$1" ]]; then
    echo "Direktori tidak valid"
    exit 1
fi

find "$1" -maxdepth 1 -type f | while read file; do
    extension=${file##*.}
    new_folder="${1%/}/${extension}"
    mkdir -p "$new_folder" \
        && mv "$file" "$new_folder"
done
```

---

Semoga bermanfaat, _barakallahufiikum_.