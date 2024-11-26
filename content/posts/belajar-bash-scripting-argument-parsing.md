---
title: "Belajar Bash Scripting: Argument Parsing"
date: 2022-08-24T14:39:03+07:00
draft: false
tags:
- bash
- scripting
categories:
- Belajar
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel kali ini, saya akan membahas mengenai _Argument Parsing_. Yakni adalah bagaimana cara agar setiap argumen yang diberikan pada suatu fungsi/script, bisa diinterpretasi sesuai posisinya (_positional argument_), atau sesuai ketentuan yang ditentukan nantinya.

Ini akan sangat bermanfaat jika Anda hendak membuat CLI _tool_ dari bash script. Misalnya membuat tool seperti berikut:

```bash
./script.sh -u username -h host 
# or
./script.sh --username user --host hostname
```

Telah dibahas pada artikel yang telah lalu ([Belajar Bash Scripting: Arguments](/posts/belajar-bash-scripting-arguments/)) bahwa bash akan menginterpretasi setiap argumen yang diberikan dengan variabel `$1`, `$2` dan seterusnya.

## Positional Argument

Yang dimaksud dengan _positional argument_, secara umum adalah argumen yang ditentukan dari urutannya (`$1 .. $n`), dan secara khusus, yaitu argumen yang sudah ditetapkan tujuannya sesuai dengan urutan penyebutannya. Misalnya pada tool `rsync`:

```bash
rsync sourcefiles user@remote:/destination/
```

Bisa didapati bahwa argumen pertama merupakan spesifikasi untuk file sumber yang hendak ditransfer kemudian argumen terakhir adalah tujuannya, entah di lokal atau _remote_.

Sehingga, argumen pertama akan selalu menjadi sumber file dan argumen kedua akan selalu menjadi tujuannya. Jikalau dibalik, tidak bisa, destinasi tujuan didefinisikan pada argumen pertama dan seterusnya.

Atau pada utilitas `mv` atau `cp`, yang ketentuannya adalah jika diberikan 2 nama file, maka yang pertama menjadi sumber file dan yang kedua menjadi tujuan file (hasil salinan, atau hasil pindahan). Namun, jika diberikan lebih dari 2 file, maka argumen paling akhir akan menjadi tujuannya.

```bash
cp file1 file2 file3 target/
mv fileA fileB fileXYZ target/
```

Maka sudah menjadi ketentuan, bahwa file/folder yang disebutkan paling akhir, maka file/folder tersebut akan menjadi targetnya.

Inilah yang dimaksud dengan _positional argument_.

## Optional Argument

_Optional argument_ adalah argument yang tidak wajib untuk diberikan. Boleh diberikan dan boleh juga tidak. Ini berfungsi untuk mengubah sifat suatu tool berdasarkan _flag/option_ yang diberikan. Misalkan pada `rysnc`:

```bash
rsync --dry-run --delete sourcefiles user@remote:/destination/
```

Lho, katanya argumen pertama itu untuk spesifikasi file/folder sumber dan argumen kedua untuk tujuannya? Bukankah pada contoh ini, argumen pertama (`$1`) adalah `--dry-run`? kemudian argumen kedua adalah `--delete`? dan yang ketiga dan keempat baru sumber dan tujuan?

Jawabannya, benar. Jika diruntut, memang terbaca seperti itu (`$1 = '--dry-run'` dan seterusnya). Ini karena _argument parsing_. Tiap argumen yang diberikan akan diproses satu-persatu menyesuaikan dengan pola argumen yang diberikan.

Tool `rsync` tersebut, diprogram sedemikian rupa sehingga ketika didapati ada argument yang berawalan `--` atau `-` maka akan dianggap sebagai _optional parameter_, dan akan diproses berdasarkan string setelahnya. Misalnya pada contoh diatas, `--dry-run` maka `rysnc` akan berjalan pada mode _dry run_ alias percobaan. 

## Argument Parsing

_Argument parsing_ sebenarnya bisa dilakukan dengan _built-in tool_ `getopts`. Namun, pada artikel ini saya hanya akan membahas _argument parsing_ menggunakan `while` dan `for` loop yang dikombinasikan dengan kondisional `case`.

