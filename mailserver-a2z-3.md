Here is your comprehensive, corrected, and enhanced step-by-step guide to configuring a Linux mail server on Ubuntu, presented in Markdown format.

---

# 📧 Ultimate Guide: How to Configure a Linux Mail Server on Ubuntu with Postfix, Dovecot, and Roundcube

This guide will walk you through setting up a **fully functional, production-ready mail server** on Ubuntu using **Postfix (SMTP)**, **Dovecot (IMAP/POP3)**, and **Roundcube (Webmail)**. You will also learn how to configure **SPF, DKIM, and DMARC** for email deliverability and set up a **central web interface to manage users**.

---

## 📋 Prerequisites

Before you begin, ensure you have the following:

1.  A **fresh Ubuntu 20.04 or 22.04 LTS server**.
2.  A **non-root user** with `sudo` privileges.
3.  A **registered domain name** (e.g., `paulco.xyz`).
4.  A **static public IP address** for your server.
5.  **DNS access** to your domain to create `A`, `MX`, `TXT` records.
6.  Port `25` (SMTP), `587` (Submission), `465` (SMTPS), `143` (IMAP), `993` (IMAPS), `110` (POP3), `995` (POP3S), and `80/443` (HTTP/HTTPS) must be open on your server's firewall.

---

## 🚀 Step 1: Initial Server Setup

### 1.1. Set the Hostname

The hostname should be a subdomain like `mail.yourdomain.com`.

```bash
sudo hostnamectl set-hostname mail.paulco.xyz
```

### 1.2. Update `/etc/hosts`

Edit the hosts file to ensure the system can resolve its own hostname.

```bash
sudo nano /etc/hosts
```

Add the following line, replacing `192.168.3.55` with your server's **private** IP address and `mail.paulco.xyz` with your FQDN:

```plaintext
192.168.3.55 mail.paulco.xyz mail
```

### 1.3. Verify the Hostname

```bash
hostname -f
# Output should be: mail.paulco.xyz
```

---

## 🧱 Step 2: Install LAMP Stack (Linux, Apache, MariaDB, PHP)

Roundcube, the webmail client, requires a web server, database, and PHP.

### 2.1. Update System & Install Core Packages

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt install apache2 mariadb-server mariadb-client -y
```

### 2.2. Install PHP and Required Extensions

For Ubuntu 22.04, PHP 8.1 is default. For Ubuntu 20.04, you can use PHP 7.4.

```bash
# Add the Ondřej Surý PPA for the latest PHP versions (optional for 20.04)
sudo apt-get update
sudo apt -y install software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt-get update

# Install PHP and extensions
sudo apt install php libapache2-mod-php php-mysql php-gd php-xml php-mbstring php-intl php-zip php-json php-curl php-bz2 php-gmp php-imap php-pear php-auth-sasl php-net-idna2 php-mail-mime php-net-ldap3 php-net-sieve -y
```

### 2.3. Enable Apache Modules

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 📮 Step 3: Install and Configure Postfix (SMTP Server)

Postfix is the Mail Transfer Agent (MTA) responsible for sending and receiving emails.

### 3.1. Install Postfix and Mail Utilities

```bash
sudo apt-get install postfix mailutils -y
```

During installation, a configuration window will appear:
*   **General type of mail configuration**: Select `Internet Site`.
*   **System mail name**: Enter your domain (e.g., `paulco.xyz`). This is crucial.

### 3.2. Basic Postfix Configuration

Backup the original configuration file.

```bash
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
```

Edit the main configuration file.

```bash
sudo nano /etc/postfix/main.cf
```

Ensure the following settings are present or updated. Replace `mail.paulco.xyz` and `paulco.xyz` with your domain.

```plaintext
# BASIC SETTINGS
myhostname = mail.paulco.xyz
mydomain = paulco.xyz
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4 # Use 'all' if you have IPv6
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
home_mailbox = Maildir/ # Tells Postfix to use Maildir format

# SECURITY & TLS (We'll configure SSL later)
smtpd_use_tls = yes
smtpd_tls_security_level = may
smtpd_tls_cert_file = /etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file = /etc/ssl/private/ssl-cert-snakeoil.key

