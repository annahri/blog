---
title: "Belajar Bash Scripting: Signal Trapping"
date: 2022-11-22T05:22:39+07:00
draft: false
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Dalam menulis _shell script_, ada salah satu _script_ yang dimana sebelum memulai fungsi utamanya, _script_ tersebut menyiapkan/membuat file-file atau folder temporer yang nantinya akan dimanfaatkan. Setelah fungsi utama dijalankan, pada akhirnya nanti file-file temporer tersebut akan dihapus agar tidak "nyampah".

Bagaimana jika script terhenti ditengah jalan (atau terjadi suatu kondisi tertentu), sehingga eksekusi _script_ belum sampai kepada baris yang mengisyaratkan untuk menghapus file-file temporer tadi?

Hal yang semacam ini bisa diatasi dengan mengatur _signal trapping_ menggunakan perintah `trap`.

## Konsep dasar

Perintah `trap` ini penggunaannya sangat sederhana, yaitu:

```shell
trap <perintah> <signal..>
```

Misalnya, perintah dibawah ini akan membuat _script_ mengoutputkan string "mantap" tiap kali _script_ tersebut berakhir/diakhiri:

```shell
trap 'echo "mantap"' EXIT
```

Argumen pertama bisa diisikan serangkaian perintah yang kompleks atau bisa disederhanakan dengan sebuah fungsi. Sehingga, fungsi yang hendak dipanggil didefinisikan dulu sebelumnya, kemudian diatur trapnya.

Kemudian, pada bagian _signal_, kita definisikan pada kejadian sinyal apa perintah yang kita tentukan akan dijalankan. Sehingga perlu diketahui macam-macam sinyal yang ada pada sistem operasi semisal Unix (_Unix-like_, seperti Linux) dan masing-masing fungsinya. 

Misal, sinyal yang dikirimkan pada suatu proses ketika mengekan kombinasi CTRL+C pada proses yang berjalan adalah sinyal INTERRUPT (INT atau SIGINT atau kode sinyal no 2). Selangkapnya bisa merujuk pada _manpages_ resmi: https://man7.org/linux/man-pages/man7/signal.7.html atau dari output perintah `trap -l`:

```textile
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

## Contoh implementasi

Sebagaimana yang telah disebutkan sebelumnya, salah satu implementasi perintah `trap` ini adalah untuk menghapus file-file temporer untuk keperluan _script_.

Misal, _script_ dibawah ini berfungsi sebagai _wrapper_ untuk memodifikasi file **/etc/hosts**:

```bash
#!/usr/bin/env bash

cleanup() { rm -f '/tmp/temp-file'; }
trap 'cleanup' EXIT

main() {
    cp -f /etc/hosts /tmp/temp-file

    if $EDITOR /tmp/temp-file; then
        cp -f /tmp/temp-file /etc/hosts
    fi
    
    cleanup
    trap - EXIT # me-unset trap untuk sinyal EXIT
}

main
```

Dimisalkan pada suatu kondisi tertentu, setelah meng-copy file **/etc/hosts** script terhenti, maka dengan diaturnya trap diatas, file **/tmp/temp-file** akan secara otomatis dihapus.

Dengan demikian, kita tidak perlu khawatir _script_ kita meninggalkan temporary files.

Atau contoh lainnya yang lebih _advanced_ adalah memanfaatkan _signal trapping_ ini untuk memungkinkan _error tracing_. Dengan begitu, ketika ada _error_ pada _script_, nantinya akan muncul pada baris berapa _error_ tersebut muncul.

```bash
trap 'printf "Script exit dengan status %d pada baris %d\n" "$?" "$LINENO"' ERR
```

Dengan contoh _script_ demo:

```bash
#!/bin/bash

set -eE
trap 'printf "Script exit dengan status %d pada baris %d\n" "$?" "$LINENO"' ERR

for x in {1..10}; do
    [[ $x == 5 ]] && qweqwe 2> /dev/null
    sleep 1 && echo "Putaran ke $x"
done
```

Ketika dijalankan, maka outputnya akan seperti ini:

```textile
$ ./script.sh 
Putaran ke 1
Putaran ke 2
Putaran ke 3
Putaran ke 4
Script exit dengan status 127 pada baris 7
```

{{% callout %}}

Bagian `set -eE` akan dibahas lebih lanjut pada artikel lain tentang _Bash Script_ mode ketat (_strict mode_).

{{%/ callout %}}

## Penutup

Masih banyak penerapan-penerapan lain dengan menggunakan _signal trapping_ ini, tidak hanya terbatas pada contoh-contoh diatas. Seperti _trapping_ sinyal **SIGUSR1** yang itu merupakan sinyal yang didefinisikan oleh user, dan banyak lainnya.

Semoga bermanfaat, _barakallahu fiikum_.