Contoh skenarionya adalah seperti ini:

```bash
./script.sh -a -b opt1 --xyz opt2 pos1 pos2
```

Maka akan didapatkan variabel `$@` yang isinya:

```bash
-a -b opt1 --xyz opt2 pos1 pos2
```

dan `$#` yang bernilai 7. 

{{% callout %}}

Variabel `$#` bernilai sejumlah dengan seluruh argumen yang diberikan.

{{% /callout %}}

Dengan demikian maka bisa diproses masing-masing argumennya dengan loop berikut:

```bash
#!/usr/bin/env bash

while [[ $# -ne 0 ]]; do
    case "$1" in
        -a)
            option_a="set"            
            shift
            ;;
        -b) 
            option_b="$2"
            shift 2
            ;;
        --xyz)
            option_xyz="$2" 
            shift 2
            ;;
        *)
            positional_arg+=( "$1" )
            shift
            ;;
    esac
done

printf '%s: %s\n' option_a "$option_a" option_b "$option_b" option_xyz "$option_xyz"
echo "Positional args: ${positional_arg[*]}"
```

### Penjelasan

Bagian `while [[ $# -ne 0 ]]; do` akan melakukan loop selama `$#` tidak bernilai nol, alias selama masih ada argumen yang bisa diproses, pemrosesan akan terus berlanjut.

Kemudian `case "$1" in` akan mengecek argumen pertama dan akan mecocokkannya dengan  _case-case_ yang didefinisikan setelahnya. Kenapa hanya argumen pertama saja? Akan datang penjelasannya nanti.

_Case-case_ yang dimaksud adalah seperti pada contoh, adalah `-a`, `-b`, `--xyz` dan `*`. Mari dibahas satu-persatu:

```bash
-a)
    option_a="set"            
    shift
    ;;
```

Nah, ketika `$1` bernilai `-a`, maka kita akan definisikan variabel `option_a` dengan nilai `set`. Kemudian ada sintaks `shift`, yang berfungsi untuk menggser _positional parameter_ yang diberikan. Maksudnya bagaimana? 

Seperti yang sudah diketahui sebelumnya, bahwa pada contoh command sebelumnya, argumen-argumen yang diberikan adalah sebagai berikut:

```bash
-a -b opt1 --xyz opt2 pos1 pos2
$1 $2  $3    $4   $5   $6   $7

# $# = 7
```

Jika _command_ `shift` dipanggil, maka penyematan argumen akan menjadi seperti ini:

```bash
-a -b opt1 --xyz opt2 pos1 pos2
   $1  $2    $3   $4   $5   $6

# $# = 6
```

Kemudian, jika `shift` dipanggil dan diberikan nilai padanya, maka pergeseran argumen akan dilakukan sebanyak nilai yang diberikan. Misalnya `shift 2`:

```bash
-a -b opt1 --xyz opt2 pos1 pos2
             $1   $2   $3   $4

# $# = 4
```

Karena _positional argument_-nya bergeser, maka jumlahnya (`$#`) juga berubah pula, sebagaimana pada contoh diatas.

Kembali ke pembahasan sebelumnya, karena kondisi `$1` saat ini bernilai `-a` maka kita isikan variabel `option_a` dengan nilai `set` dan kita geser positional argumen selanjutnya menggunakan perintah `shift`. Sehingga kondisi `$1` saat ini menjadi argumen setelahnya, tidak lagi `-a`.

Setelah `shift` diberlakukan, maka pengkondisian `case` selesai dan kembali kepada loop untuk melakukan iterasi berikutnya karena `$#` masih belum bernilai 0. Selanjutnya: 

```bash
-b) 
    option_b="$2"
    shift 2
    ;;
```

dengan kondisi `$@` yang saat ini adalah seperti berikut ini:

```bash
-b opt1 --xyz opt2 pos1 pos2
$1  $2    $3   $4   $5   $6
```

maka, karena `$1` saat ini bernilai `-b` maka kita deklarasikan variabel `option_b` dengan memberikannya nilai sesuai dengan nilai variabel `$2` yaitu `opt1`. Setelah itu kita geser kembali argumennya 2 kali. Sehingga, kondisi `$@` saat ini menjadi:

```bash
--xyz opt2 pos1 pos2
  $1   $2   $3   $4
```

Untuk bagian dibawah ini, pembahasannya sama dengan `-b`:

```bash
--xyz)
    option_xyz="$2" 
    shift 2
    ;;
```

Kemudian yang terkahir adalah:

```bash
*)
    positional_arg+=( "$1" )
    shift
    ;;
```

Ini berarti, semua argumen yang bukan berupa option (diawali dengan `-`), maka akan teranggap sebagai _positional argument_ dan akan dimasukkan ke _array_ `positional_arg` yang akan diproses tersendiri nantinya.

Dengan demikian, output dari command berikut ini:

```bash
./script.sh -a -b opt1 --xyz opt2 pos1 pos2
```

adalah

```
option_a: set
option_b: opt1
option_xyz: opt2
Positional args: pos1 pos2
```

### Parsing bentuk --opt=value dan -oValue

Setelah dijelaskan konsep dasar pemanfaatan WHILE loop dan CASE _statement_ untuk _argument parsing_ seperti pada contoh sebelumnya, saya akan jelaskan pula bagaimana cara untuk membuat _parsing_ dengan model seperti ini:

```bash
./script.sh --opt1=value1 --flag1=value2 ...
```

Susunan WHILE _loop_ sama seperti dengan contoh sebelumnya, namun bisa diperingkas seperti berikut agar mengurangi kedalaman indentasi:

```bash
#!/usr/bin/env bash

while [[ $# -ne 0 ]]; do case "$1" in
    --flag1=*) var_flag1="${1#*=}";;
    --opt1=*)  var_opt1="${1#*=}" ;;
    *) echo "Opsi tidak dikenali."; exit 1;;
esac; shift; done

printf '%s: %s\n' \
    'Flag1' "$var_flag1" \
    'Opt1' "$var_opt1"
```

{{% callout %}}

Sintaks `shift` pada contoh ini tidak lagi diletakkan pada tiap-tiap _case_ yang diberikan karena itu redundan. Sehingga cukup diletakkan sekali saja setelah _block case_ diakhir tiap _loop_.

{{% /callout %}}

Nah, pada contoh diatas, saya memanfaatkan suatu fitur dari Bash yaitu _parameter expansion_ yang akan dibahas lebih detail pada artikel lain nantinya.

_Parameter expansion_ yang dimanfaatkan adalah `${variabel#*PATTERN}` yang fungsinya adalah menghapus seluruh karakter dari awal hingga bertemu PATTERN. Pada contoh diatas adalah `${1#*=}`. 

Anggap saja nilai `$1` saat ini adalah `--flag1=hehe`, fokus pada parameter `#*=`, maka bisa diartikan dengan "menghapus seluruh karakter dari awal hingga bertemu dengan karakter `=` (inklusif)". Sehingga hasilnya adalah `hehe`.

Kemudian, contoh selanjutnya adalah argument parsing semisal pada _command_ `mysql`:

```bash
mysql -uroot -hhostname -p
```

Hampir sama dengan contoh sebelumnya, hanya berbeda pada sintaks _parameter expansion_ saja dan pada klausa CASEnya:

```bash
#!/usr/bin/env bash

while [[ $# -ne 0 ]]; do case "$1" in
    -u*) username="${1#*-u}";;
    -h*) password="${1#*-h}";;
    -p)  prompt_password=1 ;;
    *) echo "Opsi tidak dikenali."; exit 1;;
esac; shift; done

printf '%s: %s\n' \
    'Username' "$username" \
    'Hostname' "$hostname"

if [[ -n "$prompt_password" ]]; then
    read -r -s -p "Kata sandi: " password
fi
```

### Argument parsing dengan FOR loop

FOR _loop_ juga bisa digunakan untuk melakukan _argument parsing_. Namun dengan batasan, bahwa _argument parsing_ yang dilakukan tidak bisa berupa opsi/flag yang meminta value. Hanya bisa berupa opsi/flag yang bersifat _toggle_ (alias, on/off) dan juga _positional argument_ saja. Misal:

```bash
./script.sh --force --delete
```

```bash
#!/usr/bin/env bash

for arg in "$@"; do case "$arg" in
    --force)   force_enable=true ;;
    --delete) delete_enable=true ;;
    *) echo "Opsi tidak dikenali."; exit 1 ;;
esac; done

...
```

Pada model ini, kita memanfaatkan variabel `$@` dan tidak perlu memanggil `shift` karena tiap iterasinya, FOR loop akan mengkonsumsi semua ekspansi `$@` satu persatu. Bentuknya menjadi lebih ringkas namun kurang fleksibel.

## Validasi

Tentunya, setiap pemberian opsi dan/atau _positional argument_ perlu diberlakukan validasi terlebih dahulu. Misalnya script tidak memperbolehkan penggunaan flag/opsi yang sama lebih dari satu, atau jika opsi yang satu sudah diberikan maka opsi yang lain tidak boleh diberikan pula.

Strategi untuk mengatasi kondisi semacam itu beragam. Diantaranya bisa dengan misalnya opsi yang terakhir diberikan maka itu yang akan dipertimbangkan. Atau pemberian prioritas, atau bahkan script akan error jika bertemu kondisi-kondisi tersebut.

Hal ini bisa dilakukan langsung pada bagian kondisional CASE atau setelah _argument parsing_ selesai.

Contohnya pada utilitas `mv` atau `rm` pada opsi `-i` dan `-f`-nya:

```bash
rm -i -f file2
```

Opsi `-i` berfungsi agar perintah `rm` memberikan _prompt_ sebelum penghapusan file dilakukan. Sedangkan `-f` adalah opsi yang digunakan agar `rm` menghapus "paksa" suatu file tanpa menanyakan/memberitahukan terkait ada/tidaknya file yang dimaksud. Tentu keduanya bertentangan satu sama lain. Bagaimana `rm` menangani kondisi ini? Yaitu dengan memprioritaskan opsi yang paling akhir.

Sehingga bisa saja suatu script disusun seperti ini untuk meniru gaya _argument parsing_ pada `cp`:

```bash
...
while [[ $# -ne 0 ]]; do
    case "$1" in
        ...
        -i) option="interactive"; shift ;;
        -f) option="force"; shift ;;
        ...
    esac
done
...
```

dengan demikian, variabel `option` akan memiliki nilai sesuai dengan opsi yang ditentukan paling akhir.

Kemudian, suatu script dimana script tersebut memerlukan variabel tertentu agar diset oleh user, namun jika tidak diberikan, variabel tersebut akan terisi dengan nilai _default_. Misalnya pada tool `iptables`. Tool tersebut memerlukan ditentukannya nama tabel agar bisa menampilkan semua _rules_-nya dengan flag `-L`.

```bash
iptables -t nat -L
```

Jika, opsti `-t nat` tidak diberikan, maka secara asal, iptables akan menampilkan _rules_ pada tabel filter. Contoh pada script:

```bash
...
while [[ $# -ne 0 ]]; do
    case "$1" in
        ...
        -t) table="$2"; shift 2 ;;
        ...
    esac
done
...

if [[ -z "$table" ]]; then
    table=filter
fi
```

atau lebih sederhana, menggunakan _parameter expansion_:

```bash
...
case "$1" in
    ...
    -t) table="${2:-filter}"; shift 2 ;;
    ...
esac
...
```

{{% callout emoji="☝️" %}}

Yang maknanya, jika variabel `$2` itu `UNSET` atau bernilai kosong (_empty string_), maka isi variabel `table` dengan _string_ `filter`.

{{% /callout %}}

---

Bagaimana dengan bentuk option/flag seperti contoh dibawah ini?

```bash
tar xvzf archive.tar.gz
# atau rsync
rsync -avznP source/ user@host:/dest/
# atau ps
ps aux
# atau ss/netstat
ss -ant
netstat -patlun
```

Saya serahkan pada pembaca untuk mencari tahu.

Semoga bermanfaat, _barakallahufiikum._
