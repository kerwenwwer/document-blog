# Webmail Roundcube 架設指南
###### tags: `roundcube` `install`

## Interduction to Roundcube
https://roundcube.net/

## Sep1. Install Nginx, PHP-FPM and MariaDB in CentOS 7
#### 1. Indtall every package
```bash
$ yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
$ yum install yum-utils
$ yum-config-manager --enable remi-php72
$ yum install nginx php php-fpm php-mcrypt php-cli php-gd php-curl php-xml php-mysql php-mbstring php-pspell php-imagick mariadb-server   
```

#### 2. Check Nginx 

 Open the firewall port
```bash
$ firewall-cmd --permanent --add-port=80/tcp
$ firewall-cmd --permanent --add-port=443/tcp
$ firewall-cmd --reload
```
Open nginx service
```bash
$ systemctl start nginx
$ systemctl enable nginx
```


#### 3. configure PHP-FPM

you need to configure the file at ```/etc/php/php.ini```

Look for the directive ```;cgi.fix_pathinfo=1,``` uncomment it and set its value to 0.
```bash=
cgi.fix_pathinfo=0
```
Also uncomment the directive ;date.timezone and set its value to your timezone
```bash=
data.timezone = "Asia/Taipei"
```
> [!WARNING]
> 注意！！！一定要設定最大文件通過量



```bash=
post_max_size = 16M

upload_max_filesize = 16M

max_file_uploads = 20
```
Once you are done, save the file and exit.

#### 4. Then start PHP-FPM service, enable it to auto-start at boot time and check if its up and running, as follows.

```bash
systemctl start php-fpm
systemctl enable php-fpm
```
## Setp2. Secure MariaDB Server and Create Roundecube Database

1. Open mariadb service

```bash
$ systemctl start mariadb
$ systemctl enable mariadb
```

```bash
$ mysql_secure_installation
```

```bash
 mysql -u root -p
```
Setting database 

```bash
> CREATE DATABASE (database name);
> GRANT ALL PRIVILEGES ON (database name).* TO (user name)@localhost
> FLUSH PRIVILEGES;
> quit;
```

> 數請記下來，之後會使用到
> * webmail的SQL資料:
> * (database name) = roundcube
> * (user name) = mailuser
> * (user passwd) = roundcube



## Setp3. Download Roundcube Package

Check the newest version on Roundcube website
https://roundcube.net/download/

目前版本為1.3.8
```bash
$ wget https://github.com/roundcube/roundcubemail/releases/download/1.3.8/roundcubemail-1.3.8-complete.tar.gz
$ tar xzf roundcubemail-1.3.7-complete.tar.gz && mv roundcubemail-1.3.7 /var/www/html/roundcubemail
```
Set nginx to be the user of ```/var/www/html/roundcubemail```
```bash
$ chown -R nginx:nginx /var/www/html/roundcubemail
```

## Step4. Configure Nginx Server Block For Roundcube Web Installer

1.Now create an Nginx server block for the roundcube under /etc/nginx/conf.d/ (you can name the file the way you want but it should have a .conf extension).


> 在nginx的設定中，我們可以把服務接成各個block
> 全域conf 放在```/etc/nginx/nginx.conf```各個網站的block則放在```/etc/nginx/conf.d/```下


```bash 
$ vim /etc/nginx/conf.d/mail.example.com.conf
```

And copy this into the file

```bash=
server {
        listen 80;
        server_name webmail.cs.nthu.edu.tw;

        root /var/www/html/roundcubemail;
        index  index.php index.html;

        #i# Logging
        access_log /var/log/nginx/mail.example.com_access_log;
        error_log   /var/log/nginx/mail.example.com_error_log;

        location / {
                try_files $uri $uri/ /index.php?q=$uri&$args;
        }

        location ~ ^/(README.md|INSTALL|LICENSE|CHANGELOG|UPGRADING)$ {
                deny all;
        }

        location ~ ^/(config|temp|logs)/ {
                deny all;
        }

        location ~ /\. {
                deny all;
                access_log off;
                log_not_found off;
        }

        location ~ \.php$ {
                include /etc/nginx/fastcgi_params;
                #fastcgi_pass 127.0.0.1:9000;
                fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
}
```


> 注意在安裝過程中我們只開放htt服務，https則是在安裝完成後另外對nginx進行設定。


