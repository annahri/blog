---
title: "Docker: Mengupdate Restart Policy Container (tanpa recreate)"
date: 2023-07-11T11:12:38+07:00
draft: false
tags:
- docker
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Ketika kita membuat/menjalankan suatu container, terkadang kita lupa untuk mengatur _restart policy_ terkait. Sehingga, ketika ada _crash_, container tersebut tidak langsung _restart_ secara otomatis.

Berikut ini cara melakukan update _restart policy_ untuk container yang sudah dibuat (sedang berjalan/stop):

    docker update --restart=<policy> <container-id>

Contoh:

    docker update --restart=always abcdef123

Jika berhasil maka akan menghasilkan output ID dari container itu sendiri.

---

Semoga bermanfaat, _barakallahufiikum_.
