# 📧 সম্পূর্ণ গাইড: উবুন্টুতে Postfix, Dovecot এবং Roundcube ব্যবহার করে লিনাক্স মেইল সার্ভার কনফিগার করুন

এই গাইডটি আপনাকে উবুন্টুতে একটি **সম্পূর্ণ কার্যকর, প্রোডাকশন-রেডি মেইল সার্ভার** সেট আপ করতে সাহায্য করবে **Postfix (SMTP)**, **Dovecot (IMAP/POP3)**, এবং **Roundcube (ওয়েবমেইল)** ব্যবহার করে। আপনি শিখবেন কিভাবে ইমেল ডেলিভারি এবং রেপুটেশন নিশ্চিত করতে **SPF, DKIM, এবং DMARC** কনফিগার করতে হয় এবং একটি **কেন্দ্রীয় ওয়েব ইন্টারফেস** সেট আপ করে ব্যবহারকারীদের পরিচালনা করবেন।

---

## 📋 পূর্বশর্ত (Prerequisites)

শুরু করার আগে, নিশ্চিত করুন যে আপনার কাছে নিম্নলিখিতগুলি রয়েছে:

1.  **উবুন্টু 20.04 বা 22.04 LTS** সার্ভার (ফ্রেশ ইনস্টলেশন পছন্দনীয়)।
2.  `sudo` অধিকার সহ একটি **রুট নয় এমন ব্যবহারকারী**।
3.  একটি **নিবন্ধিত ডোমেইন নাম** (যেমন, `paulco.xyz`)।
4.  আপনার ডোমেইনের **DNS অ্যাক্সেস** যাতে আপনি `A`, `MX`, `TXT` রেকর্ড তৈরি করতে পারেন।
5.  আপনার সার্ভারের **স্ট্যাটিক পাবলিক IP ঠিকানা**।
6.  আপনার সার্ভারের ফায়ারওয়ালে পোর্ট **25 (SMTP)**, **587 (Submission)**, **465 (SMTPS)**, **143 (IMAP)**, **993 (IMAPS)**, **110 (POP3)**, **995 (POP3S)**, এবং **80/443 (HTTP/HTTPS)** খোলা থাকতে হবে।

---

## 🚀 ধাপ ১: প্রাথমিক সার্ভার সেটআপ

### ১.১. হোস্টনেম সেট করুন

হোস্টনেম আপনার ডোমেইনের একটি সাবডোমেইন হওয়া উচিত, যেমন `mail.yourdomain.com`।

```bash
sudo hostnamectl set-hostname mail.paulco.xyz
```

### ১.২. `/etc/hosts` ফাইল আপডেট করুন

সিস্টেম যেন নিজের হোস্টনেম রেজলভ করতে পারে তা নিশ্চিত করুন।

```bash
sudo nano /etc/hosts
```

নিচের লাইনটি যোগ করুন, `192.168.3.55` কে আপনার সার্ভারের **প্রাইভেট** IP ঠিকানা এবং `mail.paulco.xyz` কে আপনার FQDN দিয়ে প্রতিস্থাপন করুন:

```plaintext
192.168.3.55 mail.paulco.xyz mail
```

### ১.৩. হোস্টনেম যাচাই করুন

```bash
hostname -f
# আউটপুট হওয়া উচিত: mail.paulco.xyz
```

---

## 🧱 ধাপ ২: LAMP স্ট্যাক ইনস্টল করুন (Linux, Apache, MariaDB, PHP)

Roundcube, ওয়েবমেইল ক্লায়েন্ট, একটি ওয়েব সার্ভার, ডাটাবেস এবং PHP প্রয়োজন।

### ২.১. সিস্টেম আপডেট এবং মূল প্যাকেজ ইনস্টল করুন

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt install apache2 mariadb-server mariadb-client -y
```

### ২.২. PHP এবং প্রয়োজনীয় এক্সটেনশন ইনস্টল করুন

উবুন্টু 22.04-এ ডিফল্ট PHP হল 8.1। উবুন্টু 20.04-এ আপনি PHP 7.4 ব্যবহার করতে পারেন।

```bash
# Ondřej Surý PPA যোগ করুন (উবুন্টু 20.04-এর জন্য ঐচ্ছিক)
sudo apt-get update
sudo apt -y install software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt-get update

