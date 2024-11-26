---
title: "Belajar Bash Scripting: Conditional Statement"
date: 2021-12-15T21:21:59+07:00
draft: true
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel ini saya akan menjelaskan bagaimana cara melakukan kondisional if pada _shell_ bash. Perlu diketahui, sebagian besar tutorial pembelajaran mengenai _Bash Scripting_ tidak akan POSIX-compliant alias tidak portabel. Artinya, belum tentu script bash untuk versi A bisa berjalan pada sistem yang memiliki bash versi C. Dan tentunya script bash tersebut tidak akan berjalan pada sistem-sistem yang tidak memiliki shell bash.

## Struktur dasar

```
$ help if
if: if COMMANDS; then COMMANDS; [ elif COMMANDS; then COMMANDS; ]... [ else COMMANDS; ] fi
    Execute commands based on conditional.
    
    The `if COMMANDS' list is executed.  If its exit status is zero, then the
    `then COMMANDS' list is executed.  Otherwise, each `elif COMMANDS' list is
    executed in turn, and if its exit status is zero, the corresponding
    `then COMMANDS' list is executed and the if command completes.  Otherwise,
    the `else COMMANDS' list is executed, if present.  The exit status of the
    entire construct is the exit status of the last command executed, or zero
    if no condition tested true.
    
    Exit Status:
    Returns the status of the last command executed.
```

Sesuai dengan informasi yang bisa didapat melalui perintah `help if` diatas, struktur dasar untuk melakukan statement kondisional adalah sebagai berikut:

```bash
if TES; then
        PERINTAH
fi
```

Anda juga akan mendapati beberapa tutorial/artikel yang lebih memilih struktur seperti ini:

```bash
if TES
then
        PERINTAH
fi

```

Keduanya sama-sama valid dan hanya kecondongan pribadi saja untuk lebih memilih yang mana. Kalau saya pribadi terbiasa dengan cara yang pertama karena lebih rapi.

> Sekedar informasi tambahan, pada sintaks pertama, dari `if` menuju `then` terdapat tanda `;` yang menunjukkan bahwa `if` dan `then` adalah dua _command_ yang berbeda. Karena syarat suatu baris _command_ itu bisa dijadikan _oneliner_ adalah dengan memberi tanda `;` untuk setiap penggunaan _command_ yang berbeda.

Melanjutkan pembahasan, _command_ `if` akan mengevaluasi baris _command_ yang ditentukan pada `TES`. Jika kode status (_exit status_) yang dihasilkan `TES` adalah _non-zero_ (yang berarti tidak ada error), maka baris _command_ PERINTAH akan dijalankan.

Bagaimana contohnya baris TES tersebut? Yang bisa digunakan adalah semua _command_ yang menghasilkan _exit status_. Entah _command_ tersebut adalah suatu _command_ rsync, wget, curl atau apapun itu yang diakhir _command_ akan menghasilkan kode status. Kode status itulah yang kemudian dievaluasi oleh _command_ `if`. Misal:

```bash
if rsync -qaz source dest; then
    echo "Transfer berhasil."
fi
```

Contoh diatas, jika _command_ `rsync` tersebut berhasil dijalankan tanpa ada error, maka shell akan melakukan `echo "Transfer berhasil"`. Atau contoh lain:

```bash
if grep -q cari file; then
    echo "Kata 'cari' ditemukan di dalam file"
fi
```

Dan yang semisalnya.

## Ekspresi Kondisional

Pada penjelasan sebelumnya, _command_ `if` mengevaluasi kode status yang dihasilkan oleh _command_ yang sedang dites. Nah, pada bagian ini, saya akan menjelaskan bagaimana menentukan ekspresi kondisional untuk mengevaluasi suatu variabel terhadap variabel lain atau terhadap suatu nilai; atau jika variabel tersebut berupa _path_ menuju suatu file, apakah file tersebut _exists_ alias ada atau tidak; jika variabel sudah diset atau belum, dan sebagainya.

Jika pada pembahasan sebelumnya evaluasi yang dilakukan adalah terhadap _command_ eksternal alias diluar dari fungsi internal pada bash, maka kali ini evaluasi ekspresi kondisional menggunakan perintah bash _built-in_. Alias semua _command_ yang bisa dicari informasinya menggunakan sintaks `help command`. 

```
$ help help
help: help [-dms] [pattern ...]
    Display information about builtin commands.
    
    Displays brief summaries of builtin commands.  If PATTERN is
    specified, gives detailed help on all commands matching PATTERN,
    otherwise the list of help topics is printed.
    
    Options:
      -d	output short description for each topic
      -m	display usage in pseudo-manpage format
      -s	output only a short usage synopsis for each topic matching
    		PATTERN
    
    Arguments:
      PATTERN	Pattern specifying a help topic
    
    Exit Status:
    Returns success unless PATTERN is not found or an invalid option is given.
