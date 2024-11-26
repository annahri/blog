---
title: "Jurnal: Script Rincian Mail Queue Postfix"
date: 2022-04-08T09:25:02+07:00
tags:
- scripting
- bash
- python
categories:
- Jurnal
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel ini saya akan memberikan contoh script Bash dan Python yang berfungsi untuk menampilkan jumlah antrian email (_mail queue_) keluar pada Postfix. Ini berguna sebagai metrics yang bisa digunakan untuk alerting. Baik alerting manual maupun terintegrasi seperti ServerDensity, dan sebagainya.

### Dasar Script

Komponen utama utama untuk mengetahui _mail queue_ adalah folder dimana _queue_ tersebut singgah untuk sementara waktu. Untuk mengetahuinya, kita bisa memanggil command `postconf -h queue_directory` yang outputnya akan menunjukkan dimana direktori tersebut.

Contoh struktur dari folder queue tersebut adalah seperti ini:

```
root@postfix-relay-new:/var/spool/postfix# tree -d -L 1 -n
.
├── active
├── bounce
├── corrupt
├── defer
├── deferred
├── dev
├── etc
├── flush
├── hold
├── incoming
├── lib
├── maildrop
├── pid
├── private
├── public
├── saved
├── trace
└── usr

18 directories
```

Folder-folder yang berisi _queue_ email adalah active, corrupt, defer, deferred, hold dan incoming.

Contoh file _queue_ yang ada pada folder deferred:
```
root@postfix-relay-new:/var/spool/postfix/deferred# find -type f
./E/EAB4426846
./7/7C31723297
./B/B2F962B543
...
./2/28BD526F02
./8/8B6742C09D
./5/548D72A868
```

Sehingga dari situ, kita cukup meracik _command_ `find` sedemikian rupa sehingga _command_ tersebut hanya akan menampilkan file-file queue saja pada direktori yang sudah ditunjuk oleh `postconf -h queue_directory`. Karena ciri file _queue_ sifatnya seragam, maka bisa kita gunakan Regular Expression dibawah ini:

```regex
[0-9A-F]+
```

Kemudian, jika sudah dapat outputnya, maka untuk masing-masing kondisi _queue_ (active/defer, dsb) tinggal dihitung ada berapa baris yang pada _path_ -nya terdapat substring *active* misalnya. Atau *defer*, dll. Dan ini bisa dilakukan dengan berbagai macam cara. Salah satu diantaranya adalah dengan `grep`.

```bash
grep -w -c 'active' input

# Dimana:
# -w  mencari kata utuh, bukan substring
# -c  jumlah temuan
```

Dan selesai, intinya hanya itu saja.

### Versi Bash #1

Idenya adalah dengan menyimpan daftar _queue_ pada suatu file temporer, kemudian untuk setiap _state_, kita panggil grep count untuk menemukan ada berapa kali keyword _state_ tersebut ditemukan pada file temporer tersebut.

```bash
#!/usr/bin/env bash

readonly queue_dir=$(postconf -h queue_directory) 
readonly tmp_file=$(mktemp) 

states=(all active corrupt defer deferred hold incoming) 

get_count() {
    printf '%s: %d\n' "$1" $(egrep -w -c "$2" "$tmp_file") 
} 

main() {
    find "$queue_dir" -type f -regextype egrep -regex '.*/[0-9A-F]{5,}$' > "$tmp_file"
    
    for state in ${states[@]}; do
        search="$state" 
        [[ $state = all ]] && search='.*'
        get_count "$state" "$search"
    done

    rm -f "$tmp_file"
}

main 
```

Contoh output jika dijalankan:
```sh
root@postfix-relay-new:~/script# time bash script.sh 
all: 43
active: 0
corrupt: 0
defer: 28
deferred: 15
hold: 0
incoming: 0

real	0m0.098s
user	0m0.026s
sys	0m0.015s
```

### Versi Bash #2