# PHP এবং এক্সটেনশন ইনস্টল করুন
sudo apt install php libapache2-mod-php php-mysql php-gd php-xml php-mbstring php-intl php-zip php-json php-curl php-bz2 php-gmp php-imap php-pear php-auth-sasl php-net-idna2 php-mail-mime php-net-ldap3 php-net-sieve -y
```

### ২.৩. Apache মডিউল সক্রিয় করুন

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 📮 ধাপ ৩: Postfix (SMTP সার্ভার) ইনস্টল এবং কনফিগার করুন

Postfix হল মেইল ট্রান্সফার এজেন্ট (MTA) যা ইমেল পাঠানো এবং গ্রহণের জন্য দায়ী।

### ৩.১. Postfix এবং মেইল ইউটিলিটি ইনস্টল করুন

```bash
sudo apt-get install postfix mailutils -y
```

ইনস্টলেশনের সময়, একটি কনফিগারেশন উইন্ডো আসবে:
*   **মেইল কনফিগারেশনের ধরন**: `Internet Site` সিলেক্ট করুন।
*   **সিস্টেম মেইল নাম**: আপনার ডোমেইন লিখুন (যেমন, `paulco.xyz`)। এটি অত্যন্ত গুরুত্বপূর্ণ।

### ৩.২. মূল Postfix কনফিগারেশন

আসল কনফিগারেশন ফাইলটি ব্যাকআপ নিন।

```bash
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
```

মূল কনফিগারেশন ফাইল সম্পাদনা করুন।

```bash
sudo nano /etc/postfix/main.cf
```

নিশ্চিত করুন যে নিম্নলিখিত সেটিংগুলি উপস্থিত বা আপডেট করা আছে। `mail.paulco.xyz` এবং `paulco.xyz` কে আপনার ডোমেইন দিয়ে প্রতিস্থাপন করুন।

```plaintext
# বেসিক সেটিং
myhostname = mail.paulco.xyz
mydomain = paulco.xyz
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4 # IPv6 থাকলে 'all' ব্যবহার করুন
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
home_mailbox = Maildir/ # Postfix কে Maildir ফরম্যাট ব্যবহার করতে বলে

# সিকিউরিটি এবং TLS (SSL পরে কনফিগার করব)
smtpd_use_tls = yes
smtpd_tls_security_level = may
smtpd_tls_cert_file = /etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file = /etc/ssl/private/ssl-cert-snakeoil.key

# SMTP সাবমিশন (পোর্ট 587) সক্রিয় করুন
smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_security_options = noanonymous
smtpd_sasl_local_domain = $myhostname
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination
```

### ৩.৩. `master.cf` ফাইলে SMTP সাবমিশন সক্রিয় করুন

`master.cf` ফাইল সম্পাদনা করে ইমেল ক্লায়েন্টের জন্য পোর্ট 587 সক্রিয় করুন।

```bash
sudo nano /etc/postfix/master.cf
```

`submission` সেকশনটি খুঁজুন এবং এটি আনকমেন্ট করুন। এটি নিচের মতো দেখতে হবে:

```plaintext
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
```

### ৩.৪. Postfix রিস্টার্ট করুন

```bash
sudo systemctl restart postfix
sudo systemctl status postfix # চেক করুন সার্ভার চালু আছে কিনা
```

---

## 📬 ধাপ ৪: Dovecot (IMAP/POP3 সার্ভার) ইনস্টল এবং কনফিগার করুন

Dovecot ব্যবহারকারীদের মেইলবক্সে ইমেল ডেলিভার করে এবং IMAP বা POP3 এর মাধ্যমে ইমেল পুনরুদ্ধার করতে দেয়।

### ৪.১. Dovecot ইনস্টল করুন

```bash
sudo apt install dovecot-core dovecot-imapd dovecot-pop3d dovecot-lmtpd -y
```

### ৪.২. Dovecot মেইল লোকেশন কনফিগার করুন

মেইল কনফিগারেশন ফাইল সম্পাদনা করুন।

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Postfix-এর `home_mailbox` সেটিংয়ের সাথে মিল রেখে মেইল লোকেশন সেট করুন।

```plaintext
mail_location = maildir:~/Maildir
```

অন্য কোনো `mail_location` লাইন কমেন্ট আউট করুন।

### ৪.৩. Dovecot অথেন্টিকেশন কনফিগার করুন

অথেন্টিকেশন কনফিগারেশন ফাইল সম্পাদনা করুন।

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

নিচের পরিবর্তনগুলি করুন:

```plaintext
disable_plaintext_auth = yes # এনক্রিপ্টেড লগইন বাধ্যতামূলক করুন
auth_mechanisms = plain login # PLAIN এবং LOGIN মেকানিজম অনুমোদন করুন

# সিস্টেম ইউজার অথেন্টিকেশন কমেন্ট আউট করুন
#!include auth-system.conf.ext