```

Evaluasi ekpresi kondisional bisa dilakukan dengan tiga bentuk:

1. Menggunakan _command_ `test ekspresi`
2. Menggunakan _command_ `[ ekspresi ]`
3. Menggunakan _command_ `[[ ekspresi ]]`

Ketiganya valid dan sah hanya saja untuk bentuk yang ketiga adalah kekhususan untuk shell bash alias bentuk _bashism_ sedangkan yang lain lebih portabel.

## Macam-macam ekspresi

Ekspresi-ekspresi dibawah ini akan memberikan _exit status_ antara 0 atau 1. Jika berhasil atau bernilai benar, maka hasilnya 0 sebaliknya jika gagal atau salah maka hasilnya 1.

Cara penggunaannya adalah sebagai berikut:
```bash
# 1. test operator
test -f file
test -z variabel
test -n variabel
test nilai1 -eq nilai2
...

# 2. [ operator ]
[ -f file ]
[ -z variabel ]
[ -n variabel ]
[ nilai1 -eq nilai2 ]
...

# 3. [[ operator ]]
[[ -f file ]]
[[ -z variabel ]]
[[ -n variabel ]]
[[ nilai1 -eq nilai2 ]]
...
```

### Operator file

Operator-operator berikut digunakan untuk melakukan pengujian atas suatu file (dan/atau terhadap file lainnya).

| **Operator**    | **Output**                                               |
|-----------------|----------------------------------------------------------|
| -a FILE         | Benar jika file anda                                     |
| -b FILE         | Benar jika file adalah block spesial                     |
| -c FILE         | Benar jika file adalah character spesial                 |
| -d FILE         | Benar jika file adalah direktori                         |
| -e FILE         | Benar jika file ada                                      |
| -f FILE         | Benar jika file ada dan file tersebut reguler            |
| -g FILE         | Benar jika file telah diset group-id                     |
| -h FILE         | Benar jika file adalah symlink                           |
| -L FILE         | Benar jika file adalah symlink                           |
| -k FILE         | Benar jika file memiliki sticky bit                      |
| -p FILE         | Benar jika file adalah _named pipe_                      |
| -r FILE         | Benar jika file _readable_                               |
| -s FILE         | Benar jika file ada dan tidak kosong                     |
| -S FILE         | Benar jika file adalah socket                            |
| -t FD           | Benar jika FD terbuka pada terminal                      |
| -u FILE         | Benar jika file telah diset user-id                      |
| -w FILE         | Benar jika file _writable_                               |
| -x FILE         | Benar jika file _executable_                             |
| -O FILE         | Benar jika file dimiliki oleh Anda                       |
| -G FILE         | Benar jika file dimiliki oleh grup Anda                  |
| -N FILE         | Benar jika file telah dimodifikasi sejak terakhir dibaca |
| FILE1 -nt FILE2 | Benar jika file1 lebih baru dari file2                   |
| FILE1 -ot FILE2 | Benar jika file1 lebih tua dari file2                    |
| FILE1 -ef FILE2 | Benar jika file1 merupakan _hard link_ ke file2          |


### Operator string

Operator-operator berikut digunakan untuk pengujian atas suatu variabel (dan juga terhadap variabel lain) terkait dengan string. 

| **Operator**       | **Output**                                                    |
|--------------------|---------------------------------------------------------------|
| -z STRING          | Benar jika string kosong                                      |
| -n STRING          | Benar jika string tidak kosong                                |
| STRING1 = STRING2  | Benar jika string1 sama dengan string2                        |
| STRING1 != STRING2 | Benar jika string1 tidak sama dengan string2                  |
| STRING1 < STRING2  | Benar jika string1 urut sebelum string2 secara leksikografis. |
| STRING1 > STRING2  | Benar jika string1 urut setelah string2 secara leksikografis  |

### Operator aritmatika

Operator-operator berikut digunakan untuk pengujian matematis atas suatu variabel dengan suatu nilai atau dengan variabel lain.

| **Operator**  | **Output**                                                 |
|---------------|------------------------------------------------------------|
| ARG1 -eq ARG2 | Benar jika arg1 bernilai sama dengan arg2                  |
| ARG1 -lt ARG2 | Benar jika arg1 bernilai lebih kecil dari arg2             |
| ARG1 -le ARG2 | Benar jika arg1 bernilai lebih kecil atau sama dengan arg2 |
| ARG1 -gt ARG2 | Benar jika arg1 bernilai lebih besar dari arg2             |
| ARG1 -ge ARG2 | Benar jika ar1 bernilai lebih besar atau sama dengan arg2  |

### Operator lain-lain

| **Operator**   | **Output**                                                               |
|----------------|--------------------------------------------------------------------------|
| -o OPTION      | Benar jika opsi shell OPTION aktif                                       |
| -v VAR         | Benar jika variabel shell VAR telah diset                                |
| -R VAR         | Benar jika variabel shell VAR telah diset dan merupakan _name reference_ |
| EXPR1 -a EXPR2 | Benar jika kedua expresi bernilai benar                                  |
| EXPR1 -o EXPR2 | Benar jika expr1 bernilai benar atau expr2 bernilai benar                |
| ! EXPR         | Benar jika expr bernilai salah                                           |

Untuk bagian lain-lain ini, operator yang akan sering digunakan adalah operator `!`. Sedangkan sisanya, digunakan untuk scripting tingkat lanjutan.

Contoh akan dijelaskan pada bagian selanjutnya.

## Kondisional: if...

Seperti yang sudah dijelaskan di awal, struktur kondisional if... jika digabungkan dengan ekspresi-ekspresi diatas, adalah sebagai berikut:

```bash
# Bentuk 1
if test "$nilai" -gt 0; then
    command
