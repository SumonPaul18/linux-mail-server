# Configure a Full-Featured Mail Server on Ubuntu (Postfix, Dovecot, Roundcube)

This comprehensive guide will walk you through the initial configuration of a cost-free, full-featured mail server for enterprise use on Ubuntu. We'll leverage Postfix for mail transfer, Dovecot for mail delivery, and Roundcube for webmail management. While there are various ways to set up mail servers on Linux and Windows platforms (e.g., Microsoft Exchange for Windows), this tutorial focuses on a robust Linux-based solution.

## Key Components

Here are the essential software components we'll be using:

1.  **Postfix**: Postfix is a **Mail Transfer Agent (MTA)**. It's the core software responsible for sending and receiving emails, making it indispensable for a complete mail server.
2.  **Dovecot**: Dovecot acts as a **Mail Delivery Agent (MDA)**. It handles the delivery of emails to and from the mail server, allowing users to access their mailboxes using protocols like IMAP and POP3.
3.  **Roundcube**: Roundcube is a **webmail client**. It provides a user-friendly web interface for managing emails on your server, offering a simple and customizable way to access your mailboxes through a web browser.

## Prerequisites

Before you begin, ensure you have the following in place:

1.  **Valid Domain**: A registered domain name (e.g., `paulco.xyz`).
2.  **DNS Server**: Access to a DNS server where you can manage your domain's DNS records (A, MX, TXT).
3.  **Ubuntu 22.04.2 LTS Server**: A server instance with the following recommended specifications:
      * 4 CPU cores
      * 8 GB RAM
      * 50 GB Storage
      * 1 NIC with a public IP address

-----

## Basic Server Configuration

Let's start by performing some basic configurations on your Ubuntu server.

### Check OS Version

```bash
cat /etc/os-release
hostnamectl
```

### Check CPU Information

```bash
lscpu
cat /proc/cpuinfo
nproc
```

### Check RAM

```bash
free -h
```

### Check Disk Space

```bash
df -h
```

### Change Hostname

It's crucial to set a fully qualified domain name (FQDN) for your mail server.

```bash
sudo hostnamectl set-hostname mail.paulco.xyz
```

### Add Server IP and FQDN to Hosts File

Edit your `/etc/hosts` file to map your server's IP address to its FQDN and short hostname. Replace `192.168.3.55` with your server's actual IP address.

```bash
sudo nano /etc/hosts
```

Add the following line:

```
192.168.3.55 mail.paulco.xyz mail
```

### Check FQDN

Verify that your FQDN is correctly set:

```bash
hostname -f
```

### Add a Mail Server User

It's good practice to create a dedicated user for managing the mail server.

```bash
sudo adduser mail
```

### Grant Sudo Privileges

Add the newly created user to the `sudo` group.

```bash
sudo usermod -aG sudo mail
```

### Log in as the Mail User

Switch to the new user to perform further configurations.

```bash
su mail
```

-----

## Installing & Configuring Postfix Mail Server

Now, let's install and perform the initial configuration of Postfix.

### Update and Upgrade Packages