# পাসওয়ার্ড ফাইল অথেন্টিকেশন আনকমেন্ট করুন (ভার্চুয়াল ইউজারের জন্য)
!include auth-passwdfile.conf.ext
```

### ৪.৪. Postfix-এর জন্য Dovecot মাস্টার সকেট কনফিগার করুন

মাস্টার কনফিগারেশন ফাইল সম্পাদনা করুন।

```bash
sudo nano /etc/dovecot/conf.d/10-master.conf
```

`service auth` সেকশনটি খুঁজুন এবং Postfix-এর জন্য সকেট কনফিগার করুন।

```plaintext
service auth {
  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}
```

### ৪.৫. (ঐচ্ছিক) সিকিউর IMAP/POP3 সক্রিয় করুন

IMAPS (993) এবং POP3S (995) সক্রিয় করতে, `10-master.conf` ফাইলে সংশ্লিষ্ট সেকশন খুঁজুন এবং `port` এবং `ssl` লাইনগুলি আনকমেন্ট করুন।

```plaintext
# IMAPS-এর জন্য
inet_listener imaps {
  port = 993
  ssl = yes
}

# POP3S-এর জন্য
inet_listener pop3s {
  port = 995
  ssl = yes
}
```

### ৪.৬. Dovecot রিস্টার্ট করুন

```bash
sudo systemctl restart dovecot
sudo systemctl status dovecot # চেক করুন সার্ভার চালু আছে কিনা
```

---

## 🌐 ধাপ ৫: Roundcube ওয়েবমেইল ইনস্টল এবং কনফিগার করুন

Roundcube আপনার ব্যবহারকারীদের জন্য ইমেল পাঠানো এবং গ্রহণের জন্য একটি ব্যবহারকারী-বান্ধব ওয়েব ইন্টারফেস প্রদান করে।

### ৫.১. Roundcube ডাউনলোড এবং ইনস্টল করুন

```bash
cd /tmp
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.7/roundcubemail-1.6.7-complete.tar.gz
tar -xzf roundcubemail-1.6.7-complete.tar.gz
sudo mv roundcubemail-1.6.7 /var/www/html/roundcube
sudo chown -R www-data:www-data /var/www/html/roundcube/
sudo chmod 755 -R /var/www/html/roundcube/
```

### ৫.২. Roundcube-এর জন্য MariaDB ডাটাবেস তৈরি করুন

```bash
sudo mysql -u root
```

```sql
CREATE DATABASE roundcube_db CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'roundcube_user'@'localhost' IDENTIFIED BY 'YourStrongPasswordHere';
GRANT ALL PRIVILEGES ON roundcube_db.* TO 'roundcube_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### ৫.৩. Roundcube-এর ইনিশিয়াল ডাটাবেস স্কিমা ইম্পোর্ট করুন

```bash
sudo mysql -u roundcube_user -p roundcube_db < /var/www/html/roundcube/SQL/mysql.initial.sql
```

### ৫.৪. Apache ভার্চুয়াল হোস্ট তৈরি করুন

Roundcube-এর জন্য একটি নতুন কনফিগারেশন ফাইল তৈরি করুন।

```bash
sudo nano /etc/apache2/sites-available/roundcube.conf
```

নিচের কনফিগারেশন যোগ করুন। `mail.paulco.xyz` কে আপনার সার্ভারের FQDN দিয়ে প্রতিস্থাপন করুন।

```apache
<VirtualHost *:80>
    ServerName mail.paulco.xyz

    DocumentRoot /var/www/html/roundcube

    ErrorLog ${APACHE_LOG_DIR}/roundcube_error.log
    CustomLog ${APACHE_LOG_DIR}/roundcube_access.log combined

    <Directory /var/www/html/roundcube>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### ৫.৫. সাইট সক্রিয় করুন এবং Apache রিলোড করুন

```bash
sudo a2ensite roundcube.conf
sudo systemctl reload apache2
```

### ৫.৬. ওয়েব ইনস্টলার মাধ্যমে Roundcube সেটআপ সম্পন্ন করুন

1.  আপনার ওয়েব ব্রাউজারে `http://mail.paulco.xyz/roundcube/installer` ভিজিট করুন।
2.  ধাপগুলো অনুসরণ করুন:
    *   **এনভায়রনমেন্ট চেক**: নিশ্চিত করুন যে সমস্ত প্রয়োজনীয়তা পূরণ হয়েছে (সবকিছু সবুজ হওয়া উচিত)।
    *   **কনফিগ তৈরি করুন**: আপনার MariaDB বিবরণ (ডাটাবেস নাম, ব্যবহারকারীর নাম, পাসওয়ার্ড) লিখুন।
    *   **কনফিগ টেস্ট করুন**: ডাটাবেস কানেকশন এবং IMAP/SMTP সেটিং টেস্ট করুন।
        *   **IMAP সেটিং**:
            *   `IMAP হোস্ট`: `localhost`
            *   `IMAP পোর্ট`: `143`
            *   `IMAP সিকিউরিটি`: `STARTTLS`
        *   **SMTP সেটিং**:
            *   `SMTP হোস্ট`: `localhost`
            *   `SMTP পোর্ট`: `587`
            *   `SMTP সিকিউরিটি`: `STARTTLS`
            *   `SMTP ব্যবহারকারীর নাম`: `%u` (এটি Roundcube কে লগইন করা ব্যবহারকারীর নাম ব্যবহার করতে বলে)
            *   `SMTP পাসওয়ার্ড`: `%p` (এটি Roundcube কে লগইন করা পাসওয়ার্ড ব্যবহার করতে বলে)
