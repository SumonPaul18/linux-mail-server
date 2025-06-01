# 📮 একটি সম্পূর্ণ মেইল সার্ভার কনফিগারেশন in Linux using (Postfix, Dovecot, Roundcube)

এই বিস্তৃত নির্দেশিকা আপনাকে উবুন্টুতে একটি এন্টারপ্রাইজ-গ্রেডের, সম্পূর্ণ ফিচারযুক্ত এবং বিনামূল্যে মেইল সার্ভার সেটআপ করার প্রাথমিক কনফিগারেশন ধাপে ধাপে বুঝিয়ে দেবে। আমরা মেইল আদান-প্রদানের জন্য **Postfix**, মেইল ডেলিভারির জন্য **Dovecot** এবং ওয়েবমেইল ব্যবস্থাপনার জন্য **Roundcube** ব্যবহার করব। উইন্ডোজে মাইক্রোসফট এক্সচেঞ্জের মতো মেইল সার্ভার সেটআপের বিভিন্ন উপায় থাকলেও, এই টিউটোরিয়ালটি লিনাক্স-ভিত্তিক একটি শক্তিশালী সমাধান নিয়ে আলোচনা করবে।

## মূল উপাদানসমূহ

আমরা যে প্রধান সফটওয়্যার উপাদানগুলো ব্যবহার করব তা নিচে দেওয়া হলো:

1.  **Postfix**: Postfix হলো একটি **মেইল ট্রান্সফার এজেন্ট (MTA)**। এটি ইমেল পাঠানো এবং গ্রহণ করার মূল সফটওয়্যার, যা একটি সম্পূর্ণ মেইল সার্ভারের জন্য অপরিহার্য।
2.  **Dovecot**: Dovecot একটি **মেইল ডেলিভারি এজেন্ট (MDA)** হিসেবে কাজ করে। এটি মেইল সার্ভারে এবং সার্ভার থেকে ইমেল ডেলিভারি পরিচালনা করে, যা ব্যবহারকারীদের IMAP এবং POP3 প্রোটোকল ব্যবহার করে তাদের মেইলবক্সে প্রবেশ করতে দেয়।
3.  **Roundcube**: Roundcube একটি **ওয়েবমেইল ক্লায়েন্ট**। এটি আপনার সার্ভারে ইমেল পরিচালনার জন্য একটি ব্যবহারকারী-বান্ধব ওয়েব ইন্টারফেস সরবরাহ করে, যা ওয়েব ব্রাউজারের মাধ্যমে আপনার মেইলবক্স অ্যাক্সেস করার একটি সহজ এবং কাস্টমাইজযোগ্য উপায়।

## পূর্বশর্ত

শুরু করার আগে, নিশ্চিত করুন যে আপনার নিচের বিষয়গুলো প্রস্তুত আছে:

1.  **বৈধ ডোমেইন**: একটি নিবন্ধিত ডোমেইন নাম (যেমন: `paulco.xyz`)।
2.  **DNS সার্ভার**: একটি DNS সার্ভারে অ্যাক্সেস, যেখানে আপনি আপনার ডোমেইনের DNS রেকর্ডগুলো (A, MX, TXT) পরিচালনা করতে পারবেন।
3.  **উবুন্টু 22.04.2 LTS সার্ভার**: একটি সার্ভার ইনস্ট্যান্স, যার প্রস্তাবিত স্পেসিফিকেশনগুলো হলো:
      * 4 CPU কোর
      * 8 GB RAM
      * 50 GB স্টোরেজ
      * পাবলিক আইপি সহ 1 NIC

-----

## বেসিক সার্ভার কনফিগারেশন

চলুন, আপনার উবুন্টু সার্ভারে কিছু প্রাথমিক কনফিগারেশন দিয়ে শুরু করি।

### অপারেটিং সিস্টেমের সংস্করণ পরীক্ষা করুন

```bash
cat /etc/os-release
hostnamectl
```

### সিপিইউ তথ্য পরীক্ষা করুন

```bash
lscpu
cat /proc/cpuinfo
nproc
```

### RAM পরীক্ষা করুন

```bash
free -h
```

### ডিস্কের স্থান পরীক্ষা করুন

```bash
df -h
```

### হোস্টনেম পরিবর্তন করুন

আপনার মেইল সার্ভারের জন্য একটি **সম্পূর্ণ কোয়ালিফাইড ডোমেইন নাম (FQDN)** সেট করা অত্যন্ত গুরুত্বপূর্ণ।

```bash
sudo hostnamectl set-hostname mail.paulco.xyz
```

### সার্ভার আইপি এবং FQDN হোস্ট ফাইলে যোগ করুন

আপনার `/etc/hosts` ফাইলটি এডিট করুন এবং আপনার সার্ভারের আইপি অ্যাড্রেসকে তার FQDN এবং শর্ট হোস্টনেমের সাথে ম্যাপ করুন। `192.168.3.55` এর জায়গায় আপনার সার্ভারের আসল আইপি অ্যাড্রেসটি বসান।

```bash
sudo nano /etc/hosts
```

নিচের লাইনটি যোগ করুন:

```
192.168.3.55 mail.paulco.xyz mail
```

### FQDN পরীক্ষা করুন

আপনার FQDN সঠিকভাবে সেট করা হয়েছে কিনা তা যাচাই করুন:

```bash
hostname -f
```

### মেইল সার্ভারের জন্য ব্যবহারকারী যোগ করুন

মেইল সার্ভার পরিচালনার জন্য একটি ডেডিকেটেড ব্যবহারকারী তৈরি করা একটি ভালো অনুশীলন।

