---
title: "Belajar Bash Scripting: Conditional"
date: 2022-02-27T20:41:25+07:00
draft: false
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel ini saya akan menjelaskan mengenai bagaimana cara melakukan macam-macam kondisional pada Bash _scripting_. Dan karena ini adalah seri khusus _shell_ Bash, maka tentunya tidak _POSIX-compliant_.

## Kondisional IF

### Struktur dasar

Struktur dasar kondisional _if_ dalam Bash _scripting_ hampir sama dengan keumuman bahasa pemrograman, yaitu seperti dibawah ini:

```bash
if kondisi; then
    command
    command
    ...
fi

# atau

if kondisi
then
    command
    command
    ...
fi
```

Kondisi yang disebutkan diatas bisa berupa ekspresi perbandingan seperti contoh berikut:

```bash
if [[ $var == xyz ]]; then
    command
    command
fi

# atau

if test "$var" == "xyz"; then
    command
    command
fi
```
Atau, bisa berupa command tertentu seperti:

```bash
# Jika command1 sukses maka jalankan command2 dan seterusnya
if command1; then
    command2
    command3
    ...
fi

# Contoh
# Jika variabel $string mengandung substring xyz, maka jalankan command1
if grep -q 'xyz' <<< "$string"; then
    command1
fi
```

### Cara kerja

Kondisional _if_ akan menjalankan command di dalam blok jika kondisi yang diujikan memiliki _exit status_ nol. Kedua contoh diatas ( sintaks `[[ ... ]]` atau `test` dan `grep -q` ) sebenarnya sama saja. Dalam artian bahwa sintaks `[[ ... ]]` sesungguhnya adalah _command_ biasa yang bisa dijalankan diluar blok _if_. Bisa dicoba sendiri pada sesi shell seperti dibawah ini:

```bash
[[ $var == "xyz" ]] && echo true
test "$var" == "xyz" && echo true
```

Maka, jika variabel `$var` sama dengan string `xyz`, maka terminal akan mengeluarkan output "true"

```bash
annahri@asus-zorin:/tmp$ var=xyz
annahri@asus-zorin:/tmp$ [[ $var == "xyz" ]] && echo true
true
annahri@asus-zorin:/tmp$ test "$var" == "xyz" && echo true
true
```

### Ekspresi kondisional / _Conditional expression_

Salah satu contoh ekspresi kondisional adalah seperti yang sudah dijelaskan sebelumnya, yaitu `[[ $var == "xyz" ]]` yang itu merupakan bentuk dari operator string.

Disini saya hanya akan menjelaskan beberapa contoh ekspresi yang sering digunakan. Untuk lebih lengkapnya, bisa dibaca sendiri via command `help test`.

```
annahri@asus-zorin:/tmp$ help test
test: test [expr]
    Evaluate conditional expression.

    Exits with a status of 0 (true) or 1 (false) depending on
    the evaluation of EXPR.  Expressions may be unary or binary.  Unary
    expressions are often used to examine the status of a file.  There
    are string operators and numeric comparison operators as well.
.............
```

#### Operator file

Operator file berfungsi untuk mencari tahu status suatu file. Apakah file tersebut ada, apakah file tersebut adalah suatu direktori, dan lain-lain.

```bash
# Jika $file adalah file reguler
if [[ -f $file ]]; then
    command
fi

# Jika $file adalah direktori
if [[ -d $file ]]; then
    command
fi

# atau

if test -d "$file"; then
    command
fi
```
#### Operator string

Digunakan untuk melakukan perbandingan suatu variabel terhadap variabel lain atau _string_ literal. Seperti yang sudah dijelaskan pada contoh awal misalnya.

```bash
if [[ $string == "asdf" ]]; then
    command
fi

# atau negasinya
if [[ $string != "asdf" ]]; then
    command
fi

# Jika $string kosong alias unset
if [[ -z $string ]]; then
    command
fi

# Jika $string tidak kosong alias memiliki nilai
if [[ -n $string ]]; then
    command
fi
```

