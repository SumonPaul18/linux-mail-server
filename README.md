# 🌐 How to Configure a Full Linux Mail Server on Ubuntu (Postfix + Dovecot + Roundcube)

> This guide helps you build a **cost-free, full-featured enterprise mail server** using open-source tools: **Postfix, Dovecot, and Roundcube**, on Ubuntu 22.04.

---

## 📦 Components

1. **Postfix** – Mail Transfer Agent (MTA): Responsible for sending & receiving emails.
2. **Dovecot** – Mail Delivery Agent (MDA): Handles POP3/IMAP access and mail storage.
3. **Roundcube** – Web-based email client (Webmail): User-friendly interface to manage mails via browser.

---

## ✅ Requirements

* ✅ Valid Domain (e.g., `paulco.xyz`)
* ✅ DNS Server (with A, MX, SPF records)
* ✅ Ubuntu 22.04.2 LTS (4 CPU, 8 GB RAM, 50 GB Disk, 1 Public IP)

---

## 🔧 Basic Configuration

```bash
# Check OS version
cat /etc/os-release
hostnamectl

# Check CPU, RAM, Disk
lscpu
nproc
free -h
df -h

# Set hostname
sudo hostnamectl set-hostname mail.paulco.xyz

# Edit hosts file
sudo nano /etc/hosts
192.168.3.55 mail.paulco.xyz mail

# Confirm FQDN
hostname -f

# Add mail user
sudo adduser mail
sudo usermod -aG sudo mail
su - mail
```

---

## 📬 Install & Configure Postfix (MTA)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install postfix -y
```

➡️ During setup, set **System Mail Name** as `mail.paulco.xyz`.

### Postfix Configuration

```bash
# Check Postfix version
postconf mail_version

# Reconfigure if needed
sudo dpkg-reconfigure postfix

# Backup config and set Maildir
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
sudo nano /etc/postfix/main.cf
home_mailbox = Maildir/

sudo systemctl restart postfix
```

---

## 🧪 Test Postfix

```bash
sudo apt install mailutils -y
echo "Test mail body" | mail -s "Test Subject" sumonpaul267@gmail.com

# Tail mail logs
tail -f /var/log/mail.log
```

Test SMTP:

```bash
telnet smtp.gmail.com 587
telnet gmail-smtp-in.l.google.com 25
```

---

## 📨 Install Dovecot (IMAP/POP3)

```bash
sudo apt install dovecot-imapd dovecot-pop3d -y
sudo systemctl restart dovecot

# Edit main Dovecot config
sudo nano /etc/dovecot/dovecot.conf
protocols = imap imaps pop3 pop3s
listen = *

# Authentication settings
sudo nano /etc/dovecot/conf.d/10-auth.conf
disable_plaintext_auth = no

# Mail storage
sudo nano /etc/dovecot/conf.d/10-mail.conf
mail_location = maildir:~/Maildir
```

Restart:

```bash
sudo systemctl restart dovecot
sudo systemctl restart postfix
```

---

## 🌐 Install Roundcube Webmail

```bash
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql -y
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### Install Required PHP Packages

```bash
sudo apt install php php-mysql php-imap php-xml php-mbstring php-intl php-zip php-json php-pear php-gmp php-bz2 php-auth-sasl php-mail-mime php-net-smtp php-net-idna2 php-net-ldap3 php-net-sieve -y
```

If PHP fails to install:

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```

---

### Roundcube Setup

```bash
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.1/roundcubemail-1.6.1-complete.tar.gz
tar -xvzf roundcubemail-1.6.1-complete.tar.gz
sudo mv roundcubemail-1.6.1 /var/www/html/roundcubemail
sudo chown -R www-data:www-data /var/www/html/roundcubemail/
```

Create MySQL Database:

```sql
sudo mysql
CREATE DATABASE roundcube;
CREATE USER 'roundcubeuser'@'localhost' IDENTIFIED BY 'ubuntu';
GRANT ALL PRIVILEGES ON roundcube.* TO 'roundcubeuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Import DB Schema:

```bash
sudo mysql roundcube < /var/www/html/roundcubemail/SQL/mysql.initial.sql
```

---

### Configure Apache Virtual Host

```bash
sudo nano /etc/apache2/sites-available/roundcube.conf
```

Paste:

```apache
<VirtualHost *:80>
  ServerName mail.paulco.xyz
  DocumentRoot /var/www/html/roundcubemail/
  ErrorLog ${APACHE_LOG_DIR}/roundcube_error.log
  CustomLog ${APACHE_LOG_DIR}/roundcube_access.log combined

  <Directory /var/www/html/roundcubemail/>
    Options FollowSymLinks MultiViews
    AllowOverride All
    Require all granted
  </Directory>
</VirtualHost>
```

```bash
sudo a2ensite roundcube.conf
sudo systemctl reload apache2
```

➡️ Visit: `http://mail.paulco.xyz/roundcubemail/installer/`

⚠️ After setup:

```bash
sudo rm -r /var/www/html/roundcubemail/installer/
```

---

## 📌 Setup DNS Records (SPF, DKIM, DMARC)

### ✅ SPF Record

* Add the following TXT record in your domain DNS panel:

```
TXT  @  "v=spf1 a mx ip4:202.4.111.69 ~all"
```

Check via terminal:

```bash
dig txt paulco.xyz
nslookup
> set type=txt
> paulco.xyz
```

---

## 🔐 Optional: SPF Policy Agent (Postfix Integration)

```bash
sudo apt install postfix-policyd-spf-python -y

# Edit master.cf
sudo nano /etc/postfix/master.cf
# Add at the bottom:
policyd-spf  unix  -       n       n       -       0       spawn
  user=policyd-spf argv=/usr/bin/policyd-spf
```

```bash
# Edit main.cf
sudo nano /etc/postfix/main.cf

# Add:
policyd-spf_time_limit = 3600
smtpd_recipient_restrictions =
   permit_mynetworks,
   permit_sasl_authenticated,
   reject_unauth_destination,
   check_policy_service unix:private/policyd-spf
```

Restart:

```bash
sudo systemctl restart postfix
```

---

## ✅ Final Checklist

| Component | Status                              |
| --------- | ----------------------------------- |
| Postfix   | ✅ Installed & tested                |
| Dovecot   | ✅ IMAP/POP3 ready                   |
| Roundcube | ✅ Webmail operational               |
| SPF       | ✅ DNS record added                  |
| Hostname  | ✅ mail.paulco.xyz                   |
| Firewall  | 🔒 Open ports 25, 80, 443, 587, 993 |

---

## 📚 References

* [LinuxBabe - SPF, DKIM, DMARC](https://www.linuxbabe.com/mail-server/setting-up-dkim-and-spf)
* [Postfix Official Docs](http://www.postfix.org/)
* [Dovecot Documentation](https://doc.dovecot.org/)
* [Roundcube](https://roundcube.net/)

---