Melakukan loop untuk setiap entri _queue_ yang ditemukan kemudian langsung menghitungnya dan menyimpannya pada array asosiatif. Tetapi ini memerlukan 2 kali loop. Yang pertama untuk menghitung jumlah, yang kedua untuk mengeluarkan output.

```bash
#!/usr/bin/env bash

readonly queue_dir=$(postconf -h queue_directory)
declare -A count=([active]=0 [corrupt]=0 [defer]=0 [deferred]=0 [hold]=0 [incoming]=0)
total=0

while read line; do
    state=$(cut -d'/' -f5 <<< "$line")
    (( ++count[$state] ))
    (( ++total ))    
done < <(find "$queue_dir" -type f -regextype egrep -regex '.*/[0-9A-F]+$')

echo "all: $total"
for i in ${!count[@]}; do
    printf '%s: %d\n' $i ${count[$i]}
done | sort
```

Output:

```sh
root@postfix-relay-new:~/script# time bash script3.sh 
all: 43
active: 0
corrupt: 0
defer: 28
deferred: 15
hold: 0
incoming: 0

real	0m0.133s
user	0m0.084s
sys	0m0.052s
```

### Versi Bash #3 (dengan AWK)

Versi AWK ini merupakan versi paling cepat diantara versi script Bash sebelumnya. Karena shell tidak melakukan loop sama sekali, dan seluruh proses dilakukan oleh awk dalam sekali eksekusi. Sedangkan pada script sebelumnya (#2), setiap iterasi ada eksekusi eksternal tool `cut` yang berdampak pada durasi eksekusi (meskipun tidak terlalu signifikan).

Ide yang digunakan, sama dengan pada script bash versi kedua.

```bash
#!/usr/bin/env bash

find `postconf -h queue_dir` -type f -regextype egrep -regex '.*/[0-9A-F]{5,}$' \
    | awk -F'/' 'BEGIN { count["active"]=0; count["corrupt"]=0; count["defer"]=0; count["deferred"]=0; count["hold"]=0; count["incoming"]=0 }
    { count[$5]++; total++ }
    END { print "all:", total; for (n in count) print n":", count[n] | "sort" }'
```

Agar mempermudah memahami sintaks awk panjang tersebut, berikut ini versi baris per baris:

```awk
BEGIN {
    count["active"]=0
    count["corrupt"]=0
    count["defer"]=0
    count["deferred"]=0
    count["hold"]=0
    count["incoming"]=0
}
{
    count[$5]++
    total++
}
END {
    print "all:", total
    for (n in count) print n":", count[n] | "sort"
}
```

Output:

```sh
root@postfix-relay-new:~/script# time bash wawk.sh 
all: 43
active: 0
corrupt: 0
defer: 28
deferred: 15
hold: 0
incoming: 0

real	0m0.083s
user	0m0.011s
sys	0m0.011s
```

### Versi Python

Script versi Python ini menggunakan ide yang sama pada script bash pertama.

```python
import os
from subprocess import check_output

def get_list():
    queue_dir = check_output(['postconf','-h','queue_directory']).decode('utf-8').rstrip()
    command_string = 'find ' + queue_dir + ' -type f -regextype egrep -regex .*/[0-9A-F]+$'
    return check_output(command_string.split()).decode('utf-8').rstrip().split('\n')

def get_count(string, search, list):
    n = sum(search in entry for entry in list)
    print(f'{string}: {n}')

if __name__ == '__main__':
    result = get_list()
    states = ['all', 'active', 'corrupt', 'defer', 'deferred', 'hold', 'incoming']
    
    for state in states:
        search = state + '/' if state != 'all' else ''
        get_count(state, search, result)
```

Contoh output jika dijalankan:
```sh
root@postfix-relay-new:~/script# time python3 mqueue.py 
all: 43
active: 0
corrupted: 0
defer: 28
deferred: 15
hold: 0
incoming: 0

real	0m0.076s
user	0m0.055s
sys	0m0.011s
```

---

Semoga bermanfaat, _insyaAllah_.
