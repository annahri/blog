---
title: "Belajar Bash Scripting: Functions"
date: 2022-05-29T08:09:00+07:00
draft: false
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel ini saya akan menjelaskan seputar _shell functions_, bagaimana cara mendefinisikan suatu fungsi pada Bash _shell_ yang itu merupakan seperangkat perintah yang disusun sedemikian rupa agar bisa digunakan kembali pada bagian lain suatu _script_.

## Struktur dasar

Sebuah fungsi dapat didefinisikan dengan susunan sebagai berikut:

```bash
nama_fungsi() {
    ...
    command1
    command2
    ...
}

# atau
function nama_fungsi() {
    ...
}
```

Kedua bentuk diatas tidak memiliki perbedaan, namun sebagian ahli mengatakan bahwa bentuk kedua tidaklah portabel. Alias, tidak kompatibel dengan Bash shell versi lama.

## Memanggil fungsi

Cara untuk memanggil fungsi tersebut adalah cukup dengan memanggil nama fungsi tersebut (tanpa kurung).

```bash
hello_world() {
    echo "Hello, World!"
}

hello_world
# Hello, World!
```

Diantara kesalahan orang-orang yang baru pertama kali menulis shell script adalah memanggil fungsi shell dengan diakhiri tanda `()` sebagaimana pada sintaks keumuman bahsa pemrograman.

```bash
fungsi() {
    ...
}

# salah
fungsi()

# benar
fungsi
```

## Variabel global vs lokal

Pada dasarnya, setiap variabel yang didefinisikan di dalam variabel merupakan variabel lokal secara asal. Namun, Anda bisa mendefinisikannya secara eksplisit dengan sintaks `local`.

Jika ada variabel global (yang sudah didefinisikan sebelum dan diluar fungsi) kemudian ada variabel dengan nama yang sama didefinisikan didalam fungsi tersebut (lokal), maka variabel lokal tersebut yang akan diutamakan.

```bash
nama="Alan"

fungsi() {
    local nama="Fulan"
    echo "Hello, ${nama}."
}

fungsi
# Hello, Fulan.

echo "Hello, ${nama}."
# Hello, Alan.
```

## Argumen/parameter pada fungsi

Argumen atau parameter pada shell bash direpresentasikan dengan variabel shell `$1`, `$2`, `$3` dan seterusnya, menyesuaikan dengan posisi argumen tersebut. Penjelasan lebih terperinci ada pada artikel [Belajar Bash Scripting: Shell Arguments](/posts/#).

```bash
salam() {
    echo "Ahlan wa sahlan, ${1}!"
    echo "Selamat datang, ${2}."
}

salam Fulan Alan
# Ahlan wa sahlan, Fulan!
# Selamat datang, Alan.
```

## Menentukan nilai _return_ pada fungsi

Pada keumuman bahasa pemrograman, suatu fungsi dapat "mengembalikan" suatu nilai setelah dieksekusi. Itu bisa berupa _string_, _integer_, _boolean_ atau yang lainnya. 

Terkadang pula, untuk menentukan nilai _return_ tersebut perlu mendeklarasikan jenis data yang akan dikembalikan tersebut.

Hal ini tidak berlaku pada shell scripting, terutama pada shell Bash. Semua command yang mengeluarkan outputnya pada __stdout__, maka itu bisa dianggap sebagai nilai return.

```bash
cpu_cores() {
    grep -c '^processor' /proc/cpuinfo
}

worker=$(( ( `cpu_cores` * 2 ) + 1  ))

echo "Jumlah worker: $worker"
# Jumlah worker: 9
```

## Sintaks `return`

Sintaks `return` fungsinya tidak sebagaimana pada keumuman bahasa pemrograman (yaitu menentukan nilai balik suatu fungsi), tetapi ini berfungsi untuk keluar dari suatu fungsi tanpa menghentikan eksekusi script.

Misalnya, pada suatu fungsi terdapat pengkondisian seperti berikut ini:

```bash
fungsi() {
    local file="$1"

    if [[ -f "$file" ]]; then
        echo "File sudah ada"
        return
    fi

    if [[ ! -w "$file" ]]; then
        echo "Tidak dapat menulis file."
        return 1
    fi

    # Jika file belum ada
    ...
    ...
    ...
}
```

Nilai yang didefinisikan setelah sintaks `return` tersebut, adalah nilai exit. Ketentuannya adalah, jika nilai tersebut lebih dari nol, maka error terjadi.

Ini bisa dimanfaatkan untuk meniru nilai return boolean, seperti contoh berikut ini:

```bash
file_exists() {
    test -f "$1" || return 1
    return
}

if file_exists "file.jpg"; then
    ...
fi
```

Tambahan, jika melihat penjelasan sintaks `return` dari `help return`, maka dijelaskan bahwa sintaks tersebut hanya bisa dieksekusi dari dalam suatu fungsi. Sehingga, ketika sintaks tersebut dijalankan diluar fungsi, maka akan ekluar eror:

```terminal
bash: return: can only `return' from a function or sourced script
```

---

Semoga bermanfaat, _Barakallahufiikum_.