3.  **ডাটাবেস ইনিশিয়ালাইজ করুন**: ডাটাবেস ইনিশিয়ালাইজ করার জন্য বাটনে ক্লিক করুন।
4.  **ফাইনালাইজিং**: `config.inc.php` ফাইলটি ডাউনলোড করুন এবং আপনার সার্ভারে `/var/www/html/roundcube/config/` ডিরেক্টরিতে আপলোড করুন। অথবা, অনুমতি থাকলে ইনস্টলার সরাসরি এটি লিখতে পারে।
5.  **সিকিউরিটি**: **ইনস্টলার ডিরেক্টরি মুছে ফেলুন**।

```bash
sudo rm -rf /var/www/html/roundcube/installer/
```

---

## 🔐 ধাপ ৬: ফায়ারওয়াল (UFW) কনফিগার করুন

মেইল এবং ওয়েব সার্ভিসের জন্য প্রয়োজনীয় পোর্টগুলি অনুমোদন করুন।

```bash
sudo ufw allow "Apache Full" # 80 এবং 443 অনুমোদন করে
sudo ufw allow "Postfix"     # 25 অনুমোদন করে
sudo ufw allow "Postfix Submission" # 587 অনুমোদন করে
sudo ufw allow "Postfix SMTPS"      # 465 অনুমোদন করে
sudo ufw allow 993/tcp              # IMAPS
sudo ufw allow 995/tcp              # POP3S
sudo ufw enable # ফায়ারওয়াল সক্রিয় করুন (যদি ইতিমধ্যে না থাকে)
sudo ufw status # নিয়মগুলি যাচাই করুন
```

---

## ✉️ ধাপ ৭: ইমেল অথেন্টিকেশন কনফিগার করুন (SPF, DKIM, DMARC)

এটি আপনার ইমেলগুলিকে স্প্যাম হিসাবে চিহ্নিত হওয়া থেকে রোধ করার জন্য অত্যন্ত গুরুত্বপূর্ণ।

### ৭.১. SPF রেকর্ড সেট আপ করুন

**SPF** বিশ্বকে জানায় যে কোন সার্ভারগুলি আপনার ডোমেইনের হয়ে ইমেল পাঠানোর অনুমতি পেয়েছে।

*   আপনার DNS ম্যানেজমেন্ট প্যানেলে যান।
*   একটি নতুন **TXT** রেকর্ড তৈরি করুন।
*   **নাম/হোস্ট**: `@` (বা আপনার DNS প্রদানকারীর উপর নির্ভর করে খালি রাখুন)।
*   **মান**: `v=spf1 mx a ip4:YOUR_SERVER_IP ~all`
    *   `YOUR_SERVER_IP` কে আপনার সার্ভারের পাবলিক IP ঠিকানা দিয়ে প্রতিস্থাপন করুন।
    *   `~all` মানে "সফট ফেইল" অন্যান্য সার্ভারের জন্য (শুরুর জন্য সুপারিশ করা হয়)।

### ৭.২. OpenDKIM ব্যবহার করে DKIM সেট আপ করুন

**DKIM** আপনার ইমেলগুলিতে একটি ডিজিটাল স্বাক্ষর যোগ করে, প্রমাণ করে যে সেগুলি পরিবর্তন করা হয়নি।

#### ৭.২.১. OpenDKIM ইনস্টল করুন

```bash
sudo apt install opendkim opendkim-tools -y
sudo gpasswd -a postfix opendkim # postfix ব্যবহারকারীকে opendkim গ্রুপে যোগ করুন
```

#### ৭.২.২. OpenDKIM কনফিগার করুন

মূল কনফিগারেশন ফাইল সম্পাদনা করুন।

```bash
sudo nano /etc/opendkim.conf
```