Always start by updating your system's package lists and upgrading existing packages.

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```

### Install Postfix

During the installation, you'll be prompted to configure Postfix. Make sure to select "Internet Site" and set the "System mail name" to your domain name (e.g., `paulco.xyz`).

```bash
sudo apt-get install postfix -y
```

### Check Postfix Version

```bash
postconf mail_version
```

### Check Postfix Service Status

```bash
sudo systemctl status postfix
```

### Reconfigure Postfix (If Needed)

If you made a mistake during the initial setup or need to reconfigure Postfix, you can use:

```bash
sudo dpkg-reconfigure postfix
```

-----

## Advanced Postfix Configuration

Let's fine-tune Postfix for better mail handling.

### Backup Postfix Main Configuration File

It's always a good idea to back up critical configuration files before making changes.

```bash
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
```

### Add Maildir Location

Edit the Postfix main configuration file to specify the Maildir format for user mailboxes. This is a more modern and flexible way to store emails compared to the older mbox format.

```bash
sudo nano /etc/postfix/main.cf
```

Add the following line to the end of the file:

```
home_mailbox = Maildir/
```

### Restart Postfix

Apply the changes by restarting the Postfix service.

```bash
sudo systemctl restart postfix
```

-----

## Testing Postfix Mail Server

Verify that Postfix is working as expected by performing some basic tests.

### Ping SMTP Server

Test connectivity to an external SMTP server (e.g., Gmail).

```bash
ping smtp.gmail.com
```

### Test External SMTP Connectivity (Port 25)

Use `telnet` to check if you can connect to an external SMTP server on port 25. This checks outgoing connectivity.

```bash
telnet gmail-smtp-in.l.google.com 25
```

You should see an output similar to this:

```
Trying 142.251.12.108...
Connected to gmail-smtp-in.l.google.com.
Escape character is '^]'.
220 smtp.gmail.com ESMTP t187-200...
```

Type `quit` and press Enter to exit `telnet`.

### Test External SMTP Connectivity (Port 587)

Repeat the `telnet` test for port 587, which is commonly used for submission with authentication.

```bash
telnet smtp.gmail.com 587
```

You should see a similar output. Type `quit` and press Enter to exit `telnet`.

### Install Mailutils

`mailutils` provides a simple command-line utility to send emails.

```bash
sudo apt-get install mailutils -y
```

### Send a Test Email to Gmail

Send a test email to an external Gmail address. Replace `sumonpaul267@gmail.com` with your desired recipient.

```bash
echo "Hey, I am dev stack. I want to connect with you sumon" | mail -s "Connected Mail" sumonpaul267@gmail.com
```

Press `Ctrl+D` to send the email after typing the body.

### Verify Postfix with Local Telnet

Test local connectivity to your Postfix server on port 25. Replace `192.168.102.250` with your server's actual IP address or `localhost`.

```bash
telnet 192.168.102.250 25
```

### Monitor Mail Log

To see real-time mail activity and troubleshoot any issues, monitor the mail log file.

```bash
tail -f /var/log/mail.log
```

-----

## Installing Dovecot for IMAP and POP

Dovecot will handle the delivery of emails to user mailboxes and allow clients to retrieve them using IMAP and POP3.

### Install Dovecot IMAP and POP3 Daemons

```bash
sudo apt install dovecot-imapd dovecot-pop3d -y
```

### Restart and Check Dovecot Status

```bash
sudo systemctl restart dovecot
sudo systemctl status dovecot
```

### Backup Dovecot Configuration File

```bash
sudo cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.bak
```

### Configure Dovecot Protocols

Edit the main Dovecot configuration file to enable IMAP and POP3 protocols.

```bash
sudo nano /etc/dovecot/dovecot.conf
```

Uncomment or add the following line:

```
protocols = pop3 pop3s imap imaps
```

### Configure Dovecot Listen Address

Uncomment the `listen` line to allow Dovecot to listen on all network interfaces.

```bash
listen = *
```

### Configure Authentication Settings

Edit the authentication configuration file. Depending on your security requirements, you can enable or disable plain text authentication. For better security, it's generally recommended to disable it in production environments once TLS/SSL is configured.

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Uncomment and set `disable_plaintext_auth` as per your preference. For initial testing, `no` might be easier, but for production, `yes` is recommended with TLS.

```
disable_plaintext_auth = no
```

### Configure Mail Location

Specify the Maildir location for user mailboxes in Dovecot's mail configuration.

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Uncomment the following line:

```
mail_location = maildir:~/Maildir
```

And comment out the mbox line if it's present:

```
#mail_location = mbox:~/mail:INBOX=/var/mail/%u
```

### Restart Dovecot and Postfix

Apply the Dovecot changes and restart both Dovecot and Postfix to ensure they integrate correctly.

```bash
sudo systemctl restart dovecot
sudo systemctl restart postfix
```

-----

## Installing Roundcube for Webmail

Roundcube will provide a web-based interface for your users to access their emails.

### Install Apache, MariaDB, and PHP

Roundcube requires a web server (Apache), a database (MariaDB), and PHP to function.

```bash
sudo apt-get install apache2 mariadb-server php libapache2-mod-php php-mysql -y
```

### Enable Apache Rewrite Module

The Apache rewrite module is necessary for Roundcube's URL rewriting.

```bash
sudo a2enmod rewrite
```

### Reload Apache

```bash
sudo systemctl reload apache2
```

### Install Essential PHP Packages for Roundcube

You'll need several PHP extensions for Roundcube to function correctly. Choose **one** of the following options. The first option is generally more comprehensive.

**Option 1 (Recommended):**

```bash
sudo apt install openssl composer php-net-smtp php-mysql php-gd php-xml php-mbstring php-intl php-zip php-json php-pear php-bz2 php-gmp php-imap php-imagick php-auth-sasl php-net-idna2 php-mail-mime php-net-ldap3 php-net-sieve -y
```

**Option 2 (Specific PHP 7.4 - use if you have PHP 7.4 installed):**

```bash
sudo apt install php7.4 libapache2-mod-php7.4 php7.4-mysql php-net-ldap2 php-net-ldap3 php-imagick php7.4-common php7.4-gd php7.4-imap php7.4-json php7.4-curl php7.4-zip php7.4-xml php7.4-mbstring php7.4-bz2 php7.4-intl php7.4-gmp php-net-smtp php-mail-mime php-net-idna2 -y
```

**Troubleshooting PHP Installation (If you encounter errors):**

If you face issues installing PHP packages, you might need to add the Ondrej PHP repository for newer versions.

```bash
sudo apt -y install software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
sudo apt-get update -y
```

After installing PHP packages, restart Apache:

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

### Download and Extract Roundcube

Download the latest stable version of Roundcube from their GitHub releases page. Always check for the most recent stable release. (As of this guide, `1.6.1` is used, but verify on the Roundcube website).

```bash
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.1/roundcubemail-1.6.1-complete.tar.gz
tar -xvf roundcubemail-1.6.1-complete.tar.gz
```

### Move and Set Permissions for Roundcube

Move the extracted Roundcube directory to your web server's document root and set appropriate permissions.

```bash
sudo mv roundcubemail-1.6.1 /var/www/html/roundcubemail
sudo chown -R www-data:www-data /var/www/html/roundcubemail/
sudo chmod 755 -R /var/www/html/roundcubemail/
```

### Create MariaDB Database and User for Roundcube

Access the MariaDB prompt as the root user and create a database and user for Roundcube. Remember to replace `'ubuntu'` with a strong, secure password.

```bash
sudo mysql -u root
```

```sql
CREATE DATABASE roundcube DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER roundcubeuser@localhost IDENTIFIED BY 'ubuntu';
GRANT ALL PRIVILEGES ON roundcube.* TO roundcubeuser@localhost;
FLUSH PRIVILEGES;
QUIT;
```

### Import Roundcube Initial SQL Schema

Import the necessary database schema into the `roundcube` database.

```bash
sudo mysql roundcube < /var/www/html/roundcubemail/SQL/mysql.initial.sql
```

### Create Apache Virtual Host for Roundcube

Create a new Apache virtual host configuration file for Roundcube.

```bash
sudo nano /etc/apache2/sites-available/roundcube.conf
```

Add the following configuration. Replace `mail.paulco.xyz` with your mail server's FQDN.

```apache
<VirtualHost *:80>
  ServerName mail.paulco.xyz
  DocumentRoot /var/www/html/roundcubemail/
  ErrorLog ${APACHE_LOG_DIR}/roundcube_error.log
  CustomLog ${APACHE_LOG_DIR}/roundcube_access.log combined

  <Directory />
    Options FollowSymLinks
    AllowOverride All
  </Directory>

  <Directory /var/www/html/roundcubemail/>
    Options FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
  </Directory>