```bash
sudo adduser mail
```

### সুডো প্রিভিলেজ দিন

নতুন তৈরি করা ব্যবহারকারীকে `sudo` গ্রুপে যোগ করুন।

```bash
sudo usermod -aG sudo mail
```

### মেইল ব্যবহারকারী হিসেবে লগইন করুন

পরবর্তী কনফিগারেশনগুলো সম্পাদন করতে নতুন ব্যবহারকারীর কাছে স্যুইচ করুন।

```bash
su mail
```

-----

## Postfix মেইল সার্ভার ইনস্টল এবং কনফিগার করুন

এখন, আমরা Postfix ইনস্টল করব এবং এর প্রাথমিক কনফিগারেশন সম্পন্ন করব।

### প্যাকেজ আপডেট এবং আপগ্রেড করুন

সর্বদা আপনার সিস্টেমের প্যাকেজ তালিকা আপডেট করে এবং বিদ্যমান প্যাকেজগুলি আপগ্রেড করে শুরু করুন।

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```

### Postfix ইনস্টল করুন

ইনস্টলেশনের সময়, আপনাকে Postfix কনফিগার করতে বলা হবে। "Internet Site" নির্বাচন করুন এবং "System mail name" আপনার ডোমেইন নামের (যেমন, `paulco.xyz`) মতো সেট করতে ভুলবেন না।

```bash
sudo apt-get install postfix -y
```

### Postfix সংস্করণ পরীক্ষা করুন

```bash
postconf mail_version
```

### Postfix সার্ভিসের অবস্থা পরীক্ষা করুন

```bash
sudo systemctl status postfix
```

### Postfix পুনরায় কনফিগার করুন (যদি প্রয়োজন হয়)

যদি আপনি প্রাথমিক সেটআপের সময় কোনো ভুল করে থাকেন বা Postfix পুনরায় কনফিগার করার প্রয়োজন হয়, তাহলে আপনি এটি ব্যবহার করতে পারেন:

```bash
sudo dpkg-reconfigure postfix
```

-----

## Postfix এর উন্নত কনফিগারেশন

মেইল ব্যবস্থাপনার উন্নতির জন্য Postfix কে ফাইন-টিউন করা যাক।

### Postfix এর মূল কনফিগারেশন ফাইলের ব্যাকআপ নিন

পরিবর্তন করার আগে গুরুত্বপূর্ণ কনফিগারেশন ফাইলগুলির ব্যাকআপ নেওয়া সবসময় একটি ভালো অভ্যাস।

```bash
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
```

### Maildir এর অবস্থান যোগ করুন

ব্যবহারকারীর মেইলবক্সের জন্য Maildir ফরম্যাট নির্দিষ্ট করতে Postfix এর মূল কনফিগারেশন ফাইলটি এডিট করুন। এটি পুরোনো mbox ফরম্যাটের চেয়ে ইমেল সংরক্ষণের একটি আধুনিক এবং নমনীয় উপায়।

```bash
sudo nano /etc/postfix/main.cf
```

ফাইলের শেষে নিচের লাইনটি যোগ করুন:

```
home_mailbox = Maildir/
```

### Postfix রিস্টার্ট করুন

পরিবর্তনগুলো কার্যকর করতে Postfix সার্ভিসটি রিস্টার্ট করুন।

```bash
sudo systemctl restart postfix
```

-----

## Postfix মেইল সার্ভার পরীক্ষা করা

কিছু মৌলিক পরীক্ষা করে Postfix প্রত্যাশিতভাবে কাজ করছে কিনা তা যাচাই করুন।

### SMTP সার্ভার পিং করুন

একটি বাহ্যিক SMTP সার্ভারের (যেমন, Gmail) সাথে সংযোগ পরীক্ষা করুন।

```bash
ping smtp.gmail.com
```

### বাহ্যিক SMTP সংযোগ পরীক্ষা করুন (পোর্ট 25)

পোর্ট 25-এ একটি বাহ্যিক SMTP সার্ভারের সাথে সংযোগ করতে `telnet` ব্যবহার করুন। এটি আউটগোয়িং সংযোগ পরীক্ষা করে।

```bash
telnet gmail-smtp-in.l.google.com 25
```

আপনার নিচের মতো একটি আউটপুট দেখা উচিত:

```
Trying 142.251.12.108...
Connected to gmail-smtp-in.l.google.com.
Escape character is '^]'.
220 smtp.gmail.com ESMTP t187-200...
```

`quit` টাইপ করে Enter চাপুন `telnet` থেকে বের হওয়ার জন্য।

### বাহ্যিক SMTP সংযোগ পরীক্ষা করুন (পোর্ট 587)

পোর্ট 587 এর জন্য `telnet` পরীক্ষাটি পুনরাবৃত্তি করুন, যা সাধারণত প্রমাণীকরণের সাথে জমা দেওয়ার জন্য ব্যবহৃত হয়।

```bash
telnet smtp.gmail.com 587
```

আপনার একটি অনুরূপ আউটপুট দেখা উচিত। `quit` টাইপ করে Enter চাপুন `telnet` থেকে বের হওয়ার জন্য।

### Mailutils ইনস্টল করুন

`mailutils` ইমেল পাঠানোর জন্য একটি সহজ কমান্ড-লাইন ইউটিলিটি সরবরাহ করে।

```bash
sudo apt-get install mailutils -y
```

### Gmail এ একটি টেস্ট ইমেল পাঠান

একটি বাহ্যিক Gmail ঠিকানায় একটি টেস্ট ইমেল পাঠান। `sumonpaul267@gmail.com` এর জায়গায় আপনার কাঙ্ক্ষিত প্রাপকের ঠিকানা বসান।

```bash
echo "Hey, I am dev stack. I want to connect with you sumon" | mail -s "Connected Mail" sumonpaul267@gmail.com
```

বডি টাইপ করার পর ইমেল পাঠানোর জন্য `Ctrl+D` চাপুন।

### স্থানীয় Telnet দিয়ে Postfix যাচাই করুন

পোর্ট 25-এ আপনার Postfix সার্ভারে স্থানীয় সংযোগ পরীক্ষা করুন। `192.168.102.250` এর জায়গায় আপনার সার্ভারের আসল আইপি অ্যাড্রেস বা `localhost` ব্যবহার করুন।

```bash
telnet 192.168.102.250 25
```

### মেইল লগ মনিটর করুন

রিয়েল-টাইম মেইল ​​কার্যকলাপ দেখতে এবং যেকোনো সমস্যা সমাধান করতে, মেইল ​​ল ফাইলটি মনিটর করুন।

```bash
tail -f /var/log/mail.log
```

-----

## IMAP এবং POP এর জন্য Dovecot ইনস্টল করা

Dovecot ব্যবহারকারীর মেইলবক্সে ইমেল সরবরাহ করবে এবং ক্লায়েন্টদের IMAP এবং POP3 ব্যবহার করে সেগুলিকে পুনরুদ্ধার করার অনুমতি দেবে।

### Dovecot IMAP এবং POP3 ডেমন ইনস্টল করুন

```bash
sudo apt install dovecot-imapd dovecot-pop3d -y
```

### Dovecot রিস্টার্ট এবং স্ট্যাটাস পরীক্ষা করুন

```bash
sudo systemctl restart dovecot
sudo systemctl status dovecot
```

### Dovecot কনফিগারেশন ফাইলের ব্যাকআপ নিন

```bash
sudo cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.bak
```

### Dovecot প্রোটোকল কনফিগার করুন

IMAP এবং POP3 প্রোটোকল সক্রিয় করতে Dovecot এর প্রধান কনফিগারেশন ফাইলটি এডিট করুন।

```bash
sudo nano /etc/dovecot/dovecot.conf
```

নিচের লাইনটি আনকমেন্ট করুন বা যোগ করুন:

```
protocols = pop3 pop3s imap imaps
```

### Dovecot Listen Address কনফিগার করুন

Dovecot কে সকল নেটওয়ার্ক ইন্টারফেসে শোনার অনুমতি দিতে `listen` লাইনটি আনকমেন্ট করুন।

```bash
listen = *
```

### প্রমাণীকরণ সেটিংস কনফিগার করুন

প্রমাণীকরণ কনফিগারেশন ফাইলটি এডিট করুন। আপনার নিরাপত্তা প্রয়োজনীয়তা অনুযায়ী, আপনি প্লেইন টেক্সট প্রমাণীকরণ সক্ষম বা নিষ্ক্রিয় করতে পারেন। উন্নত নিরাপত্তার জন্য, TLS/SSL কনফিগার করা হলে প্রোডাকশন পরিবেশে এটি নিষ্ক্রিয় করা সাধারণত সুপারিশ করা হয়।

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

আপনার পছন্দ অনুযায়ী `disable_plaintext_auth` আনকমেন্ট করুন এবং সেট করুন। প্রাথমিক পরীক্ষার জন্য, `no` সহজ হতে পারে, তবে প্রোডাকশনের জন্য, TLS সহ `yes` সুপারিশ করা হয়।

```
disable_plaintext_auth = no
```

### মেইল লোকেশন কনফিগার করুন

Dovecot এর মেইল কনফিগারেশনে ব্যবহারকারীর মেইলবক্সের জন্য Maildir লোকেশন নির্দিষ্ট করুন।

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

নিচের লাইনটি আনকমেন্ট করুন:

```
mail_location = maildir:~/Maildir
```

এবং যদি mbox লাইনটি উপস্থিত থাকে তবে এটিকে কমেন্ট আউট করুন:

```
#mail_location = mbox:~/mail:INBOX=/var/mail/%u
```

### Dovecot এবং Postfix রিস্টার্ট করুন

Dovecot এর পরিবর্তনগুলো প্রয়োগ করুন এবং তারা সঠিকভাবে ইন্টিগ্রেট হয়েছে তা নিশ্চিত করতে Dovecot এবং Postfix উভয়ই রিস্টার্ট করুন।

```bash
sudo systemctl restart dovecot
sudo systemctl restart postfix
```

-----

## উবুন্টুতে ওয়েবমেইলের জন্য Roundcube ইনস্টল করা

Roundcube আপনার ব্যবহারকারীদের তাদের ইমেল অ্যাক্সেস করার জন্য একটি ওয়েব-ভিত্তিক ইন্টারফেস সরবরাহ করবে।

### Apache, MariaDB, এবং PHP ইনস্টল করুন

Roundcube কাজ করার জন্য একটি ওয়েব সার্ভার (Apache), একটি ডাটাবেস (MariaDB) এবং PHP প্রয়োজন।

```bash
sudo apt-get install apache2 mariadb-server php libapache2-mod-php php-mysql -y
```

### Apache Rewrite মডিউল সক্রিয় করুন

Roundcube এর URL রিরাইটিংয়ের জন্য Apache rewrite মডিউলটি প্রয়োজনীয়।

```bash
sudo a2enmod rewrite
```

### Apache রিলোড করুন

```bash
sudo systemctl reload apache2
```

### Roundcube এর জন্য প্রয়োজনীয় PHP প্যাকেজ ইনস্টল করুন

Roundcube সঠিকভাবে কাজ করার জন্য আপনার বেশ কয়েকটি PHP এক্সটেনশন প্রয়োজন হবে। নিচের বিকল্পগুলোর মধ্যে **একটি** নির্বাচন করুন। প্রথম বিকল্পটি সাধারণত আরও ব্যাপক।

**বিকল্প 1 (প্রস্তাবিত):**

```bash
sudo apt install openssl composer php-net-smtp php-mysql php-gd php-xml php-mbstring php-intl php-zip php-json php-pear php-bz2 php-gmp php-imap php-imagick php-auth-sasl php-net-idna2 php-mail-mime php-net-ldap3 php-net-sieve -y
```

**বিকল্প 2 (নির্দিষ্ট PHP 7.4 - আপনার যদি PHP 7.4 ইনস্টল করা থাকে তবে ব্যবহার করুন):**

```bash
sudo apt install php7.4 libapache2-mod-php7.4 php7.4-mysql php-net-ldap2 php-net-ldap3 php-imagick php7.4-common php7.4-gd php7.4-imap php7.4-json php7.4-curl php7.4-zip php7.4-xml php7.4-mbstring php7.4-bz2 php7.4-intl php7.4-gmp php-net-smtp php-mail-mime php-net-idna2 -y
```

**PHP ইনস্টলেশন সমস্যা সমাধান (যদি আপনি ত্রুটি সম্মুখীন হন):**

আপনি যদি PHP প্যাকেজ ইনস্টল করার সময় সমস্যার সম্মুখীন হন, তাহলে নতুন সংস্করণের জন্য Ondrej PHP রিপোজিটরি যোগ করতে হতে পারে।

```bash
sudo apt -y install software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
sudo apt-get update -y
```

PHP প্যাকেজ ইনস্টল করার পর, Apache রিস্টার্ট করুন:

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

### Roundcube ডাউনলোড এবং এক্সট্রাক্ট করুন

Roundcube এর GitHub রিলিজ পেজ থেকে সর্বশেষ স্থিতিশীল সংস্করণ ডাউনলোড করুন। সর্বদা সর্বশেষ স্থিতিশীল রিলিজ যাচাই করুন। (এই নির্দেশিকা অনুযায়ী, `1.6.1` ব্যবহার করা হয়েছে, তবে Roundcube ওয়েবসাইটে যাচাই করুন)।

```bash
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.1/roundcubemail-1.6.1-complete.tar.gz
tar -xvf roundcubemail-1.6.1-complete.tar.gz
```

### Roundcube এর জন্য স্থানান্তরিত এবং পারমিশন সেট করুন

এক্সট্রাক্ট করা Roundcube ডিরেক্টরি আপনার ওয়েব সার্ভারের ডকুমেন্ট রুটে সরান এবং উপযুক্ত পারমিশন সেট করুন।

```bash
sudo mv roundcubemail-1.6.1 /var/www/html/roundcubemail
sudo chown -R www-data:www-data /var/www/html/roundcubemail/
sudo chmod 755 -R /var/www/html/roundcubemail/
```

### Roundcube এর জন্য MariaDB ডাটাবেস এবং ব্যবহারকারী তৈরি করুন

রুট ব্যবহারকারী হিসেবে MariaDB প্রম্পটে প্রবেশ করুন এবং Roundcube এর জন্য একটি ডাটাবেস ও ব্যবহারকারী তৈরি করুন। শক্তিশালী, সুরক্ষিত পাসওয়ার্ড দিয়ে `'ubuntu'` প্রতিস্থাপন করতে ভুলবেন না।

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

### Roundcube এর প্রাথমিক SQL স্কিমা আমদানি করুন

`roundcube` ডাটাবেসে প্রয়োজনীয় ডাটাবেস স্কিমা আমদানি করুন।

```bash
sudo mysql roundcube < /var/www/html/roundcubemail/SQL/mysql.initial.sql
```

### Roundcube এর জন্য Apache ভার্চুয়াল হোস্ট তৈরি করুন

Roundcube এর জন্য একটি নতুন Apache ভার্চুয়াল হোস্ট কনফিগারেশন ফাইল তৈরি করুন।

```bash
sudo nano /etc/apache2/sites-available/roundcube.conf
```

নিচের কনফিগারেশনটি যোগ করুন। `mail.paulco.xyz` এর জায়গায় আপনার মেইল সার্ভারের FQDN বসান।

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

### Roundcube ভার্চুয়াল হোস্ট সক্রিয় করুন এবং Apache রিলোড করুন

```bash
sudo a2ensite roundcube.conf
sudo systemctl reload apache2
```

### Dovecot এবং Postfix পুনরায় চালু করুন (আবার)

নিশ্চিত করুন যে নতুন কনফিগারেশন সম্পর্কে সমস্ত পরিষেবা অবগত আছে।

```bash
sudo systemctl restart dovecot
sudo systemctl restart postfix
```

### ওয়েব ইনস্টলারের মাধ্যমে Roundcube ইনস্টলেশন সম্পন্ন করুন

আপনার ওয়েব ব্রাউজার খুলুন এবং Roundcube ইনস্টলার URL-এ যান:

`http://mail.paulco.xyz/roundcubemail/installer/`