# Enable SMTP Submission (Port 587)
smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_security_options = noanonymous
smtpd_sasl_local_domain = $myhostname
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination
```

### 3.3. Enable SMTP Submission in `master.cf`

Edit the master configuration file to enable port 587 for email clients.

```bash
sudo nano /etc/postfix/master.cf
```

Find the section for `submission` and uncomment it. It should look like this:

```plaintext
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
```

### 3.4. Restart Postfix

```bash
sudo systemctl restart postfix
sudo systemctl status postfix # Verify it's running
```

---

## 📬 Step 4: Install and Configure Dovecot (IMAP/POP3 Server)

Dovecot delivers emails to users' mailboxes and allows them to retrieve emails via IMAP or POP3.

### 4.1. Install Dovecot

```bash
sudo apt install dovecot-core dovecot-imapd dovecot-pop3d dovecot-lmtpd -y
```

### 4.2. Configure Dovecot for Mail Location

Edit the mail configuration file.

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Set the mail location to match Postfix's `home_mailbox` setting.

```plaintext
mail_location = maildir:~/Maildir
```

Comment out any other `mail_location` lines.

### 4.3. Configure Dovecot Authentication

Edit the authentication configuration file.

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Make the following changes:

```plaintext
disable_plaintext_auth = yes # Force encrypted logins
auth_mechanisms = plain login # Allow both PLAIN and LOGIN mechanisms

# Comment out system user auth
#!include auth-system.conf.ext

# Uncomment password file auth (for virtual users, we'll set this up later)
!include auth-passwdfile.conf.ext
```

### 4.4. Configure Dovecot Master Socket for Postfix

Edit the master configuration file.

```bash
sudo nano /etc/dovecot/conf.d/10-master.conf
```

Find the `service auth` section and configure the socket for Postfix.

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

### 4.5. (Optional) Enable Secure IMAP/POP3

To enable IMAPS (993) and POP3S (995), find the relevant sections in `10-master.conf` and uncomment the `port` and `ssl` lines.

```plaintext
# For IMAPS
inet_listener imaps {
  port = 993
  ssl = yes
}

# For POP3S
inet_listener pop3s {
  port = 995
  ssl = yes
}
```

### 4.6. Restart Dovecot

```bash
sudo systemctl restart dovecot
sudo systemctl status dovecot # Verify it's running
```

---

## 🌐 Step 5: Install and Configure Roundcube Webmail

Roundcube provides a user-friendly web interface for your users to send and receive emails.

### 5.1. Download and Install Roundcube

```bash
cd /tmp
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.7/roundcubemail-1.6.7-complete.tar.gz
tar -xzf roundcubemail-1.6.7-complete.tar.gz
sudo mv roundcubemail-1.6.7 /var/www/html/roundcube
sudo chown -R www-data:www-data /var/www/html/roundcube/
sudo chmod 755 -R /var/www/html/roundcube/
```

### 5.2. Create a MariaDB Database for Roundcube

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

### 5.3. Import Roundcube’s Initial Database Schema

```bash
sudo mysql -u roundcube_user -p roundcube_db < /var/www/html/roundcube/SQL/mysql.initial.sql
```

### 5.4. Create an Apache Virtual Host

Create a new configuration file for Roundcube.

```bash
sudo nano /etc/apache2/sites-available/roundcube.conf
```

Add the following configuration. Replace `mail.paulco.xyz` with your server's FQDN.

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

### 5.5. Enable the Site and Reload Apache

```bash
sudo a2ensite roundcube.conf
sudo systemctl reload apache2
```

### 5.6. Complete Roundcube Setup via Web Installer

1.  Visit `http://mail.paulco.xyz/roundcube/installer` in your web browser.
2.  Go through the steps:
    *   **Check environment**: Ensure all requirements are met (green checks).
    *   **Create config**: Enter your MariaDB details (database name, username, password).
    *   **Test config**: Test the database connection and IMAP/SMTP settings.
        *   **IMAP Settings**:
            *   `IMAP host`: `localhost`
            *   `IMAP port`: `143`
            *   `IMAP security`: `STARTTLS`
        *   **SMTP Settings**:
            *   `SMTP host`: `localhost`
            *   `SMTP port`: `587`
            *   `SMTP security`: `STARTTLS`
            *   `SMTP username`: `%u` (This tells Roundcube to use the logged-in username)
            *   `SMTP password`: `%p` (This tells Roundcube to use the logged-in password)
