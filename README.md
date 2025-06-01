# ☁️ Configure a Full-Featured Linux Mail Server on Ubuntu

In this tutorial, you'll learn how to **configure a fully functional and cost-free mail server** using **Postfix, Dovecot, and Roundcube** on Ubuntu 22.04.2 LTS. This setup is suitable for enterprise-grade mail communication and includes optional SPF, DKIM, and DMARC for security.


## 🛠 Tools Used

| Tool          | Role                                                   |
| ------------- | ------------------------------------------------------ |
| **Postfix**   | Mail Transfer Agent (MTA) for sending/receiving emails |
| **Dovecot**   | Mail Delivery Agent (MDA) for POP3/IMAP access         |
| **Roundcube** | Webmail interface for users                            |

---

## 🔰 Prerequisites

* Valid domain: `paulco.xyz`
* DNS Server (or Domain Panel access)
* Ubuntu 22.04.2 LTS

  * 4 CPU
  * 8 GB RAM
  * 50 GB Disk
  * Public IP

---

## 🧱 Basic System Configuration

```bash
# Check OS version
cat /etc/os-release
hostnamectl

# Check CPU info
lscpu
nproc

# Check RAM
free -h

# Check disk usage
df -h

# Change hostname
sudo hostnamectl set-hostname mail.paulco.xyz

# Update /etc/hosts
sudo nano /etc/hosts
192.168.3.55 mail.paulco.xyz mail

# Check FQDN
hostname -f

# Create new user
sudo adduser mail
sudo usermod -aG sudo mail
su - mail
```

---

## 📬 Install & Configure Postfix (MTA)

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install postfix -y
```

> 💡 During setup, set "System mail name" to your domain name (e.g., `paulco.xyz`)

```bash
# Check version and status
postconf mail_version
sudo systemctl status postfix

# Reconfigure if needed
sudo dpkg-reconfigure postfix

# Enable Maildir format
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
sudo nano /etc/postfix/main.cf
# Add or edit:
home_mailbox = Maildir/

# Restart service
sudo systemctl restart postfix
```

---

## ✅ Test Postfix Setup

```bash
# Install mail utilities
sudo apt install mailutils -y

# Send test email
echo "Test from Postfix" | mail -s "Hello Postfix" your@email.com

# Tail logs
tail -f /var/log/mail.log

# Telnet test (external)
telnet smtp.gmail.com 587
```

---

## 📥 Install & Configure Dovecot (IMAP & POP3)

```bash
sudo apt install dovecot-imapd dovecot-pop3d -y
sudo systemctl restart dovecot
sudo systemctl status dovecot
```

### Configuration

```bash
# Backup and edit main config
sudo cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.bak
sudo nano /etc/dovecot/dovecot.conf
# Add:
protocols = imap pop3
listen = *

# Edit authentication
sudo nano /etc/dovecot/conf.d/10-auth.conf
disable_plaintext_auth = no

# Configure mail storage
sudo nano /etc/dovecot/conf.d/10-mail.conf
mail_location = maildir:~/Maildir
# Comment out mbox line if exists

# Restart services
sudo systemctl restart dovecot
sudo systemctl restart postfix
```

---

## 📧 Install Roundcube Webmail

### Prerequisites

```bash
# Apache, MariaDB, PHP
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql -y
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### PHP Modules

```bash
sudo apt install openssl composer php-net-smtp php-mysql php-gd php-xml php-mbstring php-intl php-zip php-json php-pear php-bz2 php-gmp php-imap php-auth-sasl php-net-idna2 php-mail-mime php-net-ldap3 php-net-sieve -y
```

> 🔁 **If PHP version errors occur**, install from `ppa:ondrej/php`.

---

### Install Roundcube

```bash
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.1/roundcubemail-1.6.1-complete.tar.gz
tar -xvf roundcubemail-1.6.1-complete.tar.gz
sudo mv roundcubemail-1.6.1 /var/www/html/roundcubemail
sudo chown -R www-data:www-data /var/www/html/roundcubemail/
sudo chmod 755 -R /var/www/html/roundcubemail/
```

### Setup Database

```bash
sudo mysql -u root
CREATE DATABASE roundcube DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'roundcubeuser'@'localhost' IDENTIFIED BY 'ubuntu';
GRANT ALL PRIVILEGES ON roundcube.* TO 'roundcubeuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

sudo mysql roundcube < /var/www/html/roundcubemail/SQL/mysql.initial.sql
```

---

### Apache Virtual Host

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

Now visit:

```
http://mail.paulco.xyz/roundcubemail/installer/
```

> ⚠️ After installation, delete installer directory:

```bash
sudo rm -r /var/www/html/roundcubemail/installer/
```

---

## 📡 Configure SPF Record (Basic Email Authentication)

### Add SPF DNS Record

```txt
v=spf1 a mx ip4:202.4.111.69 ~all
```

You can verify:

```bash
dig txt paulco.xyz
nslookup
> set type=txt
> paulco.xyz
```

---

## ⚙️ Optional: Enforce SPF with Postfix

```bash
sudo apt install postfix-policyd-spf-python -y
```

### Edit `/etc/postfix/master.cf`

```ini
policyd-spf  unix  -       n       n       -       0       spawn
  user=policyd-spf argv=/usr/bin/policyd-spf
```

### Edit `/etc/postfix/main.cf`

```ini
policyd-spf_time_limit = 3600
smtpd_recipient_restrictions =
    permit_mynetworks,
    permit_sasl_authenticated,
    reject_unauth_destination,
    check_policy_service unix:private/policyd-spf
```

```bash
sudo systemctl restart postfix
```

---

## 🔐 Coming Up Next

> Add **DKIM & DMARC Records** for enhanced email authenticity and protection from spoofing.
> Refer: [https://www.linuxbabe.com/mail-server/setting-up-dkim-and-spf](https://www.linuxbabe.com/mail-server/setting-up-dkim-and-spf)

---

## ✅ Final Checklist

* [x] Domain name configured
* [x] DNS records (A, MX, SPF) setup
* [x] Postfix and Dovecot installed and tested
* [x] Roundcube webmail working at `http://mail.paulco.xyz`
* [x] Outgoing and incoming mail tested
* [x] Logs verified with `tail -f /var/log/mail.log`

---

