---
title: "Zimbra: Delegated Admin untuk Manajemen Quota"
date: 2024-02-13T20:09:11+07:00
tags:
- zimbra
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

Pada artikel kali ini, saya akan membahas mengenai cara membuat _Delegate Admin_ dengan _permission_ yang sangat terbatas. Dalam kasus ini, hanya sebagai manajemen quota saja. Seperti menambah atau mengurangi mailbox quota user.

Solusi ini bisa diterapkan jika ingin membuat satu akun yang didelegasikan untuk orang tertentu namun tidak ingin memberikan akses admin zimbra standar.

# Sintaks

Sebagai user `zimbra`:

```bash
# Jika user belum dibuat
zmprov ca user@domain.tld <password> zimbraIsAdminAccount TRUE
# Jika user sudah dibuat
zmprov ma user@domain.tld zimbraIsAdminAccount TRUE

# Agar bisa melist akun
zmprov ma user@domain.tld zimbraAdminConsoleUIComponents accountListView

# Agar bisa mengganti quota mailbox
zmprov ma user@domain.tld zimbraDomainAdminMaxMailQuota 0

# Agar bisa melakukan Edit akun
zmprov grr domain domain.tld usr user@domain.tld getAccount
zmprov grr domain domain.tld usr user@domain.tld getAccountInfo
zmprov grr domain domain.tld usr user@domain.tld getMailboxInfo
zmprov grr domain domain.tld usr user@domain.tld listAccount

# Agar bisa memodifikasi quota mailbox
zmprov grr domain domain.tld usr user@domain.tld set.account.zimbraMailQuota
```

# Penutup

Dengan demikian, user yang didelegasikan akan bisa login sebagai "admin" dengan akses yang sangat terbatas. Kolom-kolom yang muncul di dalam menu `Edit` seluruhnya bersifat `read-only` kecuali bagian *Quota Mailbox*.

Semoga bermanfaat, _barakallhu fiikum_.