3.  **Initialize database**: Click the button to initialize the database.
4.  **Finalizing**: Download the `config.inc.php` file and upload it to `/var/www/html/roundcube/config/` on your server. Alternatively, the installer can try to write it directly if permissions allow.
5.  **SECURITY**: **Delete the installer directory**.

```bash
sudo rm -rf /var/www/html/roundcube/installer/
```

---

## 🔐 Step 6: Configure Firewall (UFW)

Allow necessary ports for mail and web services.

```bash
sudo ufw allow "Apache Full" # Allows 80 and 443
sudo ufw allow "Postfix"     # Allows 25
sudo ufw allow "Postfix Submission" # Allows 587
sudo ufw allow "Postfix SMTPS"      # Allows 465
sudo ufw allow 993/tcp              # IMAPS
sudo ufw allow 995/tcp              # POP3S
sudo ufw enable # Enable the firewall if it's not already
sudo ufw status # Verify rules
```

---

## ✉️ Step 7: Configure Email Authentication (SPF, DKIM, DMARC)

This is critical to prevent your emails from being marked as spam.

### 7.1. Set Up SPF Record

**SPF** tells the world which servers are allowed to send email for your domain.

*   Go to your DNS management panel.
*   Create a new **TXT** record.
*   **Name/Host**: `@` (or leave blank, depending on your DNS provider).
*   **Value**: `v=spf1 mx a ip4:YOUR_SERVER_IP ~all`
    *   Replace `YOUR_SERVER_IP` with your server's public IP address.
    *   `~all` means "soft fail" for other servers (recommended for beginners).

### 7.2. Set Up DKIM with OpenDKIM

**DKIM** adds a digital signature to your emails, proving they haven't been tampered with.

#### 7.2.1. Install OpenDKIM

```bash
sudo apt install opendkim opendkim-tools -y
sudo gpasswd -a postfix opendkim # Add postfix user to opendkim group
```

#### 7.2.2. Configure OpenDKIM

Edit the main configuration file.

```bash
sudo nano /etc/opendkim.conf
```

Add or uncomment the following lines. Replace `paulco.xyz` with your domain.

```plaintext
Syslog yes
LogWhy yes

# These are often commented out by default; leave them commented.
#Domain example.com
#KeyFile /etc/dkimkeys/dkim.key
#Selector 2007

# Uncomment and modify these
Canonicalization relaxed/simple
Mode sv
SubDomains no

# Add these for better operation
AutoRestart yes
AutoRestartRate 10/1M
Background yes
DNSTimeout 5
SignatureAlgorithm rsa-sha256

# OpenDKIM user
UserID opendkim

# Signing and Key tables
KeyTable refile:/etc/opendkim/key.table
SigningTable refile:/etc/opendkim/signing.table

# Hosts to ignore for verification, but sign for
ExternalIgnoreList /etc/opendkim/trusted.hosts
InternalHosts /etc/opendkim/trusted.hosts
```

#### 7.2.3. Create Required Directories and Files

```bash
sudo mkdir /etc/opendkim
sudo mkdir /etc/opendkim/keys
sudo chown -R opendkim:opendkim /etc/opendkim
sudo chmod go-rw /etc/opendkim/keys

# Create Signing Table
sudo nano /etc/opendkim/signing.table
```

Add this line (replace `paulco.xyz`):

```plaintext
*@paulco.xyz default._domainkey.paulco.xyz
```

```bash
# Create Key Table
sudo nano /etc/opendkim/key.table
```

Add this line (replace `paulco.xyz`):

```plaintext
default._domainkey.paulco.xyz paulco.xyz:default:/etc/opendkim/keys/paulco.xyz/default.private
```

```bash
# Create Trusted Hosts
sudo nano /etc/opendkim/trusted.hosts
```

Add these lines:

```plaintext
127.0.0.1
localhost
.paulco.xyz
```

#### 7.2.4. Generate DKIM Keys

```bash
sudo mkdir /etc/opendkim/keys/paulco.xyz
sudo opendkim-genkey -b 2048 -d paulco.xyz -D /etc/opendkim/keys/paulco.xyz -s default -v
sudo chown opendkim:opendkim /etc/opendkim/keys/paulco.xyz/default.private
sudo chmod 600 /etc/opendkim/keys/paulco.xyz/default.private
```

#### 7.2.5. Publish the DKIM Public Key in DNS