</VirtualHost>
```

### Enable Roundcube Virtual Host and Reload Apache

```bash
sudo a2ensite roundcube.conf
sudo systemctl reload apache2
```

### Restart Dovecot and Postfix (Again)

Ensure all services are aware of the new configuration.

```bash
sudo systemctl restart dovecot
sudo systemctl restart postfix
```

### Complete Roundcube Installation via Web Installer

Open your web browser and navigate to the Roundcube installer URL:

`http://mail.paulco.xyz/roundcubemail/installer/`

Follow the on-screen instructions:

  * **Make sure to correctly configure the database name (`roundcube`), database username (`roundcubeuser`), and database password (the one you set, e.g., `ubuntu`).**
  * In the **IMAP** settings, use `localhost` and the default port `143` (or `993` for SSL/TLS if configured).
  * In the **SMTP** settings, use `localhost` and port `25` (or `587` for submission with authentication if configured). **Do not set a username/password in the SMTP settings for now, as Postfix is configured for local submission without authentication for internal users.**

### Remove Installer Directory

After successful installation, it is crucial to remove the installer directory for security reasons.

```bash
sudo rm -r /var/www/html/roundcubemail/installer/
```

Before you can test sending and receiving emails through Roundcube, make sure to create mail users in Ubuntu using the `adduser` command for each desired email address (e.g., `adduser user1`, `adduser user2`).

