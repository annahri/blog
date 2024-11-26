---
title: "Jurnal: Zimbra - Mencari Tahu Konfigurasi Filter Untuk Semua Akun"
date: 2022-05-22T21:39:00+07:00
draft: false
tags:
- zimbra
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Ini adalah artikel ringkas yang membahas tentang cara untuk mencari tahu seluruh konfigurasi filter yang diatur oleh tiap akun email.

Pada dasarnya, perintah yang digunakan adalah:

```bash
zmprov -l ga <nama akun> zimbraMailSieveScript
```
Contoh output:

```bash
# name user1@example.com
zimbraMailSieveScript: require ["fileinto", "copy", "reject", "tag", "flag", "variables", "log", "enotify", "envelope", "body", "ereject", "reject", "relational", "comparator-i;ascii- 
numeric"];
 
# forward
if anyof (address :all :contains :comparator "i;ascii-casemap" ["to"] "user1@example.com") {
  redirect "user2@example.com";
  stop;
}
```

Sehingga, untuk meng-_query_ keseluruhan akun, bisa menggunakan loop atau `xargs`:

```bash
zmprov -l gaa | xargs -n1 -I zmprov -l ga {} zimbraMailSieveScript

# atau 

for account in $(zmprov -l gaa); do
    echo "$account"
    zmprov -l ga "$account" zimbraMailSieveScript
done

# atau
zmprov -l gaa | while read account; do
    echo "$account"
    zmprov -l ga "$account" zimbraMailSieveScript
done

# atau
while read account; do
    echo "$account"
    zmprov -l ga "$account" zimbraMailSieveScript
done < <(zmprov -l gaa)
```

Semoga bermanfaat, _barakallahufiikum_.

---

Referensi: [Steps to get filters of all accounts](https://wiki.zimbra.com/wiki/Steps_to_get_filters_of_all_accounts)