স্ক্রিনে দেখানো নির্দেশাবলী অনুসরণ করুন:

  * **ডাটাবেসের নাম (`roundcube`), ডাটাবেসের ব্যবহারকারীর নাম (`roundcubeuser`), এবং ডাটাবেসের পাসওয়ার্ড (আপনি যা সেট করেছেন, যেমন, `ubuntu`) সঠিকভাবে কনফিগার করতে ভুলবেন না।**
  * **IMAP** সেটিংসে, `localhost` এবং ডিফল্ট পোর্ট `143` (বা SSL/TLS কনফিগার করা থাকলে `993`) ব্যবহার করুন।
  * **SMTP** সেটিংসে, `localhost` এবং পোর্ট `25` (বা প্রমাণীকরণ সহ সাবমিশনের জন্য `587` যদি কনফিগার করা থাকে) ব্যবহার করুন। **আপাতত SMTP সেটিংসে কোনো ব্যবহারকারীর নাম/পাসওয়ার্ড সেট করবেন না, কারণ Postfix অভ্যন্তরীণ ব্যবহারকারীদের জন্য প্রমাণীকরণ ছাড়াই স্থানীয় সাবমিশনের জন্য কনফিগার করা হয়েছে।**

### ইনস্টলার ডিরেক্টরি মুছে ফেলুন