-----

## Adding SPF, DKIM, DMARC Records to DNS

Implementing SPF, DKIM, and DMARC is crucial for email deliverability and preventing spam. These records help authenticate your outgoing emails, reducing the likelihood of them being marked as spam.

**Reference**: [https://www.linuxbabe.com/mail-server/setting-up-dkim-and-spf](https://www.linuxbabe.com/mail-server/setting-up-dkim-and-spf)

-----

## How to Set up SPF Record

**Sender Policy Framework (SPF)** is an email authentication method designed to detect forged sender addresses in email.

### Check Existing TXT Records (Optional)

You can check existing TXT records for your domain using `dig` or `nslookup`.

```bash
dig txt paulco.xyz
```

Using `nslookup`:

```bash
nslookup
> set type=txt
> paulco.xyz
```

### Add SPF Record in DNS Server

You need to add a TXT record to your domain's DNS settings. This can be done either in your local DNS server (if you're managing one) or through your domain registrar's DNS management panel.

**Example for `nano /etc/bind/forward.zone` (if using Bind9):**

```
@  IN  TXT "v=spf1 a mx ip4:202.4.111.69 ~all"
```

**Example for Domain Panel:**

Go to your Domain Dashboard (e.g., Namecheap, GoDaddy, Cloudflare) -\> DNS Management -\> Add a new TXT record:

  * **Type**: `TXT`
  * **Host/Name**: `@` (or leave blank for the root domain)
  * **Value/Text**: `"v=spf1 a mx ip4:202.4.111.69 ~all"`
  * **TTL**: `Auto` or `3600` (seconds)

**Explanation of the SPF record:**

  * `v=spf1`: Specifies the SPF version.
  * `a`: Allows the A record of the domain to send email.
  * `mx`: Allows the MX records of the domain to send email.
  * `ip4:202.4.111.69`: Explicitly allows the specified IPv4 address to send email. Replace `202.4.111.69` with your mail server's public IP address.
  * `~all`: A "softfail" mechanism, meaning emails from other sources *might* be legitimate, but should be treated with suspicion. For stricter policies, you can use `-all` (hardfail), but be cautious with `~all` initially.

### Verify SPF Record

After adding the record, give it some time to propagate (DNS propagation can take a few minutes to several hours). Then, re-check using `dig` or `nslookup`.

-----

### Optional: Configuring SPF Policy Agent (Postfix-policyd-spf-python)

This policy agent for Postfix helps to reject incoming emails that fail SPF checks.

```bash
sudo apt install postfix-policyd-spf-python -y
```

Edit the Postfix master configuration file:

```bash
sudo nano /etc/postfix/master.cf
```

Add the following lines at the end of the file:

```
policyd-spf  unix  -       n       n       -       0       spawn
  user=policyd-spf argv=/usr/bin/policyd-spf
```

Save and close the file. Next, edit the Postfix main configuration file:

```bash
sudo nano /etc/postfix/main.cf
```

Append the following lines at the end of the file:

```
policyd-spf_time_limit = 3600
smtpd_recipient_restrictions =
  permit_mynetworks,
  permit_sasl_authenticated,
  reject_unauth_destination,
  check_policy_service unix:private/policyd-spf
```

Save and close the file. Then restart Postfix:

```bash
sudo systemctl restart postfix
```

After this, when you receive an email from a domain with an SPF record, you can check the raw email header for SPF check results, e.g.:

`Received-SPF: Pass (sender SPF authorized).`

-----

## How to Set up DMARC Record

**Domain-based Message Authentication, Reporting, and Conformance (DMARC)** is an email authentication protocol that builds on SPF and DKIM. It allows domain owners to specify how their emails should be handled if they fail SPF or DKIM checks, and to receive reports on email authentication failures.

### Add DMARC Record in Domain Dashboard

