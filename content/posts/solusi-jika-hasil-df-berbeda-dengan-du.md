---
title: "Jurnal: Solusi jika Hasil df Berbeda dengan du"
date: 2023-04-10T19:08:37+07:00
tags:
- df
- du
- filesystem
- lsof
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel kali ini saya akan membahas mengenai fenomena dimana hasil output dari `df` berbeda dari `du`.

Contohnya adalah berikut ini:

```
root@host:~# df -h /
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   19G   18G     0 100% /

root@host:~# du -shx /
5.2G    /
```

Bisa diperhatikan bahwa keduanya menunjukkan hasil yang sangat berbeda sama sekali. Kok bisa begitu? Jawabannya [disini](#penjelasan-ringkas)

## Solusi

Pada contoh diatas, `df` melaporkan _size_ yang tersedia adalah 0. Sedangkan pada `du`, penggunaan _disk_ pada partisi `/` hanyalah sekitar __5.2G__ saja.

Kemungkinan besar, ada suatu proses yang sedang berjalan yang menahan suatu/beberapa file yang berukuran besar. Sehingga, file tersebut tidak benar-benar terhapus pada sistem.

Untuk memastikan hal ini, bisa kita gunakan perintah `lsof +1L` yang berfungsi untuk melihat file-file yang masih tertahan oleh suatu proses yang sebenarnya file tersebut sudah terhapus:

```
root@host:~# lsof +1L 
...
cron      12338        root    5u   REG  253,0 31407562752     0    356 /tmp/#356 (deleted)
sh        12339    www-data    1u   REG  253,0 31407562752     0    356 /tmp/#356 (deleted)
sh        12339    www-data    2u   REG  253,0 31407562752     0    356 /tmp/#356 (deleted)
php7.4    12341    www-data    1u   REG  253,0 31407562752     0    356 /tmp/#356 (deleted)
php7.4    12341    www-data    2u   REG  253,0 31407562752     0    356 /tmp/#356 (deleted)
sh        13644    www-data    2u   REG  253,0 31407562752     0    356 /tmp/#356 (deleted)
gs        13645    www-data    2u   REG  253,0 31407562752     0    356 /tmp/#356 (deleted)
...
```

Dari sini bisa diperhatikan bahwa, ada proses-proses yang masih menahan suatu file yang sudah dihapus dengan ukuran yang terhitung cukup besar. Hal seperti ini tidak akan bisa ditemukan jika hanya menggunakan tool seperti `du`, `ncdu` atau tool semisal.

Kemudian bagaimana "membebaskan" ruang disk yang "tersita" tersebut?


### 1. Melakukan _truncate_ pada _file descriptor_ proses terkait

_File descriptor_ suatu proses yang berjalan bisa diakses pada direktori `/proc/<PID>/fd`. Pada direktori tersebut, terdapat daftar _file descriptor_ yang sedang aktif. Contoh:

```
root@host:/proc/12338/fd# ll
total 0
dr-x------ 2 root root  0 Apr 10 11:32 ./
dr-xr-xr-x 9 root root  0 Apr  1 23:45 ../
lr-x------ 1 root root 64 Apr 10 11:32 0 -> /dev/null                                  
lrwx------ 1 root root 64 Apr 10 11:32 1 -> 'socket:[798]'                                                                                 lrwx------ 1 root root 64 Apr 10 11:32 2 -> 'socket:[798]'                                  
lrwx------ 1 root root 64 Apr 10 11:32 5 -> '/tmp/#356 (deleted)'                                                                          lrwx------ 1 root root 64 Apr 10 11:32 6 -> 'socket:[455791167]'
```

Terlihat bahwa _file descriptor_ 5 merujuk pada file yang "tersita" tersebut. Langkah selanjutnya adalah melakukan truncate proses terkait, yaitu dengan:

```
: > /proc/<PID>/fd/<FD>
```

{{% callout %}}
Perintah `:` adalah perintah untuk tidak melakukan apa-apa. Sehingga ketika dijalankan, dia tidak akan memberikan output.

Jika output dari perintah tersebut diarahkan (_redirect_) kepada suatu file, maka file tersebut akan menjadi kosong.

Dengan demikian, inode dari file terkait tidak akan berubah.

Alternatif lain adalah dengan menggunakan perintah `true`.
{{% /callout %}}

Namun, yang perlu diperhatikan pada cara ini, meskipun tidak perlu dilakukan penghentian proses (_kill_), tetapi file yang tertahan tersebut akan terisi kembali seiring berjalannya waktu. Sehingga cara ini merupakan cara sementara saja.

### 2. Menghentikan proses yang menahan file tersebut

Jika sudah bisa dipastikan proses yang sedang berjalan itu aman untuk dihentikan (karena alasan tertentu seperti, proses macet atau sebagainya), maka proses terkait bisa dihentikan untuk membebaskan ruang _disk_ yang tertahant tersebut.

Cukup lakukan:

```
kill -9 <PID>
```

## Penjelasan ringkas

Secara singkatnya, `df` melakukan penghitungan pada tingkatan _filesystem_ sedangkan `du`, melakukan penghitungan langsung pada objek file itu sendiri.

Lebih lengkapnya bisa dibaca disini:
- [Why DU And DF Display Different Values On Linux And Unix](https://linuxshellaccount.blogspot.com/2008/12/why-du-and-df-display-different-values.html)
- [Serverfault - du vs. df difference](https://serverfault.com/questions/57098/du-vs-df-difference)

---

Semoga bermanfaat, _barakallahufiikum_
