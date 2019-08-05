<font size='10' >Hackmd Server</font> 

在ubuntu 18.04 (or newest)上安裝Hackmd Server (Codimd)

## 安裝Docker & Docker-compose
```bash
sudo apt install docker docker-compose 
```

## Appaemor 問題

我們必須注意一點，在ubuntu 18.04後 Apparmor 變成預設啟動得東西，AppArmor 的安全模型將對存取屬性的控制繫結到程式而非用戶。

Docker官方的[Documetation](https://docs.docker.com/engine/security/apparmor/)表明，其會自動生成一個 Profile放在```/etc/apparmor/docker``` 但是我在 Ubuntu 18.04 下卻沒有看到這個 Profile，所以我們只好自己產一個。

語法的部分可以參考Apparmor的gitlab
* [Quick Profile Language](https://gitlab.com/apparmor/apparmor/wikis/QuickProfileLanguage)
* [Globbing Syntax](https://gitlab.com/apparmor/apparmor/wikis/AppArmor_Core_Policy_Reference#AppArmor_globbing_syntax)

```c
#include <tunables/global>


profile docker-default flags=(attach_disconnected,mediate_deleted) {

  #include <abstractions/base>


  network,
  capability,
  file,
  umount,

  deny @{PROC}/* w,   # deny write for all files directly in /proc (not in a subdir)
  # deny write to files not in /proc/<number>/** or /proc/sys/**
  deny @{PROC}/{[^1-9],[^1-9][^0-9],[^1-9s][^0-9y][^0-9s],[^1-9][^0-9][^0-9][^0-9]*}/** w,
  deny @{PROC}/sys/[^k]** w,  # deny /proc/sys except /proc/sys/k* (effectively /proc/sys/kernel)
  deny @{PROC}/sys/kernel/{?,??,[^s][^h][^m]**} w,  # deny everything except shm* in /proc/sys/kernel/
  deny @{PROC}/sysrq-trigger rwklx,
  deny @{PROC}/mem rwklx,
  deny @{PROC}/kmem rwklx,
  deny @{PROC}/kcore rwklx,

  deny mount,

  deny /sys/[^f]*/** wklx,
  deny /sys/f[^s]*/** wklx,
  deny /sys/fs/[^c]*/** wklx,
  deny /sys/fs/c[^g]*/** wklx,
  deny /sys/fs/cg[^r]*/** wklx,
  deny /sys/firmware/efi/efivars/** rwklx,
  deny /sys/kernel/security/** rwklx,


  # suppress ptrace denials when using 'docker ps' or using 'ps' inside a container
  ptrace (trace,read) peer=docker-default,

}
```
加載配置文件
```bash
sudo apparmor_parser -r -W /etc/apparmor.d/docker
```
搞定後重啟 Apparmor

讓我們來看一下 Profile 有沒有被正確載入
```shell
root@xxxx:# aa-status | grep docker
   docker-default
   snap-update-ns.docker
   snap.docker.compose
   snap.docker.docker
   snap.docker.dockerd
   snap.docker.help
   snap.docker.hook.install
   snap.docker.hook.post-refresh
   snap.docker.machine
   /usr/local/bin/postgres (3227) docker-default
   /usr/local/bin/postgres (3355) docker-default
   /usr/local/bin/postgres (3356) docker-default
   /usr/local/bin/postgres (3357) docker-default
   /usr/local/bin/postgres (3358) docker-default
   /usr/local/bin/postgres (3359) docker-default
   /usr/local/bin/node (3363) docker-default
   /usr/local/bin/node (3487) docker-default
   /snap/docker/384/bin/dockerd (1195) snap.docker.dockerd
   /snap/docker/384/bin/docker-containerd (1708) snap.docker.dockerd
```
!> 我們只要看到最上面那個 **docker-default** 有被載入就好了，至於下面那堆snap....對現在無論你是server版還是desktop版都預設加入了snap這個邪教。


## Up your container 

去github把codimd-container拉下來
```bash
cd /home
git clone https://github.com/hackmdio/codimd-container.git
```

```bash
docker-compose up -d
```

現在可以連上 http://your-ip:3000 使用 HackMd 寫文章了。


## Nginx proxy_pass

一般來說我們不會把服務開在 3000 port 上，而且目前也沒有https。從設定跟維護得角度來說使用nginx proxy_pass把網站拉到 80 port上是最好的方法。

因此讓我們新建一個檔案```/etc/nginx/sites-available/hackmd.conf```
```shell=
server {
    server_name = xxx.xxx;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localhost:3000;
        client_max_body_size       600m; 
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/xxxx.pem # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/xxxx.pem # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


```

!> ```client_max_body_size```一定要設定最大文件通過量，不然使用者的圖片傳不上去。

然後再用 return 301 把所有 request 扔到 https 去
```bash=
server {
    if ($host = xxx.xxx) {
        return 301 https://$host$request_uri;
    } 


    listen 80;
    server_name = xxx.xxx;
    return 404;

}
```
```bash
ln -s /etc/nginx/site-available/hackmd.conf /etc/nginx/site-enable/
systemctl restart nginx
```
搞定！接下來我們來處裡認證得部分。

## OAuth

在登入的選項中除了預設得mail註冊以外還可以使用OAuth去連結google, github, gitlab 等帳號。

詳細請參考：https://hackmd.io/c/codimd-documentation/%2Fs%2Fcodimd-configuration