Go to your Domain Dashboard (e.g., Namecheap, GoDaddy, Cloudflare) -\> DNS Management -\> Add a new TXT record:

  * **Type**: `TXT`
  * **Host/Name**: `_dmarc`
  * **Value/Text**: `v=DMARC1; p=quarantine; aspf=r; sp=none; rua=mailto:your_email@paulco.xyz; ruf=mailto:your_email@paulco.xyz`
      * **Explanation:**
          * `v=DMARC1`: Specifies the DMARC version.
          * `p=quarantine`: Policy for emails that fail DMARC. `quarantine` means recipients should treat them as suspicious (e.g., move to spam). Other options are `none` (monitor only, no action) and `reject` (reject completely). Start with `none` or `quarantine` and move to `reject` once you are confident.
          * `aspf=r`: Alignment mode for SPF. `r` means relaxed alignment.
          * `sp=none`: Policy for subdomains. `none` means no specific policy for subdomains, they inherit the main domain's policy.
          * `rua=mailto:your_email@paulco.xyz`: Email address to send aggregate DMARC reports to. **Replace `your_email@paulco.xyz` with an actual email address where you want to receive reports.**
          * `ruf=mailto:your_email@paulco.xyz`: Email address to send forensic (failure) DMARC reports to. **Replace `your_email@paulco.xyz` with an actual email address.**

-----

## How to Set up DKIM Record

**DomainKeys Identified Mail (DKIM)** is an email authentication method designed to detect forged sender addresses. DKIM allows the recipient to check if an email was actually sent from the stated domain.

### Install OpenDKIM and OpenDKIM Tools

```bash
sudo apt-get install opendkim opendkim-tools -y
```

### Start and Enable OpenDKIM Service

```bash
sudo systemctl start opendkim
sudo systemctl enable opendkim
```

### Create OpenDKIM Directory

```bash
mkdir /etc/opendkim
```

### Generate DKIM Keys

Generate the private and public DKIM keys for your domain. Replace `paulco.xyz` with your domain.

```bash
opendkim-genkey -D /etc/opendkim/ --domain paulco.xyz --selector mail
```

This command will create two files: `mail.private` (your private key) and `mail.txt` (your public key, to be added to DNS).

### View DKIM Public Key

Display the contents of your public key file. You'll need this information for your DNS record.

```bash
cat /etc/opendkim/mail.txt
```

The output will look something like this:

```
mail._domainkey IN TXT ( "v=DKIM1; h=sha256; k=rsa; "
          "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAqm42/tBAcceewB+FTl5SWVUT0Xa80QZfBBjP4xP16xRJo7YsOoaB9DZlVehNbZxPBJLhDiJdEyV9nDODqABroInlLM0pT2pRGHcXoiamU5tSzWWMWfd8d4kOZZgSeDyJk89+XNHMZeKRX45fJBdxzWVmaFRljxnEz90496aYScyr8WFGSUHCtZD1n2Rq/8O6u8Q8HJKoHfV2fJ"
          "/ZQxN3Z2uUGWBjvB1fjaszHr/iZpsbMRTQM9gAxLI1xmtgs1k4jQua+deZWCLzylLeOW05/itDz87psLBOAZsqH2yh+x6FY1Mf2jGaROV0nSBY0Nf2JyY05FOVB0JgvOEKL569jwIDAQAB" )  ; ----- DKIM key mail for paulco.xyz
```

**Important**: When adding this to your DNS, remove the quotes, parentheses, and newlines so it becomes a single continuous string.

**Example of how it should look for DNS entry (single line, no quotes, no extra spaces):**

```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAqm42/tBAcceewB+FTl5SWVUT0Xa80QZfBBjP4xP16xRJo7YsOoaB9DZlVehNbZxPBJLhDiJdEyV9nDODqABroInlLM0pT2pRGHcXoiamU5tSzWWMWfd8d4kOZZgSeDyJk89+XNHMZeKRX45fJBdxzWVmaFRljxnEz90496aYScyr8WFGSUHCtZD1n2Rq/8O6u8Q8HJKoHfV2fJ/ZQxN3Z2uUGWBjvB1fjaszHr/iZpsbMRTQM9gAxLI1xmtgs1k4jQua+deZWCLzylLeOW05/itDz87psLBOAZsqH2yh+x6FY1Mf2jGaROV0nSBY0Nf2JyY05FOVB0JgvOEKL569jwIDAQAB
```

