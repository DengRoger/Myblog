---
title: Mail server
date: 2023-06-19 09:16:17
tags:
---

<!-- more -->

# Instailation
```bash
sudo apt-get install opendkim opendkim-tools postfix

# add postfix user to opendkim group
sudo usermod -a -G opendkim postfix
```

# 新增使用者
```bash
useradd -d -g mail -s /sbin/nologin me
```

# 設定主機名
```bash
sudo hostnamectl set-hostname mail.rogerdeng.net
```

# 設定 SSL
```bash
sudo apt-get install certbot
sudo mkdir /etc/ssl/mail
cd /etc/ssl/mail
sudo openssl req -newkey rsa:2048 -nodes -sha512 -x509 -days 365 -nodes -out public_cert.pem -keyout private_key.pem
```

- Postfix ssl 設定 /etc/postfix/main.cf
```bash
smtp_use_tls = yes
smtpd_use_tls = yes
smtp_tls_note_starttls_offer = yes
smtpd_tls_cert_file=/etc/ssl/mail/public_cert.pem
smtpd_tls_key_file=/etc/ssl/mail/private_key.pem
smtpd_tls_security_level=may
smtpd_tls_loglevel = 1
```

- Dovecot 安裝
```bash
sudo apt-get install dovecot-core dovecot-imapd dovecot-pop3d dovecot-lmtpd dovecot-mysql
```

- 設定 /etc/dovecot/conf.d/10-auth.conf
```bash
unix_listener auth-userdb {
    #mode = 0666
    #user =
    #group =
}

# Postfix smtp-auth
unix_listener /var/spool/postfix/private/auth {
    mode = 0666
}
```

# 簽署 DKIM
```bash
sudo mkdir -p /etc/opendkim/keys
sudo chown -R opendkim:opendkim /etc/opendkim/keys
sudo chmod  744 /etc/opendkim/keys 
cd /etc/opendkim/keys
sudo mkdir rogerdeng.net
sudo mkdir mail.rogderdeng.net
sudo chown -R root:root rogerdeng.net
sudo chown -R root:root mail.rogerdeng.net
sudo chmod  755 rogerdeng.net
sudo chmod  755 mail.rogerdeng.net
sudo opendkim-genkey -b 2048 -d rogerdeng.net -D /etc/opendkim/keys/rogerdeng.net -s default -v 
sudo opendkim-genkey -b 2048 -d mail.rogerdeng.net -D /etc/opendkim/keys/mail.rogerdeng.net -s default -v
sudo chown opendkim:opendkim /etc/opendkim/keys/rogerdeng.net/default.private
sudo chown opendkim:opendkim /etc/opendkim/keys/mail.rogerdeng.net/default.private
```

# 設定 DKIM 到 DNS Server
```bash
sudo cat /etc/opendkim/keys/rogerdeng.net/default.txt
sudo cat /etc/opendkim/keys/mail.rogerdeng.net/default.txt
# 你會得到 default._domainkey	IN	TXT  ...
#        default._domainkey    IN	TXT ...
# (根據自己的Dns Server Zone 做修改)
# default._domainkey	    IN	TXT ...
# default._domainkey.mail	IN	TXT ...

ssh <DNS Server>
sudo vim /etc/bind/foward.rogerdeng.net # 寫入 DKIM
# @       IN      MX  10  mail.rogerdeng.net. 
# mail    IN      A       <IP>
# @       IN	  TXT	"v=spf1 a mx ipv4:<IP> -all"
# mail    IN	  TXT	"v=spf1 a mx ipv4:<IP> -all"
# update u're serial
sudo vim /etc/bind/internal.rogerdeng.net
# setup dns record
sudo service bind9 reload
```

## 測試 DNS record
```bash
sudo opendkim-testkey -d rogerdeng.net -s default -vvv 
sudo opendkim-testkey -d mail.rogerdeng.net -s default -vvv
```

# 設定 OpenDKIM
```bash
sudo vim /etc/opendkim.conf
```

- 設定 /etc/opendkim.conf
```bash 
Syslog			    yes
SyslogSuccess		yes
LogWhy			    no

Canonicalization	relaxed/simple
Mode			    sv
SubDomains		    no
OversignHeaders		From

AutoRestart			yes
AutoRestartRate		10/1M
Background			yes
DNSTimeout			5
SignatureAlgorithm	rsa-sha256

UserID			opendkim
UMask			007

KeyTable            refile:/etc/opendkim/key.table
SigningTable        refile:/etc/opendkim/signing.table
ExternalIgnoreList  /etc/opendkim/trusted.hosts
InternalHosts       /etc/opendkim/trusted.hosts
```

- 設定 /etc/default/opendkim
```bash
rogerdeng.net    	default._domainkey.rogerdeng.net
mail.rogerdeng.net    	default._domainkey.mail.rogerdeng.net
```

- 設定 /etc/opendkim/trusted.hosts
```bash
127.0.0.1
localhost

.rogerdeng.net
.mail.rogerdeng.net
```

- Restart OpenDKIM
```bash
sudo systemctl restart opendkim 
```

# 讓 Postfix 使用 OpenDKIM
- 編輯 /etc/postfix/main.cf
```bash
Socket			local:/var/spool/postfix/opendkim/opendkim.sock
```

- 編輯 /etc/default/opendkim
```bash
sudo mkdir /var/spool/postfix/opendkim 
sudo chown opendkim:postfix /var/spool/postfix/opendkim
```

```bash
SOCKET="local:/var/spool/postfix/opendkim/opendkim.sock"
```

## OpenDKIM signed for Postfix
```bash
milter_default_action = accept
milter_protocol = 6
smtpd_milters = local:opendkim/opendkim.sock inet:localhost:8891
non_smtpd_milters = $smtpd_milters
```

## restart postfix and opendkim
```bash
sudo systemctl restart postfix
sudo systemctl restart opendkim
sudo systemctl enable opendkim
sudo systemctl enable postfix
```