Display the public key.

```bash
sudo cat /etc/opendkim/keys/paulco.xyz/default.txt
```

You will see output like this:
`default._domainkey IN TXT ( "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC..." )`

*   Go to your DNS management panel.
*   Create a new **TXT** record.
*   **Name/Host**: `default._domainkey` (This is your **selector**).
*   **Value**: Copy **everything between the parentheses `(...)`**, then **remove all double quotes `"` and spaces**. It should be one long, continuous string starting with `v=DKIM1;...`.

#### 7.2.6. Test the DKIM Key

```bash
sudo opendkim-testkey -d paulco.xyz -s default -vvv
```

You should see `Key OK` in the output. If you see `Key not secure`, it's okay; it just means you haven't set up DNSSEC.

#### 7.2.7. Connect OpenDKIM to Postfix

Edit the OpenDKIM defaults file to fix the socket path for Postfix's chroot jail.

```bash
sudo nano /etc/default/opendkim
```

Find the `SOCKET` line and change it to:

```plaintext
SOCKET="local:/var/spool/postfix/opendkim/opendkim.sock"
```

Now, create the socket directory and set permissions.

```bash
sudo mkdir /var/spool/postfix/opendkim
sudo chown opendkim:postfix /var/spool/postfix/opendkim
```

Edit the OpenDKIM config again to set the correct socket.

```bash
sudo nano /etc/opendkim.conf
```

Find the `Socket` line (it might be commented out) and set it to:

```plaintext
Socket local:/var/spool/postfix/opendkim/opendkim.sock
```

Finally, tell Postfix to use the OpenDKIM milter. Edit `/etc/postfix/main.cf`:

```bash
sudo nano /etc/postfix/main.cf
```

Add these lines at the **end** of the file:

```plaintext
# Milter configuration for OpenDKIM
milter_default_action = accept
milter_protocol = 6
smtpd_milters = local:opendkim/opendkim.sock
non_smtpd_milters = $smtpd_milters
```

#### 7.2.8. Restart Services

```bash
sudo systemctl restart opendkim
sudo systemctl restart postfix
```

### 7.3. Set Up DMARC Record

**DMARC** tells receiving servers what to do if an email fails SPF or DKIM checks.

*   Go to your DNS management panel.
*   Create a new **TXT** record.
*   **Name/Host**: `_dmarc`.
*   **Value**: `v=DMARC1; p=none; rua=mailto:postmaster@paulco.xyz; ruf=mailto:postmaster@paulco.xyz; fo=1`
    *   `p=none`: Monitor only, don't reject/quarantine. Change to `p=quarantine` or `p=reject` once you're sure everything is working.
    *   Replace `postmaster@paulco.xyz` with your admin email.

---

## 👥 Step 8: User Management (Central Web Interface)

The setup above uses system users (created with `adduser`). For a central web interface to manage email users, you need to set up **virtual users**. This is more complex but is the standard for production servers.

### 8.1. Why Virtual Users?

*   **Security**: Email users don't have shell access to your server.
*   **Scalability**: Easier to manage hundreds of users.
*   **Central Management**: Can be integrated with a database and managed via a web panel.

### 8.2. Configure Postfix & Dovecot for Virtual Users

This is a significant configuration change. Here’s a high-level overview:

1.  **Database Setup**: Create a new MariaDB database (e.g., `mailserver_db`) with tables for `virtual_domains`, `virtual_users`, and `virtual_aliases`.
2.  **Postfix Configuration**:
    *   Edit `/etc/postfix/main.cf` to use SQL lookups for domains, mailboxes, and aliases.
    *   Set `virtual_mailbox_domains`, `virtual_mailbox_maps`, `virtual_alias_maps` to point to SQL configuration files (e.g., `/etc/postfix/mysql-virtual-mailbox-domains.cf`).
    *   Set `virtual_mailbox_base = /var/mail/vhosts` and `virtual_uid_maps = static:5000`, `virtual_gid_maps = static:5000`.
3.  **Dovecot Configuration**:
    *   Edit `/etc/dovecot/conf.d/10-mail.conf` to set `mail_location = maildir:/var/mail/vhosts/%d/%n`.
    *   Edit `/etc/dovecot/conf.d/10-auth.conf` to ensure `!include auth-passwdfile.conf.ext` is uncommented and `!include auth-system.conf.ext` is commented.
    *   Edit `/etc/dovecot/conf.d/auth-sql.conf.ext` to configure SQL authentication.
    *   Create a `vmail` system user and group (UID/GID 5000) and set ownership of `/var/mail/vhosts`.