### Add DKIM Record to DNS

Go to your Domain Dashboard -\> DNS Management -\> Add a new TXT record:

  * **Type**: `TXT`
  * **Host/Name**: `mail._domainkey` (the selector you used, followed by `._domainkey`)
  * **Value/Text**: Paste the single-line public key string you formatted above.
  * **TTL**: `Auto` or `3600`

### Set Ownership for OpenDKIM Directory

```bash
sudo chown -R opendkim:opendkim /etc/opendkim
```

### Configure OpenDKIM

Edit the OpenDKIM main configuration file.

```bash
sudo nano /etc/opendkim.conf
```

Ensure the following lines are uncommented or added/modified as shown:

```
AutoRestart Yes
AutoRestartRate 10/1M
Umask 002
Syslog yes
SyslogSuccess Yes
LogWhy Yes
Mode sv
Canonicalization relaxed/simple
UserID opendkim:opendkim
Socket inet:8891@localhost
PidFile /var/run/opendkim/opendkim.pid
ExternalIgnoreList refile:/etc/opendkim/TrustedHosts
InternalHosts refile:/etc/opendkim/TrustedHosts
KeyTable refile:/etc/opendkim/KeyTable
SigningTable refile:/etc/opendkim/SigningTable
SignatureAlgorithm rsa-sha256
```

### Add Your Domain to Trusted Hosts

Create or edit the `TrustedHosts` file for OpenDKIM.

```bash
sudo nano /etc/opendkim/TrustedHosts
```

Add your domain and `localhost` (or any internal networks that will send email) to this file:

```
127.0.0.1
localhost
.paulco.xyz
```

### Define Key Table

Create or edit the `KeyTable` file, which maps selectors to private keys.

```bash
sudo nano /etc/opendkim/KeyTable
```

Add the following line:

```
mail._domainkey.paulco.xyz paulco.xyz:mail:/etc/opendkim/mail.private
```

### Define Signing Table

Create or edit the `SigningTable` file, which specifies which domains should be signed with which key.

```bash
sudo nano /etc/opendkim/SigningTable
```

Add the following lines:

```
*@paulco.xyz mail._domainkey.paulco.xyz
*@*.paulco.xyz mail._domainkey.paulco.xyz
```

### Restart OpenDKIM and Postfix

Apply the OpenDKIM changes and restart both services.

```bash
sudo systemctl restart opendkim
sudo systemctl restart postfix
```

-----

### Check DKIM Record and Mail Performance

### Verify DKIM in Terminal

You can test your DKIM record using `opendkim-testkey`:

```bash
sudo opendkim-testkey -d paulco.xyz -s mail -vvv
```

This should output `Key OK` if your DNS record and configuration are correct.

### Check DKIM Using Online Tools

Online tools can help verify your DKIM record.

  * Visit: [https://dmarcian.com/dkim-inspector/](https://dmarcian.com/dkim-inspector/)
  * Enter your domain (`paulco.xyz`) and selector (`mail`).

### Test Mail Performance

To thoroughly test your mail server's SPF, DKIM, and DMARC configuration, send a test email to a service like Mail-Tester.

  * Visit: [https://www.mail-tester.com/](https://www.mail-tester.com/)
  * Copy the unique email address provided by Mail-Tester.
  * Send a test email from your newly configured mail server to this address (e.g., using Roundcube or the `mail` command).

<!-- end list -->

```bash
echo "Hey, I am dev stack. I want to connect with you sumon" | mail -s "Connected Mail" sumonpaul267@gmail.com
```

  * Go back to Mail-Tester and check your score. Aim for a high score (usually 9/10 or 10/10) to ensure good deliverability.

-----

## Conclusion

You have successfully installed and configured Postfix, Dovecot, and Roundcube, ensuring your mail server is functional. You've also implemented essential DNS records like SPF, DKIM, and DMARC to improve email deliverability and combat spam.

With this setup, you now have a robust, self-hosted email solution. Remember to regularly update your server and software, and monitor your mail logs for any issues.

What would you like to explore next? Perhaps securing your mail server with SSL/TLS, adding virtual users and domains, or further optimizing performance?
