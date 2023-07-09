---
title: Mail server
date: 2023-06-19 09:16:17
tags:
    - Mail
    - Postfix
    - Dovecot
    - OpenDKIM
    - SPF
    - DKIM
    - DMARC
    - Greylisting
categories:
    - System
---

# Setup Mail Server
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

ssl = yes

ssl_cert = <證書位置
ssl_key = <私鑰位置
```

# 測試 SMTP IMAP

## SMTP
```bash
# using base64 encode account and passwd 
echo -n "<account or passwd>" | base64

telnet DomainName 25
ehlo DomainName
auth login
<account> # base64 encode
<passwd> # base64 encode
# 若成功會出現 235 Authentication successful
# 若失敗會出現 535 5.7.8 Error: authentication failed: authentication failure

# 寄信
mail from: <account>
rcpt to: <account>
data
subject: test
test
.
quit
```

## IMAP
```bash
telnet DomainName 143
A01 LOGIN <account> <passwd>
# 若成功會出現 OK LOGIN Ok.
A02 LIST "" *
A03 SELECT INBOX
A04 SEARCH ALL
A05 FETCH 32 FULL
A06 LOGOUT
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

# 防止冒名發信
- 安裝資料庫
```bash
sudo apt-get install mariadb-server
sudo mysql_secure_installation
```

- 編輯 /etc/postfix/main.cf
```bash
non_smtpd_milters = $smtpd_milters

smtpd_sender_login_maps = regexp:/etc/postfix/sender_login_map
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated reject_unauth_destination
smtpd_sender_restrictions = reject_non_fqdn_sender reject_unknown_sender_domain reject_sender_login_mismatch
```

- 編輯 /etc/postfix/sender_login_map
```bash
/^(.+)@(mail\.)?rogerdeng\.net$/ $1
```

- 編輯 /etc/postfix/sender_check
```bash
<>      REJECT  null users are not allowed
```

# SPF、DKIM 和 DMARC 的郵件輸入檢查
- 安裝 opendmarc
```bash
sudo apt-get install opendmarc
```

- 編輯 /etc/postfix/main.cf
```bash
milter_default_action = accept
milter_protocol = 6
smtpd_milters = inet:127.0.0.1:8893 local:opendkim/opendkim.sock
```
- 編輯 /etc/opendkim.conf
```bash
UserID opendmarc

Socket inet:8893@localhost
SoftwareHeader true
SPFIgnoreResults true
SPFSelfValidate true
Syslog true
UMask 777
UserID opendmarc:mail
TrustedAuthservIDs mail.rogerdeng.net
RejectFailures true
RequiredHeaders false
IgnoreAuthenticatedClients true
```

- 加入開機啟動
```bash
systemctl enable opendmarc
systemctl start opendmarc
```

# 設定 Greylisting 
- 安裝
```bash
sudo apt-get install postgrey
```
- 編輯 /etc/sysconfig/postgrey
```bash
POSTGREY_OPTS="--delay=30"
```

- 新增白名單(self) 編輯 /etc/postfix/postgrey_whitelist_clients.local
```bash
rogerdeng.net
mail.rogerdeng.net
```

- 編輯 /etc/postfix/main.cf
```bash
smtpd_recipient_restrictions = check_policy_service unix:postgrey/socket
```

```bash
systemctl enable postgrey
systemctl restart postgrey
```

#合併並確認 main.cf
```bash
smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# See http://www.postfix.org/COMPATIBILITY_README.html -- default to 2 on
# fresh installs.
compatibility_level = 2
broken_sasl_auth_clients = yes
smtpd_sasl_type = dovecot
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
smtpd_sasl_tls_security_options = $smtpd_sasl_security_options
smtpd_sasl_path = private/auth

# TLS parameters
#ssl_cert = </etc/dovecot/private/dovecot.pem
#ssl_key = </etc/dovecot/private/dovecot.key
smtp_use_tls = yes
smtpd_use_tls = yes
smtp_tls_note_starttls_offer = yes
smtpd_tls_cert_file=/etc/ssl/mail/public_cert.pem
smtpd_tls_key_file=/etc/ssl/mail/private_key.pem
smtpd_tls_security_level=may
smtpd_tls_loglevel = 1

smtp_tls_CApath=/etc/ssl/certs
smtp_tls_security_level=may
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache


mydomain = rogerdeng.net
myhostname = mail.rogerdeng.net
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = $mydomain
mydestination = mail.rogerdeng.net, rogerdeng.net, localhost.localdomain, localhost
relayhost =
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all
mynetworks = 10.1.1.0/24, 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128

# Milter configuration
non_smtpd_milters = $smtpd_milters

smtpd_sender_login_maps = regexp:/etc/postfix/sender_login_map
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated reject_unauth_destination
smtpd_sender_restrictions = reject_non_fqdn_sender reject_unknown_sender_domain reject_sender_login_mismatch
milter_default_action = accept
milter_protocol = 6
smtpd_milters = inet:127.0.0.1:8893 local:opendkim/opendkim.sock
```