নিচের লাইনগুলি যোগ করুন বা আনকমেন্ট করুন। `paulco.xyz` কে আপনার ডোমেইন দিয়ে প্রতিস্থাপন করুন।

```plaintext
Syslog yes
LogWhy yes

# Canonicalization relaxed/simple
Mode sv
SubDomains no

# ভালো অপারেশনের জন্য যোগ করুন
AutoRestart yes
AutoRestartRate 10/1M
Background yes
DNSTimeout 5
SignatureAlgorithm rsa-sha256

# OpenDKIM ব্যবহারকারী
UserID opendkim

# সাইনিং এবং কী টেবিল
KeyTable refile:/etc/opendkim/key.table
SigningTable refile:/etc/opendkim/signing.table

# যাচাইকরণের জন্য উপেক্ষা করার হোস্ট, কিন্তু সাইন করার জন্য
ExternalIgnoreList /etc/opendkim/trusted.hosts
InternalHosts /etc/opendkim/trusted.hosts
```

#### ৭.২.৩. প্রয়োজনীয় ডিরেক্টরি এবং ফাইল তৈরি করুন

```bash
sudo mkdir /etc/opendkim
sudo mkdir /etc/opendkim/keys
sudo chown -R opendkim:opendkim /etc/opendkim
sudo chmod go-rw /etc/opendkim/keys

# সাইনিং টেবিল তৈরি করুন
sudo nano /etc/opendkim/signing.table
```

এই লাইনটি যোগ করুন (`paulco.xyz` প্রতিস্থাপন করুন):

```plaintext
*@paulco.xyz default._domainkey.paulco.xyz
```

```bash
# কী টেবিল তৈরি করুন
sudo nano /etc/opendkim/key.table
```

এই লাইনটি যোগ করুন (`paulco.xyz` প্রতিস্থাপন করুন):

```plaintext
default._domainkey.paulco.xyz paulco.xyz:default:/etc/opendkim/keys/paulco.xyz/default.private
```

```bash
# ট্রাস্টেড হোস্ট তৈরি করুন
sudo nano /etc/opendkim/trusted.hosts
```

এই লাইনগুলি যোগ করুন:

```plaintext
127.0.0.1
localhost
.paulco.xyz
```

#### ৭.২.৪. DKIM কী জেনারেট করুন

```bash
sudo mkdir /etc/opendkim/keys/paulco.xyz
sudo opendkim-genkey -b 2048 -d paulco.xyz -D /etc/opendkim/keys/paulco.xyz -s default -v
sudo chown opendkim:opendkim /etc/opendkim/keys/paulco.xyz/default.private
sudo chmod 600 /etc/opendkim/keys/paulco.xyz/default.private
```

#### ৭.২.৫. DNS-এ DKIM পাবলিক কী প্রকাশ করুন

পাবলিক কী দেখান।

```bash
sudo cat /etc/opendkim/keys/paulco.xyz/default.txt
```

আপনি এই আউটপুটটি দেখতে পাবেন:
`default._domainkey IN TXT ( "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC..." )`

*   আপনার DNS ম্যানেজমেন্ট প্যানেলে যান।
*   একটি নতুন **TXT** রেকর্ড তৈরি করুন।
*   **নাম/হোস্ট**: `default._domainkey` (এটি আপনার **সিলেক্টর**)।
*   **মান**: **প্রথম বন্ধনী `(...)` এর ভিতরের সবকিছু কপি করুন**, তারপর **সমস্ত ডাবল কোটেশন `"` এবং স্পেস সরিয়ে ফেলুন**। এটি একটি দীর্ঘ, ধারাবাহিক স্ট্রিং হওয়া উচিত যা `v=DKIM1;...` দিয়ে শুরু হয়।

#### ৭.২.৬. DKIM কী টেস্ট করুন

```bash
sudo opendkim-testkey -d paulco.xyz -s default -vvv
```

আউটপুটে `Key OK` দেখতে পাবেন। যদি `Key not secure` দেখেন, তাহলে চিন্তা করবেন না; এর মানে আপনি DNSSEC সেট আপ করেননি।

#### ৭.২.৭. OpenDKIM কে Postfix-এর সাথে সংযুক্ত করুন

OpenDKIM ডিফল্ট ফাইল সম্পাদনা করে Postfix-এর chroot jail-এর জন্য সকেট পাথ ঠিক করুন।

```bash
sudo nano /etc/default/opendkim
```

`SOCKET` লাইনটি খুঁজুন এবং এটি পরিবর্তন করুন:

```plaintext
SOCKET="local:/var/spool/postfix/opendkim/opendkim.sock"
```

