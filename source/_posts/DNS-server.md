---
title: DNS server
date: 2023-06-19 09:10:52
categories:
    - System
tags:
    - DNS
    - bind9
---
# bind9 DNS server
<!-- more -->

## bind9 view 做分割

- spec
    - 外網無法詢問內部 dns record
    - 內網可以查詢外網及內網 dns record

```
acl internal {
    10.1.1.0/24;
};

acl external {
    0.0.0.0/0;
};

view "internal-view" {
    match-clients { internal; };
    zone "rogerdeng.net" {
        type master;
        file "/etc/bind/internal.rogerdeng.net.db";
    };
};

view "external-view" {
    match-clients { external; };
    zone "rogerdeng.net" {
        type master;
        file "/etc/bind/forward.rogerdeng.net.db";
    };
};
```

- 讓內網查詢外網 dns record
    - insert `$INCLUDE /etc/bind/forward.rogerdeng.net.db` into `internal.rogerdeng.net.db`

## DNSSEC 

1. 產生 KSK
    - `dnssec-keygen -f KSK -a RSASHA256 -b 2048 -n ZONE <ZONENAME>`
3. 產生 ZSK
    - `dnssec-keygen -a RSASHA256 -b 2048 -n ZONE <ZONENAME>`
5. 將 KSK 與 ZSK 的 record 放入 zonefile
    - `cat *.key >> <ZONEFILE>`
7. 簽署 zonefile
    - `dnssec-signzone -u -o rogerdeng.net -k <KSK.key> <ZONEFILE> <ZSK.key>`
9. 指定 zonefile.signed
    ```
    view "external-view" {
        match-clients { external; };
        zone "rogerdeng.net" {
            type master;
            file "/etc/bind/forward.rogerdeng.net.db.signed";
        };
    };
    ```
11. 產生 DS record 放入上層 dns server 
    - `dnssec-dsfromkey <KSK.key>`
    - `rogerdeng.net. IN DS 10246 8 2 FB2C50A9AB6561B75D7033DDA209E2C8BBEF7138274F325691DAD456582EAAF8`
    - ![](https://hackmd.io/_uploads/HJciLTvU3.png)