#### Operator aritmatik

Sesuai dengan judulnya, maka operator ini berfungsi untuk melakukan perbandingan aritmatis.

```bash
# Jika $var sama dengan 10
if [[ $var -eq 10 ]]; then
    command
fi

# Jika $var tidak sama dengan 10
if [[ $var -ne 10 ]]; then
    command
fi

# Jika $var lebih besar dari 10
if [[ $var -gt 10 ]]; then
    command
fi

# Jika $var lebih besar atau sama dengan 10
if [[ $var -ge 10 ]]; then
    command
fi

# Jika $var lebih kecil dari 10
if [[ $var -lt 10 ]]; then
    command
fi

# Jika $var lebih kecil atau sama dengan 10
if [[ $var -le 10 ]]; then
    command
fi
```

### Kondisi ganda (Operator AND dan OR)

Melakukan evaluasi lebih dari satu kondisi dapat dilakukan dengan operator `&&` (AND) dan operator `||` (OR). Contoh penggunaannya adalah sebagai berikut:

```bash
# Jika $val lebih besar dari 10 dan lebih kecil dari 20
if [[ $val -gt 10 && $val -lt 20 ]]; then
    command
fi

# Jika $string tidak kosong dan $string tidak bernilai "xyz"
if [[ -n $string && $string != "xyz" ]]; then
    command
fi

# Jika $string tidak kosong atau $val lebih dari nol
if [[ -n $string || $val -lt 0 ]]; then
    command
fi
```

Tidak terbatas hanya pada perbandingan variabel, namun operator AND dan OR juga bisa digunakan untuk mengevaluasi 2 _command_ atau lebih.

Biasanya ini digunakan untuk melakukan perintah yang memerlukan prekondisi dari perintah lain. Seperti memastikan suatu file berhasil dibuat terlebih dahulu sebelum melakukan command setelahnya.

```bash
# Jika path /a/b/c berhasil dibuat dan jika file /a/b/c/d berhasil dibuat
if mkdir -p /a/b/c && touch /a/b/c/d; then
    command
fi
```

### Negasi

Untuk melakukan evaluasi yang sebaliknya, maka cukup untuk menambahkan sintaks `!` sebelum memulai ekspresi atau sebelum `[[ ... ]]`. Contoh:

```bash
# Jika $file tidak ada, alias kebalikan dari `-f`
if [[ ! -f $file ]]; then
    command
fi
```

Beda penempatan operator negasi, juga berbeda pula dampaknya. Jika negasi ditaruh sebelum ekspresi, maka tiap-tiap ekspresi yang berada di konteks yang sama (berada pada satu kurung siku `[[ ... ]]`) bisa diberlakukan negasi masing-masing. Namun, jika tanda negasi ditaruh diluar seperti `! [[ ... ]]` maka negasi diberlakukan setelah seluruh ekspresi yang ada didalamnya dievaluasi.

```bash
# Masing-masing
if [[ ! -f $file || ! -d $dir ]]; then
    command
fi

# Keseluruhan
if ! [[ -z $string || -f $file ]]; then
    command
fi
```

### IF berlapis/_nested_

Kondisional if tentu saja bisa dilakukan secara berlapis alias _nested_.

```bash
if konsisi 1; then
    command

    if kondisi 2; then
        command
        ...
    fi
fi
```

### Bentuk IF ELSE

Kondisional IF ELSE jika ekspresi yang dievaluasi tidak terpenuhi maka ELSE akan dijalankan

```bash
# Jika kondisi 1 terpenuhi maka command1 akan dijalankan, jika tidak maka command2 yang akan dijalankan
if kondisi 1; then
    command1
else
    command2
fi
```

### bentuk IF ELSE IF

Kondisional IF ELSE IF merupakan bentuk evaluasi ekspresi yang berkelanjutan tetapi perintah-perintah yang ada di dalam kondisi tersebut yang akan dijalankan, alias tidak akan menjalankan 2 kondisi secara bersamaan.