4.  **Create SQL Configuration Files**: Create files like `/etc/postfix/mysql-virtual-mailbox-domains.cf` with database connection details and queries.

### 8.3. Install a Web-Based Control Panel

To manage these virtual users via a web interface, you can install a control panel like **PostfixAdmin**.

#### 8.3.1. Install PostfixAdmin

```bash
cd /tmp
wget https://github.com/postfixadmin/postfixadmin/archive/refs/tags/postfixadmin-3.3.13.tar.gz
tar -xzf postfixadmin-3.3.13.tar.gz
sudo mv postfixadmin-postfixadmin-3.3.13 /var/www/html/postfixadmin
sudo chown -R www-data:www-data /var/www/html/postfixadmin
```

#### 8.3.2. Configure PostfixAdmin

1.  Visit `http://mail.paulco.xyz/postfixadmin/setup.php`.
2.  It will guide you through creating a setup password and configuring the database.
3.  After setup, it will provide you with a hashed password for the `config.local.php` file.
4.  Create `/var/www/html/postfixadmin/config.local.php` and add the database and admin credentials.
5.  **SECURITY**: Delete or rename the `setup.php` file.

#### 8.3.3. Integrate with Roundcube (Optional)

You can install the **Roundcube PostfixAdmin plugin** to allow users to manage their own aliases and vacation messages directly from Roundcube.

---

## 🧪 Step 9: Testing Your Mail Server

1.  **Send a Test Email**: Log in to Roundcube (`http://mail.paulco.xyz/roundcube`) with a user account and send an email to a Gmail or Outlook address.
2.  **Check Email Headers**: In Gmail, open the email and click "Show original". Look for `spf=pass` and `dkim=pass`.
3.  **Use Online Tools**:
    *   **Mail Tester**: Go to `https://www.mail-tester.com`, send an email to the provided address, and check your score.
    *   **DKIM Validator**: Use `https://www.dmarcanalyzer.com/dkim/dkim-check/` to verify your DKIM record.
    *   **SPF Validator**: Use `https://www.kitterman.com/spf/validate.html` to validate your SPF record.

---

## 🛡️ Step 10: Advanced Security and Maintenance

*   **Install Fail2ban**: Protects against brute-force attacks on Postfix, Dovecot, and SSH.
    ```bash
    sudo apt install fail2ban -y
    ```
*   **Set Up Automatic Security Updates**:
    ```bash
    sudo apt install unattended-upgrades -y
    sudo dpkg-reconfigure unattended-upgrades # Select 'Yes'
    ```
*   **Monitor Logs**: Regularly check `/var/log/mail.log`, `/var/log/mail.err`, and `/var/log/syslog`.
*   **Backup**: Regularly back up your configuration files (`/etc/postfix`, `/etc/dovecot`, `/etc/opendkim`) and your MariaDB databases.

---

## 📝 Corrections and Enhancements to Your Original Content

*   **PHP Installation**: Added the `ondrej/php` PPA for better PHP version control on Ubuntu 20.04.
*   **Postfix Configuration**: Provided a more complete and secure `main.cf` configuration, including SASL for authentication.
*   **Dovecot Configuration**: Corrected the configuration to properly integrate with Postfix using the `private/auth` socket and set up `Maildir`.
*   **Roundcube Version**: Updated to a more recent, stable version (1.6.7).
*   **Firewall**: Added rules for IMAPS (993) and POP3S (995).
*   **SPF/DKIM/DMARC**: Provided a complete, step-by-step guide for all three, which is essential for email deliverability. Your original guide only partially covered SPF.
*   **User Management**: Added a dedicated section explaining the need for and setup of **virtual users** and a **web-based control panel (PostfixAdmin)**, which was a key missing piece.
*   **Testing**: Added specific steps and tools for thoroughly testing your server.
*   **Structure**: Reorganized the steps into a logical, chronological order for a smoother setup process.
*   **Clarity**: Added more explanatory comments and context throughout the guide.

This guide provides a robust foundation for a self-hosted email server. Remember that maintaining a mail server requires ongoing effort to manage security, reputation, and deliverability.