fi

if test -d dir; then
    command
fi

# Bentuk 2
if [ "$nilai" -gt 0 ]; then
    command
fi

if [ -d dir ]; then
    command
fi

# Bentuk 3
if [[ $nilai -gt 0 ]]; then
    command
fi

if [[ -d dir ]]; then
    command
fi
```

### Bentuk Lain

Yang saya maksud bentuk lain adalah, kondisional if tetapi tidak berupa struktur seperti contoh-contoh sebelumnya. Kali ini Anda akan berkenalan dengan _lazy logical operator_ AND `&&`. Operator `&&` dapat dipanggil diantara 2 _command_. Contoh:

```
command1 && command2
```

Ketika `command1` berhasil dijalankan tanpa error (_exit status_ = 0) maka `command2` akan dijalankan. Berbeda dengan _operand_ `;`. Dia akan tetap menjalankan `command2` meskipun `command1` gagal dijalankan. Lantas bagaimana mengkaitkan `&&` dengan kondisional?

Penting untuk diketahui bahwa ketiga bentuk kondisional sebelumnya bisa dijalankan tanpa harus diawali `if`. Jika dikolaborasikan dengan operand `&&`, maka contohnya sebagai berikut:

```bash
# Dengan struktur if 
if test -z "$variabel"; then
    echo "variabel kosong"
fi

if [ -z "$variabel" ]; then
    echo "variabel kosong"
fi

if [[ -z $variabel ]]; then
    echo "variabel kosong"
fi

# Dengan menggunakan && tanpa if
test -z "$variabel" && echo "variabel kosong"
[ -z "$variabel" ] && echo "variabel kosong"
[[ -z $variabel ]] && echo "variabel kosong"
```

Terlihat lebih ringkas bukan? Untuk statement kondisional yang sederhana, bentuk seperti diatas bisa digunakan.

Beberapa pihak mengatakan bahwa bentuk tersebut mengorbankan sisi keterbacaan daripada bentuk struktur if.


## Kondisional: if... else...



## Kondisional: if... elif... else...
