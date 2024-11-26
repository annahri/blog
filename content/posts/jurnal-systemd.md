---
title: "Jurnal: Mengatasi service systemd yang tidak mau restart proses"
date: 2022-06-14T10:25:00+07:00
draft: true
tags:
- systemd
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

_InsyaAllah_, pada artikel ini saya akan menjelaskan bagaimana cara agar suatu service systemd yang file initnya ada pada /etc/init.d (alias, service tersebut didesain untuk SysV tetapi dijalankan pada sistem yang menggunakan systemd sebagai init-nya).

Jika demikian, maka systemd akan men-generate file service saat file init yang dimaksud dijalankan, dan tidak bisa ditemukan pada folder `/etc/systemd/system`. Tetapi ada pada `/run`. Sehingga segala modifikasi yang ada didalamnya akan hilang ketika service tersebut dijalankan ulang.

## Latar permasalahan

Suatu service bernama `rrdcached` tidak diketahui kenapa, sewaktu-waktu mati sendiri. Saat dicek dengan `systemctl status rrdcached` maka statusnya `Active (exited)`.

Bisa disimpulkan bahwa service tersebut hanya menjalankan program yang diset pada file init, dan tidak memonitor state prosesnya. Sehingga jika ada kendala seperti proses mati sendiri, atau semacamnya, maka systemd tidak akan berbuat apa-apa.

```
systemctl cat service 
systemctl edit service
```
Systemd Unit file overrides

Forking:

> If set to **forking**, it is expected that the process configured with `ExecStart=` will call `fork()` as part of its start-up. The parent process is expected to exit when start-up is complete and all communication channels are set up. The child continues to run as the main service process, and the service manager will consider the unit started when the parent process exits. This is the behavior of traditional UNIX services. If this setting is used, it is recommended to also use the **PIDFile=** option, so that systemd can reliably identify the main process of the service. systemd will proceed with starting follow-up units as soon as the parent process exits. 