এখন, সকেট ডিরেক্টরি তৈরি করুন এবং অনুমতি সেট করুন।

```bash
sudo mkdir /var/spool/postfix/opendkim
sudo chown opendkim:postfix /var/spool/postfix/opendkim
```

OpenDKIM কনফিগ আবার সম্পাদনা করুন যাতে সঠিক সকেট সেট করা যায়।

```bash
sudo nano /etc/opendkim.conf
```

`Socket` লাইনটি খুঁজুন (এটি কমেন্ট আউট থাকতে পারে) এবং এটি সেট করুন:

```plaintext
Socket local:/var/spool/postfix/opendkim/opendkim.sock
```

অবশেষে, Postfix কে OpenDKIM milter ব্যবহার করতে বলুন। `/etc/postfix/main.cf` সম্পাদনা করুন:

```bash
sudo nano /etc/postfix/main.cf
```

ফাইলের **শেষে** এই লাইনগুলি যোগ করুন:

```plaintext
# OpenDKIM-এর জন্য Milter কনফিগারেশন
milter_default_action = accept
milter_protocol = 6
smtpd_milters = local:opendkim/opendkim.sock
non_smtpd_milters = $smtpd_milters
```

#### ৭.২.৮. সার্ভিসগুলি রিস্টার্ট করুন

```bash
sudo systemctl restart opendkim
sudo systemctl restart postfix
```

### ৭.৩. DMARC রেকর্ড সেট আপ করুন

**DMARC** গ্রহণকারী সার্ভারগুলিকে বলে যে কী করতে হবে যদি একটি ইমেল SPF বা DKIM চেক ব্যর্থ হয়।

*   আপনার DNS ম্যানেজমেন্ট প্যানেলে যান।
*   একটি নতুন **TXT** রেকর্ড তৈরি করুন।
*   **নাম/হোস্ট**: `_dmarc`।
*   **মান**: `v=DMARC1; p=none; rua=mailto:postmaster@paulco.xyz; ruf=mailto:postmaster@paulco.xyz; fo=1`
    *   `p=none`: শুধুমাত্র মনিটর করুন, রিজেক্ট/কোয়ারেন্টাইন করবেন না। সবকিছু ঠিকঠাক কাজ করছে নিশ্চিত হওয়ার পর `p=quarantine` বা `p=reject` এ পরিবর্তন করুন।
    *   `postmaster@paulco.xyz` কে আপনার অ্যাডমিন ইমেল দিয়ে প্রতিস্থাপন করুন।

---

## 👥 ধাপ ৮: ব্যবহারকারী ম্যানেজমেন্ট (কেন্দ্রীয় ওয়েব ইন্টারফেস)

উপরের সেটআপটি সিস্টেম ব্যবহারকারীদের ব্যবহার করে (যারা `adduser` দিয়ে তৈরি হয়)। একটি কেন্দ্রীয় ওয়েব ইন্টারফেস থেকে ইমেল ব্যবহারকারীদের পরিচালনা করার জন্য, আপনাকে **ভার্চুয়াল ব্যবহারকারী** সেট আপ করতে হবে। এটি আরও জটিল কিন্তু প্রোডাকশন সার্ভারের জন্য স্ট্যান্ডার্ড।

### ৮.১. ভার্চুয়াল ব্যবহারকারী কেন?

*   **সিকিউরিটি**: ইমেল ব্যবহারকারীদের আপনার সার্ভারে শেল অ্যাক্সেস থাকবে না।
*   **স্কেলেবিলিটি**: শত শত ব্যবহারকারী পরিচালনা করা সহজ।
*   **কেন্দ্রীয় ম্যানেজমেন্ট**: একটি ডাটাবেসের সাথে ইন্টিগ্রেট করা যায় এবং একটি ওয়েব প্যানেল দিয়ে পরিচালনা করা যায়।

### ৮.২. ভার্চুয়াল ব্যবহারকারীর জন্য Postfix এবং Dovecot কনফিগার করুন

এটি একটি উল্লেখযোগ্য কনফিগারেশন পরিবর্তন। এখানে একটি উচ্চ-স্তরের ওভারভিউ দেওয়া হল:

1.  **ডাটাবেস সেটআপ**: একটি নতুন MariaDB ডাটাবেস (যেমন, `mailserver_db`) তৈরি করুন `virtual_domains`, `virtual_users`, এবং `virtual_aliases` টেবিল সহ।
2.  **Postfix কনফিগারেশন**:
    *   `/etc/postfix/main.cf` সম্পাদনা করুন যাতে SQL লুকআপ ব্যবহার করে ডোমেইন, মেইলবক্স এবং অ্যালিয়াসের জন্য।
    *   `virtual_mailbox_domains`, `virtual_mailbox_maps`, `virtual_alias_maps` কে SQL কনফিগারেশন ফাইলের পথে সেট করুন (যেমন, `/etc/postfix/mysql-virtual-mailbox-domains.cf`)।
    *   `virtual_mailbox_base = /var/mail/vhosts` এবং `virtual_uid_maps = static:5000`, `virtual_gid_maps = static:5000` সেট করুন।
3.  **Dovecot কনফিগারেশন**:
    *   `/etc/dovecot/conf.d/10-mail.conf` সম্পাদনা করুন যাতে `mail_location = maildir:/var/mail/vhosts/%d/%n` সেট করা যায়।
    *   `/etc/dovecot/conf.d/10-auth.conf` সম্পাদনা করুন যাতে `!include auth-passwdfile.conf.ext` আনকমেন্ট করা থাকে এবং `!include auth-system.conf.ext` কমেন্ট আউট করা থাকে।
    *   `/etc/dovecot/conf.d/auth-sql.conf.ext` সম্পাদনা করুন যাতে SQL অথেন্টিকেশন কনফিগার করা যায়।
    *   একটি `vmail` সিস্টেম ব্যবহারকারী এবং গ্রুপ (UID/GID 5000) তৈরি করুন এবং `/var/mail/vhosts` এর মালিকানা সেট করুন।
4.  **SQL কনফিগারেশন ফাইল তৈরি করুন**: `/etc/postfix/mysql-virtual-mailbox-domains.cf` এর মতো ফাইলগুলি তৈরি করুন যাতে ডাটাবেস কানেকশন বিবরণ এবং কোয়েরি থাকে।

### ৮.৩. একটি ওয়েব-ভিত্তিক কন্ট্রোল প্যানেল ইনস্টল করুন

এই ভার্চুয়াল ব্যবহারকারীদের একটি ওয়েব ইন্টারফেস থেকে পরিচালনা করতে, আপনি **PostfixAdmin** এর মতো একটি কন্ট্রোল প্যানেল ইনস্টল করতে পারেন।

#### ৮.৩.১. PostfixAdmin ইনস্টল করুন

```bash
cd /tmp
wget https://github.com/postfixadmin/postfixadmin/archive/refs/tags/postfixadmin-3.3.13.tar.gz
tar -xzf postfixadmin-3.3.13.tar.gz
sudo mv postfixadmin-postfixadmin-3.3.13 /var/www/html/postfixadmin
sudo chown -R www-data:www-data /var/www/html/postfixadmin
```

#### ৮.৩.২. PostfixAdmin কনফিগার করুন

1.  `http://mail.paulco.xyz/postfixadmin/setup.php` ভিজিট করুন।
2.  এটি আপনাকে একটি সেটআপ পাসওয়ার্ড তৈরি করতে এবং ডাটাবেস কনফিগার করতে নির্দেশনা দেবে।
3.  সেটআপ শেষে, এটি আপনাকে `config.local.php` ফাইলের জন্য একটি হ্যাশড পাসওয়ার্ড প্রদান করবে।
4.  `/var/www/html/postfixadmin/config.local.php` তৈরি করুন এবং ডাটাবেস এবং অ্যাডমিন ক্রেডেনশিয়াল যোগ করুন।
5.  **সিকিউরিটি**: `setup.php` ফাইলটি মুছে ফেলুন বা রিনেম করুন।

#### ৮.৩.৩. Roundcube-এর সাথে ইন্টিগ্রেট করুন (ঐচ্ছিক)

আপনি **Roundcube PostfixAdmin প্লাগইন** ইনস্টল করতে পারেন যাতে ব্যবহারকারীরা সরাসরি Roundcube থেকে তাদের নিজস্ব অ্যালিয়াস এবং ভ্যাকেশন মেসেজ পরিচালনা করতে পারে।

---

## 🧪 ধাপ ৯: আপনার মেইল সার্ভার টেস্ট করুন

1.  **একটি টেস্ট ইমেল পাঠান**: Roundcube-এ লগইন করুন (`http://mail.paulco.xyz/roundcube`) এবং একটি Gmail বা Outlook ঠিকানায় একটি ইমেল পাঠান।
2.  **ইমেল হেডার চেক করুন**: Gmail-এ, ইমেলটি খুলুন এবং "Show original" ক্লিক করুন। `spf=pass` এবং `dkim=pass` খুঁজুন।
3.  **অনলাইন টুল ব্যবহার করুন**:
    *   **Mail Tester**: `https://www.mail-tester.com` এ যান, প্রদত্ত ঠিকানায় একটি ইমেল পাঠান এবং আপনার স্কোর চেক করুন।
    *   **DKIM Validator**: `https://www.dmarcanalyzer.com/dkim/dkim-check/` ব্যবহার করে আপনার DKIM রেকর্ড যাচাই করুন।
    *   **SPF Validator**: `https://www.kitterman.com/spf/validate.html` ব্যবহার করে আপনার SPF রেকর্ড যাচাই করুন।