সফল ইনস্টলেশনের পরে, সুরক্ষার কারণে ইনস্টলার ডিরেক্টরি মুছে ফেলা অত্যন্ত গুরুত্বপূর্ণ।

```bash
sudo rm -r /var/www/html/roundcubemail/installer/
```

Roundcube এর মাধ্যমে ইমেল পাঠানো এবং গ্রহণ করা পরীক্ষা করার আগে, উবুন্টুতে প্রতিটি কাঙ্ক্ষিত ইমেল ঠিকানার জন্য ব্যবহারকারী তৈরি করতে `adduser` কমান্ড ব্যবহার করে মেইল ব্যবহারকারী তৈরি করতে ভুলবেন না (যেমন, `adduser user1`, `adduser user2`)।

-----

## DNS এ SPF, DKIM, DMARC রেকর্ড যোগ করা

ইমেলের ডেলিভারিবিলিটি এবং স্প্যাম প্রতিরোধ করার জন্য SPF, DKIM এবং DMARC প্রয়োগ করা অত্যন্ত গুরুত্বপূর্ণ। এই রেকর্ডগুলি আপনার আউটগোয়িং ইমেলগুলিকে প্রমাণীকরণ করতে সহায়তা করে, যা সেগুলিকে স্প্যাম হিসাবে চিহ্নিত হওয়ার সম্ভাবনা হ্রাস করে।