2. Next, open the file /etc/php-fpm.d/www.conf to make a few changes to PHP-FPM web directive.
```bash
$ vim /etc/php-fpm/www.conf
```
Change the user apache to nginx in the following variables.

```bash=
user = nginx
group = nginx
```

```bash=
listen = /var/run/php-fpm/php-fpm.sock
```

```bash=
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

> php 的conf檔註解格式，是在該行指令前加上";"，如果要啟用該行指令，刪除前面的註解符號即可

```bash
$ systemctl restart nginx php-fpm
```
## Step5. Access Roundcube Web UI

change the dir user 
```bash
$ ls -ld /var/lib/php/session/
$ chown :nginx /var/lib/php/session/
$ ls -ld /var/lib/php/session/
```

> 注意 資料夾使用者跟權限相關，因此尤其重要
> 若要查看目前資料夾的使用者設定
> command: ls -al


Open https://webmail.cs.nthu.edu.tw/installer

![](https://i.imgur.com/j294rnc.png)

確認ok的數量跟上面一樣
確認以下設定都是正確的
![](https://i.imgur.com/XJbAQ91.png)
![](https://i.imgur.com/p61XeLZ.png)
![](https://i.imgur.com/u1sTlUr.png)
![](https://i.imgur.com/3v1mAxD.png)

將本config檔案儲存在

```bash=
<?php

/* Local configuration for Roundcube Webmail */

// ----------------------------------
// SQL DATABASE
// ----------------------------------
// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// NOTE: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
$config['db_dsnw'] = 'mysql://mailuser:roundcube@localhost/roundcube';

// ----------------------------------
// IMAP
// ----------------------------------
// The IMAP host chosen to perform the log-in.
// Leave blank to show a textbox at login, give a list of hosts
// to display a pulldown menu or set one host as string.
// To use SSL/TLS connection, enter hostname with prefix ssl:// or tls://
// Supported replacement variables:
// %n - hostname ($_SERVER['SERVER_NAME'])
// %t - hostname without the first part
// %d - domain (http hostname $_SERVER['HTTP_HOST'] without the first part)
// %s - domain name after the '@' from e-mail address provided at login screen
// For example %n = mail.domain.tld, %t = domain.tld
// WARNING: After hostname change update of mail_host column in users table is
//          required to match old user data records with the new host.
$config['default_host'] = 'cs.nthu.edu.tw';

// ----------------------------------
// INTERFACE SETTING
// ----------------------------------
$config['default_font_size'] = '12pt';
$config['layout'] = 'desktop';
$config['default_charset'] = 'UTF-8';
// When replying:
// -1 - don't cite the original message
// 0  - place cursor below the original message
// 1  - place cursor above original message (top posting)
$config['reply_mode'] = 1;


// ----------------------------------
// SMTP
// ----------------------------------
// SMTP server host (for sending mails).
// Enter hostname with prefix tls:// to use STARTTLS, or use
// prefix ssl:// to use the deprecated SSL over SMTP (aka SMTPS)
// Supported replacement variables:
// %h - user's IMAP hostname
// %n - hostname ($_SERVER['SERVER_NAME'])
// %t - hostname without the first part
// %d - domain (http hostname $_SERVER['HTTP_HOST'] without the first part)
// %z - IMAP domain (IMAP hostname without the first part)
// For example %n = mail.domain.tld, %t = domain.tld
$config['smtp_server'] = 'smtp.cs.nthu.edu.tw';

// provide an URL where a user can get support for this Roundcube installation
// PLEASE DO NOT LINK TO THE ROUNDCUBE.NET WEBSITE HERE!
$config['support_url'] = '';

// This key is used for encrypting purposes, like storing of imap password
// in the session. For historical reasons it's called DES_key, but it's used
// with any configured cipher_method (see below).
$config['des_key'] = '6sDH3aWaYNSQmqlnhLbb4br0';

// Name your service. This is displayed on the login screen and in the window title
$config['product_name'] = ' NTHU_CS Webmail';

// ----------------------------------
// PLUGINS
// ----------------------------------
// List of active plugins (in plugins/ directory)
$config['plugins'] = array('enigma', 'filesystem_attachments', 'jqueryui', 'new_user_dialog', 'new_user_identity', 'newmail_notifier', 'password', 'userinfo', 'zipdownload','mobile','filters');