```bash
# Jika kondisi 1 terpenuhi maka command1 dijalankan, jika tidak, 
# maka jika kondisi 2 terpenuhi, maka command2 akan dijalankan, namun jika tidak
# maka jika kondisi 3 terpenuhi maka command3 akan dijalankan
# tetapi jika seluruh kondisi tidak terpenuhi, maka command4 akan dijalankan

if kondisi 1; then
    command1
elif kondisi 2; then
    command2
elif kondisi 3; then
    command3
else
    command4
fi
```

Pada skenario diatas, jika variabel yang dievaluasi adalah variabel yang sama dan hendak mengevaluasi atas nilai-nilai tertentu, maka kondisional CASE-lah yang lebih cocok untuk digunakan. Contohnya sebagai berikut:

```bash
if [[ $string == "a" ]]; then
    command1
elif [[ $string == "b" ]]; then
    command2
elif [[ $string == "c" ]]; then
    command3
.
.
.
elif [[ $string == "z" ]]; then
    commandX
else
    commandZ
fi
```

Bisa dilihat pada contoh diatas bahwa akan banyak pengulangan yang sebetulnya tidak perlu jika menggunakan kondisional CASE.

## Kondisional CASE

Kondisional CASE digunakan untuk melakukan evaluasi terhadap suatu variabel atau output command tertentu seperti pada contoh sebelumnya. Evaluasi yang dilakukan adalah ekspresi apakah variabel tersebut bernilai sekian atau sekian.

### Struktur dasar

Kondisional CASE bisa memiliki beberapa bentuk/model dan merupakan preferensi masing-masing individu, tidak ada benar/salah.

Tetapi pada dasarnya, kondisional CASE diawali dengan `case VARIABEL in` kemudian diikuti dengan kondisi-kondisi. Tiap kondisi dibuka dengan `PATTERN)` (perhatikan tanda tutup kurung di akhir) dan ditutup dengan tanda `;;`. Dan diakhiri dengan `esac`.

Bentuk yang umum adalah seperti ini:

```bash
# Jika $string bernilai a, maka command1 dijalankan
# Jika $string bernilai b, maka command2 dijalankan
# Jika $string bernilai c, maka command3 dijalankan
# namun jika tidak seluruhnya, maka commandZ akan dijalankan

case "$string" in
    a)
        command1
        ;;
    b)
        command2
        ;;
    c)
        command3
        ;;
    *)
        commandZ
        ;;
esac
```

Ada juga yang suka menulis blok CASE seperti ini, jika command untuk tiap kondisi tidaklah panjang.

```bash
case "$string" in
    a) command1 ;;
    b) command2 ;;
    c) command3 ;;
    *) commandZ ;;
esac
```

### Operator OR

Satu kondisi dalam CASE bisa diberi operator OR, contoh:

```bash
case "$string" in
    a|b) 
        command1
        ;;
    c|d)
        command2
        ;;
    *)
        commandZ
        ;;
esac
```

### Operator Wildcard *

Operator wildcard * digunakan untuk mewakili satu atau lebih karakter. Contoh:

```bash
case "$string" in
    # Jika diawali dengan a
    a*)
        command1
        ;;
    # Jika diawali dengan b dan diakhiri dengan c
    b*c)
        command3
        ;;
    # Jika diakhiri dengan z
    *z)
        command4
        ;;
    # Jika seluruh pattern tidak cocok
    *)
        commandZ
        ;;
esac
```

Perlu diperhatikan bahwa penempatan kondisi CASE sangat penting. Jika suatu kondisi/pattern yang bersifat lebih umum diletakkan lebih awal daripada yang lebih khusus, maka kondisi yang lebih khusus tersebut tidak akan pernah terevaluasi. Contoh:

```bash
# Pola a*z sudah terwakili dengan pola sebelumnya (a*)
# sehingga tidak akan terevaluasi.

case "$string" in
    a*)
        command1
        ;;
    a*z)
        command2
        ;;
esac
```

---

Semoga bermanfaat, _insyaAllah_.