**রেফারেন্স**: [https://www.linuxbabe.com/mail-server/setting-up-dkim-and-spf](https://www.linuxbabe.com/mail-server/setting-up-dkim-and-spf)

-----

## SPF রেকর্ড কিভাবে সেট আপ করবেন

**Sender Policy Framework (SPF)** হল একটি ইমেল প্রমাণীকরণ পদ্ধতি যা ইমেলের জাল প্রেরকের ঠিকানা সনাক্ত করার জন্য ডিজাইন করা হয়েছে।

### বিদ্যমান TXT রেকর্ড পরীক্ষা করুন (ঐচ্ছিক)

আপনি `dig` বা `nslookup` ব্যবহার করে আপনার ডোমেইনের বিদ্যমান TXT রেকর্ড পরীক্ষা করতে পারেন।

```bash
dig txt paulco.xyz
```

`nslookup` ব্যবহার করে:

```bash
nslookup
> set type=txt
> paulco.xyz
```

### DNS সার্ভারে SPF রেকর্ড যোগ করুন

আপনার ডোমেইনের DNS সেটিংসে একটি TXT রেকর্ড যোগ করতে হবে। এটি আপনার স্থানীয় DNS সার্ভারে (যদি আপনি একটি পরিচালনা করছেন) অথবা আপনার ডোমেইন রেজিস্ট্রারের DNS ম্যানেজমেন্ট প্যানেলের মাধ্যমে করা যেতে পারে।

**`nano /etc/bind/forward.zone` এর উদাহরণ (যদি Bind9 ব্যবহার করেন):**

```
@  IN  TXT "v=spf1 a mx ip4:202.4.111.69 ~all"
```

**ডোমেইন প্যানেলের উদাহরণ:**

আপনার ডোমেইন ড্যাশবোর্ডে (যেমন, Namecheap, GoDaddy, Cloudflare) যান -\> DNS ম্যানেজমেন্ট -\> একটি নতুন TXT রেকর্ড যোগ করুন:

  * **Type**: `TXT`
  * **Host/Name**: `@` (অথবা রুট ডোমেইনের জন্য খালি রাখুন)
  * **Value/Text**: `"v=spf1 a mx ip4:202.4.111.69 ~all"`
  * **TTL**: `Auto` অথবা `3600` (সেকেন্ড)

**SPF রেকর্ডের ব্যাখ্যা:**

  * `v=spf1`: SPF সংস্করণ নির্দিষ্ট করে।
  * `a`: ডোমেইনের A রেকর্ডকে ইমেল পাঠানোর অনুমতি দেয়।
  * `mx`: ডোমেইনের MX রেকর্ডগুলিকে ইমেল পাঠানোর অনুমতি দেয়।
  * `ip4:202.4.111.69`: স্পষ্টভাবে নির্দিষ্ট IPv4 অ্যাড্রেসকে ইমেল পাঠানোর অনুমতি দেয়। `202.4.111.69` এর জায়গায় আপনার মেইল সার্ভারের পাবলিক আইপি অ্যাড্রেস বসান।
  * `~all`: একটি "সফটফেইল" মেকানিজম, যার অর্থ অন্যান্য উৎস থেকে আসা ইমেলগুলি বৈধ হতে পারে, তবে সেগুলিকে সন্দেহের চোখে দেখা উচিত। কঠোর নীতির জন্য, আপনি `-all` (হার্ডফেইল) ব্যবহার করতে পারেন, তবে প্রাথমিকভাবে `~all` ব্যবহার করার ক্ষেত্রে সতর্ক থাকুন।

### SPF রেকর্ড যাচাই করুন

রেকর্ড যোগ করার পর, এটিকে প্রচারের জন্য কিছু সময় দিন (DNS প্রচার কয়েক মিনিট থেকে কয়েক ঘন্টা পর্যন্ত সময় নিতে পারে)। তারপর, `dig` বা `nslookup` ব্যবহার করে আবার পরীক্ষা করুন।

-----

### ঐচ্ছিক: SPF পলিসি এজেন্ট কনফিগার করা (Postfix-policyd-spf-python)

Postfix এর জন্য এই পলিসি এজেন্ট SPF চেক ব্যর্থ হওয়া ইনকামিং ইমেল প্রত্যাখ্যান করতে সাহায্য করে।

```bash
sudo apt install postfix-policyd-spf-python -y
```

Postfix মাস্টার কনফিগারেশন ফাইলটি এডিট করুন:

```bash
sudo nano /etc/postfix/master.cf
```

ফাইলের শেষে নিচের লাইনগুলো যোগ করুন:

```
policyd-spf  unix  -       n       n       -       0       spawn
  user=policyd-spf argv=/usr/bin/policyd-spf
```

ফাইলটি সেভ করুন এবং বন্ধ করুন। এরপর, Postfix এর প্রধান কনফিগারেশন ফাইলটি এডিট করুন:

```bash
sudo nano /etc/postfix/main.cf
```

ফাইলের শেষে নিচের লাইনগুলো যোগ করুন:

```
policyd-spf_time_limit = 3600
smtpd_recipient_restrictions =
  permit_mynetworks,
  permit_sasl_authenticated,
  reject_unauth_destination,
  check_policy_service unix:private/policyd-spf
```

ফাইলটি সেভ করুন এবং বন্ধ করুন। তারপর Postfix রিস্টার্ট করুন:

```bash
sudo systemctl restart postfix
```

এর পর, যখন আপনি একটি SPF রেকর্ড আছে এমন ডোমেইন থেকে একটি ইমেল পাবেন, আপনি কাঁচা ইমেল হেডার-এ SPF চেক ফলাফল দেখতে পারবেন, উদাহরণস্বরূপ:

`Received-SPF: Pass (sender SPF authorized).`

-----

## DMARC রেকর্ড কিভাবে সেট আপ করবেন

**Domain-based Message Authentication, Reporting, and Conformance (DMARC)** একটি ইমেল প্রমাণীকরণ প্রোটোকল যা SPF এবং DKIM এর উপর ভিত্তি করে তৈরি। এটি ডোমেইন মালিকদের নির্দেশিকা দেয় যে SPF বা DKIM চেক ব্যর্থ হলে তাদের ইমেলগুলি কীভাবে পরিচালনা করা উচিত এবং ইমেল প্রমাণীকরণ ব্যর্থতার উপর রিপোর্ট গ্রহণ করার অনুমতি দেয়।

### ডোমেইন ড্যাশবোর্ডে DMARC রেকর্ড যোগ করুন

আপনার ডোমেইন ড্যাশবোর্ডে (যেমন, Namecheap, GoDaddy, Cloudflare) যান -\> DNS ম্যানেজমেন্ট -\> একটি নতুন TXT রেকর্ড যোগ করুন:

  * **Type**: `TXT`
  * **Host/Name**: `_dmarc`
  * **Value/Text**: `v=DMARC1; p=quarantine; aspf=r; sp=none; rua=mailto:your_email@paulco.xyz; ruf=mailto:your_email@paulco.xyz`
      * **ব্যাখ্যা:**
          * `v=DMARC1`: DMARC সংস্করণ নির্দিষ্ট করে।
          * `p=quarantine`: DMARC চেক ব্যর্থ হওয়া ইমেলগুলির জন্য নীতি। `quarantine` মানে প্রাপকদের তাদের সন্দেহজনক হিসাবে বিবেচনা করা উচিত (যেমন, স্প্যাম ফোল্ডারে সরানো)। অন্যান্য বিকল্প হল `none` (শুধুমাত্র মনিটর, কোনো অ্যাকশন নেই) এবং `reject` (সম্পূর্ণভাবে প্রত্যাখ্যান)। প্রাথমিকভাবে `none` বা `quarantine` দিয়ে শুরু করুন এবং নিশ্চিত হলে `reject` এ যান।
          * `aspf=r`: SPF এর জন্য অ্যালাইনমেন্ট মোড। `r` মানে রিল্যাক্সড অ্যালাইনমেন্ট।
          * `sp=none`: সাবডোমেনগুলির জন্য নীতি। `none` মানে সাবডোমেনগুলির জন্য কোনো নির্দিষ্ট নীতি নেই, তারা মূল ডোমেইনের নীতি উত্তরাধিকার সূত্রে পায়।
          * `rua=mailto:your_email@paulco.xyz`: সমষ্টিগত DMARC রিপোর্ট পাঠানোর জন্য ইমেল ঠিকানা। **`your_email@paulco.xyz` কে একটি আসল ইমেল ঠিকানা দিয়ে প্রতিস্থাপন করুন যেখানে আপনি রিপোর্ট পেতে চান।**
          * `ruf=mailto:your_email@paulco.xyz`: ফরেনসিক (ব্যর্থতা) DMARC রিপোর্ট পাঠানোর জন্য ইমেল ঠিকানা। **`your_email@paulco.xyz` কে একটি আসল ইমেল ঠিকানা দিয়ে প্রতিস্থাপন করুন।**

-----

## DKIM রেকর্ড কিভাবে সেট আপ করবেন

**DomainKeys Identified Mail (DKIM)** একটি ইমেল প্রমাণীকরণ পদ্ধতি যা নকল প্রেরকের ঠিকানা সনাক্ত করার জন্য ডিজাইন করা হয়েছে। DKIM প্রাপককে পরীক্ষা করতে দেয় যে একটি ইমেল আসলে উল্লিখিত ডোমেইন থেকে পাঠানো হয়েছে কিনা।

### OpenDKIM এবং OpenDKIM টুলস ইনস্টল করুন

```bash
sudo apt-get install opendkim opendkim-tools -y
```

### OpenDKIM সার্ভিস চালু এবং সক্রিয় করুন

```bash
sudo systemctl start opendkim
sudo systemctl enable opendkim
```

### OpenDKIM ডিরেক্টরি তৈরি করুন

```bash
mkdir /etc/opendkim
```

### DKIM কী তৈরি করুন

আপনার ডোমেইনের জন্য প্রাইভেট এবং পাবলিক DKIM কী তৈরি করুন। `paulco.xyz` এর জায়গায় আপনার ডোমেইন বসান।

```bash
opendkim-genkey -D /etc/opendkim/ --domain paulco.xyz --selector mail
```

এই কমান্ডটি দুটি ফাইল তৈরি করবে: `mail.private` (আপনার প্রাইভেট কী) এবং `mail.txt` (আপনার পাবলিক কী, যা DNS রেকর্ডে যোগ করতে হবে)।

### DKIM পাবলিক কী দেখুন

আপনার পাবলিক কী ফাইলের বিষয়বস্তু প্রদর্শন করুন। DNS রেকর্ডের জন্য আপনার এই তথ্যের প্রয়োজন হবে।

```bash
cat /etc/opendkim/mail.txt
```

আউটপুটটি দেখতে নিচের মতো হবে:

```
mail._domainkey IN TXT ( "v=DKIM1; h=sha256; k=rsa; "
          "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAqm42/tBAcceewB+FTl5SWVUT0Xa80QZfBBjP4xP16xRJo7YsOoaB9DZlVehNbZxPBJLhDiJdEyV9nDODqABroInlLM0pT2pRGHcXoiamU5tSzWWMWfd8d4kOZZgSeDyJk89+XNHMZeKRX45fJBdxzWVmaFRljxnEz90496aYScyr8WFGSUHCtZD1n2Rq/8O6u8Q8HJKoHfV2fJ"
          "/ZQxN3Z2uUGWBjvB1fjaszHr/iZpsbMRTQM9gAxLI1xmtgs1k4jQua+deZWCLzylLeOW05/itDz87psLBOAZsqH2yh+x6FY1Mf2jGaROV0nSBY0Nf2JyY05FOVB0JgvOEKL569jwIDAQAB" )  ; ----- DKIM key mail for paulco.xyz
```

**গুরুত্বপূর্ণ**: এটি আপনার DNS এ যোগ করার সময়, উদ্ধৃতি চিহ্ন, বন্ধনী এবং নতুন লাইনগুলি সরিয়ে দিন যাতে এটি একটি একক ধারাবাহিক স্ট্রিং হয়।

**DNS এন্ট্রির জন্য এটি কেমন হওয়া উচিত তার উদাহরণ (একক লাইন, কোনো উদ্ধৃতি চিহ্ন নেই, কোনো অতিরিক্ত স্থান নেই):**

```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAqm42/tBAcceewB+FTl5SWVUT0Xa80QZfBBjP4xP16xRJo7YsOoaB9DZlVehNbZxPBJLhDiJdEyV9nDODqABroInlLM0pT2pRGHcXoiamU5tSzWWMWfd8d4kOZZgSeDyJk89+XNHMZeKRX45fJBdxzWVmaFRljxnEz90496aYScyr8WFGSUHCtZD1n2Rq/8O6u8Q8HJKoHfV2fJ/ZQxN3Z2uUGWBjvB1fjaszHr/iZpsbMRTQM9gAxLI1xmtgs1k4jQua+deZWCLzylLeOW05/itDz87psLBOAZsqH2yh+x6FY1Mf2jGaROV0nSBY0Nf2JyY05FOVB0JgvOEKL569jwIDAQAB
```

### DNS এ DKIM রেকর্ড যোগ করুন

আপনার ডোমেইন ড্যাশবোর্ডে যান -\> DNS ম্যানেজমেন্ট -\> একটি নতুন TXT রেকর্ড যোগ করুন:

  * **Type**: `TXT`
  * **Host/Name**: `mail._domainkey` (আপনি যে সিলেক্টর ব্যবহার করেছেন, তার পরে `._domainkey`)
  * **Value/Text**: আপনি উপরে ফরম্যাট করা একক-লাইনের পাবলিক কী স্ট্রিংটি পেস্ট করুন।
  * **TTL**: `Auto` অথবা `3600`

### OpenDKIM ডিরেক্টরির মালিকানা সেট করুন

```bash
sudo chown -R opendkim:opendkim /etc/opendkim
```

### OpenDKIM কনফিগার করুন

OpenDKIM এর প্রধান কনফিগারেশন ফাইলটি এডিট করুন।

```bash
sudo nano /etc/opendkim.conf
```

নিশ্চিত করুন যে নিচের লাইনগুলো আনকমেন্ট করা হয়েছে অথবা দেখানো অনুযায়ী যোগ/পরিবর্তন করা হয়েছে:

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

### আপনার ডোমেইনকে ট্রাস্টেড হোস্ট হিসাবে যোগ করুন

OpenDKIM এর জন্য `TrustedHosts` ফাইলটি তৈরি বা এডিট করুন।

```bash
sudo nano /etc/opendkim/TrustedHosts
```

এই ফাইলে আপনার ডোমেইন এবং `localhost` (অথবা যে কোনো অভ্যন্তরীণ নেটওয়ার্ক যা ইমেল পাঠাবে) যোগ করুন:

```
127.0.0.1
localhost
.paulco.xyz
```

### কী টেবিল সংজ্ঞায়িত করুন

`KeyTable` ফাইলটি তৈরি বা এডিট করুন, যা প্রাইভেট কীগুলির সাথে সিলেক্টরগুলিকে ম্যাপ করে।

```bash
sudo nano /etc/opendkim/KeyTable
```

নিচের লাইনটি যোগ করুন:

```
mail._domainkey.paulco.xyz paulco.xyz:mail:/etc/opendkim/mail.private
```

### সাইনিং টেবিল সংজ্ঞায়িত করুন

`SigningTable` ফাইলটি তৈরি বা এডিট করুন, যা নির্দিষ্ট করে কোন ডোমেইনগুলিকে কোন কী দিয়ে স্বাক্ষর করা উচিত।

```bash
sudo nano /etc/opendkim/SigningTable
```

নিচের লাইনগুলো যোগ করুন:

```
*@paulco.xyz mail._domainkey.paulco.xyz
*@*.paulco.xyz mail._domainkey.paulco.xyz
```

### OpenDKIM এবং Postfix রিস্টার্ট করুন

OpenDKIM পরিবর্তনগুলি প্রয়োগ করুন এবং উভয় পরিষেবা পুনরায় চালু করুন।

```bash
sudo systemctl restart opendkim
sudo systemctl restart postfix
```

-----

### DKIM রেকর্ড এবং মেইল পারফরম্যান্স পরীক্ষা করুন

### টার্মিনালে DKIM যাচাই করুন

আপনি `opendkim-testkey` ব্যবহার করে আপনার DKIM রেকর্ড পরীক্ষা করতে পারেন:

```bash
sudo opendkim-testkey -d paulco.xyz -s mail -vvv
```

আপনার DNS রেকর্ড এবং কনফিগারেশন সঠিক হলে এটি `Key OK` আউটপুট দেখাবে।

### অনলাইন টুল ব্যবহার করে DKIM পরীক্ষা করুন

অনলাইন টুলগুলি আপনার DKIM রেকর্ড যাচাই করতে সহায়তা করতে পারে।

  * ভিজিট করুন: [https://dmarcian.com/dkim-inspector/](https://dmarcian.com/dkim-inspector/)
  * আপনার ডোমেইন (`paulco.xyz`) এবং সিলেক্টর (`mail`) লিখুন।

### মেইল পারফরম্যান্স পরীক্ষা করুন

আপনার মেইল সার্ভারের SPF, DKIM, এবং DMARC কনফিগারেশন পুঙ্খানুপুঙ্খভাবে পরীক্ষা করতে, Mail-Tester এর মতো একটি সেবায় একটি পরীক্ষা ইমেল পাঠান।

  * ভিজিট করুন: [https://www.mail-tester.com/](https://www.mail-tester.com/)
  * Mail-Tester দ্বারা প্রদত্ত অনন্য ইমেল ঠিকানাটি কপি করুন।
  * আপনার নতুন কনফিগার করা মেইল সার্ভার থেকে এই ঠিকানায় একটি পরীক্ষা ইমেল পাঠান (উদাহরণস্বরূপ, Roundcube বা `mail` কমান্ড ব্যবহার করে)।

<!-- end list -->

```bash
echo "Hey, I am dev stack. I want to connect with you sumon" | mail -s "Connected Mail" sumonpaul267@gmail.com
```

  * Mail-Tester-এ ফিরে যান এবং আপনার স্কোর পরীক্ষা করুন। ভাল ডেলিভারিবিলিটি নিশ্চিত করতে একটি উচ্চ স্কোর (সাধারণত 9/10 বা 10/10) লক্ষ্য করুন।

-----

## উপসংহার

আপনি সফলভাবে Postfix, Dovecot এবং Roundcube ইনস্টল ও কনফিগার করেছেন, যা আপনার মেইল সার্ভারকে কার্যকরী করেছে। আপনি SPF, DKIM, এবং DMARC-এর মতো প্রয়োজনীয় DNS রেকর্ডগুলিও প্রয়োগ করেছেন যা ইমেলের ডেলিভারিবিলিটি উন্নত করতে এবং স্প্যাম প্রতিরোধ করতে সাহায্য করে।

এই সেটআপের মাধ্যমে, এখন আপনার কাছে একটি শক্তিশালী, স্ব-হোস্ট করা ইমেল সমাধান রয়েছে। আপনার সার্ভার এবং সফটওয়্যার নিয়মিত আপডেট করতে এবং যেকোনো সমস্যার জন্য আপনার মেইল লগগুলি নিরীক্ষণ করতে ভুলবেন না।

আপনি কি পরবর্তীতে আপনার মেইল সার্ভারকে SSL/TLS দিয়ে সুরক্ষিত করতে, ভার্চুয়াল ব্যবহারকারী এবং ডোমেইন যোগ করতে, নাকি কর্মক্ষমতা আরও অপ্টিমাইজ করতে চান?