---

## 🛡️ ধাপ ১০: উন্নত সিকিউরিটি এবং রক্ষণাবেক্ষণ

*   **Fail2ban ইনস্টল করুন**: Postfix, Dovecot এবং SSH-এ ব্রুট-ফোর্স অ্যাটাক থেকে রক্ষা করে।
    ```bash
    sudo apt install fail2ban -y
    ```
*   **অটোমেটিক সিকিউরিটি আপডেট সেট আপ করুন**:
    ```bash
    sudo apt install unattended-upgrades -y
    sudo dpkg-reconfigure unattended-upgrades # 'Yes' সিলেক্ট করুন
    ```
*   **লগ মনিটর করুন**: নিয়মিত `/var/log/mail.log`, `/var/log/mail.err`, এবং `/var/log/syslog` চেক করুন।
*   **ব্যাকআপ**: নিয়মিত আপনার কনফিগারেশন ফাইল (`/etc/postfix`, `/etc/dovecot`, `/etc/opendkim`) এবং MariaDB ডাটাবেসগুলি ব্যাকআপ করুন।

---

## 📝 আপনার মূল কনটেন্টের সংশোধন এবং উন্নয়ন

*   **PHP ইনস্টলেশন**: উবুন্টু 20.04-এ ভালো PHP ভার্সন কন্ট্রোলের জন্য `ondrej/php` PPA যোগ করা হয়েছে।
*   **Postfix কনফিগারেশন**: আরও সম্পূর্ণ এবং সিকিউর `main.cf` কনফিগারেশন প্রদান করা হয়েছে, SASL অথেন্টিকেশন সহ।
*   **Dovecot কনফিগারেশন**: Postfix-এর সাথে `private/auth` সকেট ব্যবহার করে সঠিকভাবে ইন্টিগ্রেট করার জন্য কনফিগারেশন সংশোধন করা হয়েছে এবং `Maildir` সেট আপ করা হয়েছে।
*   **Roundcube ভার্সন**: একটি আরও সাম্প্রতিক, স্থিতিশীল ভার্সন (1.6.7) আপডেট করা হয়েছে।
*   **ফায়ারওয়াল**: IMAPS (993) এবং POP3S (995) এর জন্য নিয়ম যোগ করা হয়েছে।
*   **SPF/DKIM/DMARC**: ইমেল ডেলিভারি নিশ্চিত করার জন্য সমস্ত তিনটির জন্য একটি সম্পূর্ণ, ধাপে ধাপে গাইড প্রদান করা হয়েছে। আপনার মূল গাইডে শুধুমাত্র SPF আংশিকভাবে কভার করা ছিল।
*   **ব্যবহারকারী ম্যানেজমেন্ট**: **ভার্চুয়াল ব্যবহারকারী** এবং একটি **ওয়েব-ভিত্তিক কন্ট্রোল প্যানেল (PostfixAdmin)** সেট আপ করার জন্য একটি নিবেদিত বিভাগ যোগ করা হয়েছে, যা একটি গুরুত্বপূর্ণ অনুপস্থিত অংশ ছিল।
*   **টেস্টিং**: আপনার সার্ভার সম্পূর্ণরূপে টেস্ট করার জন্য নির্দিষ্ট ধাপ এবং টুল যোগ করা হয়েছে।
*   **স্ট্রাকচার**: একটি মসৃণ সেটআপ প্রক্রিয়ার জন্য যৌক্তিক, কালানুক্রমিক ক্রমে ধাপগুলি পুনর্গঠন করা হয়েছে।
*   **স্পষ্টতা**: গোটা গাইড জুড়ে আরও ব্যাখ্যামূলক মন্তব্য এবং প্রেক্ষাপট যোগ করা হয়েছে।

এই গাইডটি একটি স্ব-হোস্টেড ইমেল সার্ভার সেট আপ করার জন্য একটি শক্তিশালী ভিত্তি প্রদান করে। মনে রাখবেন যে একটি মেইল সার্ভার বজায় রাখা সিকিউরিটি, রেপুটেশন এবং ডেলিভারি নিশ্চিত করার জন্য চলমান প্রচেষ্টা প্রয়োজন।