// Set the spell checking engine. Possible values:
// - 'googie'  - the default (also used for connecting to Nox Spell Server, see 'spellcheck_uri' setting)
// - 'pspell'  - requires the PHP Pspell module and aspell installed
// - 'enchant' - requires the PHP Enchant module
// - 'atd'     - install your own After the Deadline server or check with the people at http://www.afterthedeadline.com before using their API
// Since Google shut down their public spell checking service, the default settings
// connect to http://spell.roundcube.net which is a hosted service provided by Roundcube.
// You can connect to any other googie-compliant service by setting 'spellcheck_uri' accordingly.
$config['spellcheck_engine'] = 'pspell';
```

把/var/www/html/roundcubemail/installer 移到家目錄下。確保不會被外界連到。

然後就可以用了

## Setp6. https/SSL 

### 安裝 Certbot:

```bash
$ yum install mod_ssl -y 
$ yum install certbot -y
```

### 取得憑證

現在假設要申請 www.mydomain.com 及 mydomain.com 的憑證，先要在 DNS Server 將以上 hostname 指向伺服器的 IP，Let’s Encrypt 也會用 hostname 驗證是否域名持有人，而假設網頁的目錄在 /var/www/html, 可以執行以下指令：

```bash
$ certbot certonly --webroot -w /var/www/html -d webmail.cs.nthu.edu.tw -d --email <YOUR_EMAIL_ADDRESS> --agree-tos
```
當驗證成功後，SSL 憑證, private key 及 LE chain 會放在 /etc/letsencrypt/live/ webmail.cs.nthu.edu.tw/ 下面。

安裝完成以後重起nginx
```bash
$ systemctl restart nginx
```
然後可以測試是否可以正常renew憑證
```bash
$ certbot renew --dry-run
```
若看到成功字樣代表OK

### 加入排程
```bash
$ crontab -e 
```
加上一行
```bash=
0 0,12 * * * /usr/bin/certbot renew --quiet --no-self-upgrade "systemctl restart ngnix"
```
代表每天0點和12點的0分會自動renew一次

(雖然憑證3個月，但是官方建議12小時做一次 ，避免initiated revocation發生
以下是官方原文）

if you're setting up a cron or systemd job, we recommend running it twice per day (it won't do anything until your certificates are due for renewal or revoked, but running it regularly would give your site a chance of staying online in case a Let's Encrypt-initiated revocation happened for some reason). Please select a random minute within the hour for your renewal tasks.)

編輯完crotab之後，將crontab服務啟用並restart

```bash
$ systemctl enable crond
$ systemctl start  crond
```


參考：
https://docs.google.com/document/d/1uFJOE8twJ3ydPohPMrypdct-CzEZR9PEZXX2WS84HOw/edit

change nginx setting 
```/etc/nginx/conf.d/webmail.con```
```bash=
ssl_protocols TLSv1.2;
ssl_session_cache shared:SSL:9m;
ssl_session_cache shared:ssl_session_cache:10m;
client_max_body_size    1G;
```
:::danger
為了安全性
只保留了TSLv1.2;
然後就是nginx也要設定最大通過量
size就給1G
:::

## Sept7. Security 

參考

https://hackmd.io/oPkDuoApTaeqKsjCqjEiGA
https://hackmd.io/pmPJC9v9Slelh2JiE9_gIw

## Sept8. Plugin

### 新增手機瀏覽畫面
此次使用github上找到的手機瀏覽器plugin。
總共需要下載三個plugin

    a. https://github.com/messagerie-melanie2/Roundcube-Plugin-Mobile
    b. https://github.com/messagerie-melanie2/Roundcube-Skin-Melanie2-Larry-Mobile
    c. https://github.com/messagerie-melanie2/Roundcube-Plugin-JQuery-Mobile


分別將三個zip檔案解壓縮後

將a.下載的檔案資料夾更名成mobile，放入webmail5主機內roundcube資料夾中的plugins資料夾，資料夾位置請參考安裝roundcube的文件

2.2 將b.下載的檔案資料夾更名melanie2_larry_mobile，放入主機內roundcube資料夾中的skins資料夾。

2.3 將c.下載的檔案資料夾更名jquery_mobile，放入plugins資料夾

3.修改config資料夾中的config.inc.php，尋找$config['plugins'] 欄位，將其加入'mobile'，重新整理網頁即可完成。


