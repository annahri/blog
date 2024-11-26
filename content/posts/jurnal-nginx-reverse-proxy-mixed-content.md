---
title: "Jurnal: Membenahi kendala mixed-content pada Nginx Reverse-Proxy"
date: 2021-11-13T22:09:50Z
draft: false
tags:
- nginx
- reverse-proxy
- jurnal
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Sekedar dokumentasi dari permasalahan yang pernah saya jumpai. Terkadang dari sisi aplikasi tidak siap untuk dipublikasi secara HTTPS karena di dalam _source code_-nya masih ter-_hardcoded_ beberapa _resource_ eksternal menggunakan protokol HTTP.

Hal ini membuat kebanyakan _web browser_ mengeluh dan enggan untuk memuat _resource_ yang dipanggil dari protokol HTTP tadi.

Maka solusinya ada 2:

1. Edit _source code_-nya, ganti semua kode yang memanggil resource dari HTTP menjadi HTTPS.
2. Dari sisi Reverse-Proxy, tinggal tambahkan header berikut:

```
add_header 'Content-Security-Policy' 'upgrade-insecure-requests';
```


Kali ini saya akan membahas solusi untuk Reverse-Proxy Nginx

Contoh konfigurasi server block dan penempatan header :

```
server {
    listen 443 ssl http2;
    server_name namawebsite.tld;

    root /var/www/public_html;
    ... konfigurasi ssl ...

    add_header 'Content-Security-Policy' 'upgrade-insecure-requests';  <-- Taruh disini, di dalam server block dan diluar block location

    location / {
        ... konfigurasi ...
    }
}
```

Semoga bermanfaat, _insyaAllah_.
