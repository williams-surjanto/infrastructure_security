## Introduction

Note: this repository assume you used docker in KALI LINUX with intel architecture and also some familiarity with Linux. If you used M1/M2/M3 related chip, you have to find your way to adjust with the lab. 

But wait what's intel arch? M1? M2? M3? Kali Linux? if you cannot answer this basic question its better to choose more basic stuff. Don't be lazy! Internet have a lot of good reading that cover basic of cyber security that you can read for free, use your brain! if you cannot simply do this simple things you will certainly getting replaced by AI in no time!

Reference: 
- DOCKER DEEP DIVE ZERO TO DOCKER IN A SINGLE BOOK, By Nigel Poulton
- Nginx Documentation
- Google

### Why docker?

| Point| Detail	   					     		 |
|-----:|-----------------------------------------------------------------|
|     1| In the past one app equal to one server	     		 |
|     2| New business => New app => New server => Cost go up 		 |
|     3| Then come VMWare, to solve this problem	     		 |
|     4| But it introduce new problem - Its slow to boot and not portable|

Then container came along! it has the same approach as VM but it does not require a full-blown OS, thus, its portable and easy to setup.

### Installing Docker
To install docker, open terminal and use the following command:
```
~# sudo apt install docker.io
~# sudo apt install docker-compose
```
Once the installation complete, you can check if the docker is installed correctly:
```
~# docker version
Client:
 Version:           26.1.5+dfsg1
 API version:       1.45
 Go version:        go1.24.2
 Git commit:        a72d7cd
 Built:             Sat May 24 17:38:32 2025
 OS/Arch:           linux/amd64
 Context:           default

```
Please be aware that result might be different but thats not gonna be a problem for now.
 
### Get your Hands Dirty

Run the following command in the terminal:
```
~# sudo docker images           
REPOSITORY                 TAG       IMAGE ID       CREATED       SIZE
```
To used docker you need to used sudo! As you can see this will show docker images as the name implies. Images is basically contain all the necessary component including dependencies to run application.

Okay, now let's try to download our first image!
Run the following command in the terminal:
```
~# sudo docker pull ubuntu:plucky
plucky: Pulling from library/ubuntu
60fb2420030a: Pull complete 
Digest: sha256:95a416ad2446813278ec13b7efdeb551190c94e12028707dd7525632d3cec0d1
Status: Downloaded newer image for ubuntu:plucky
docker.io/library/ubuntu:plucky
```
Once download is done, you can check by entering this command again:
```
~# sudo docker images                           
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       plucky    92598a7a70c7   3 weeks ago   77MB
```
Let's create the container(Think about image is the raw material and container is the product of that raw material).
Now that the container is ready, its time to start it:
```
~# sudo docker run -it ubuntu:plucky /bin/bash   
root@7b6c74e6fd4d:/# 
```
The container in a very minimalistic environment, we need to do some installation! run the following command in the docker:
```
root@7b6c74e6fd4d:/# apt update
root@7b6c74e6fd4d:/# apt install net-tools
root@7b6c74e6fd4d:/# apt install nano
```
Once installation complete, we can use the standard linux network cli, like:
```
root@7b6c74e6fd4d:/# netstat -aptn
root@7b6c74e6fd4d:/# ifconfig
```
So on and so forth. Now lets try to install apache(web server) into our docker for testing, put the following command:
```
root@7b6c74e6fd4d:/# apt install apache2 
```
Once its done, lets start the service:
```
root@7b6c74e6fd4d:/# service apache2 start
```
We can check if the HTTP port open using the command:
```
root@3f55c4081b93:/# netstat -aptn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      3573/apache2      
```
Let's visit the web server via browser from kali linux. As you can see it will run!
Note: you can only visit the ubuntu instance because you are the host, if you want other user to visit the same apache page. You have to run the docker using -p parameter so it mapped to host port.

Now that we have a very minimalistic ubuntu server to play with, lets do network scanning and enumeration using nmap from kali linux.
```
~# nmap 172.17.0.2             
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-07 11:03 EDT
Nmap scan report for 172.17.0.2 (172.17.0.2)
Host is up (0.0000070s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.38 seconds

~# nmap -A 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-07 11:03 EDT
Nmap scan report for 172.17.0.2 (172.17.0.2)
Host is up (0.00013s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.63 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.63 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.12 ms 172.17.0.2 (172.17.0.2)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.43 seconds


```
As you can see using nmap we can get information regarding the open port and the software running in the port. 
Let's right now switch side as the operation team, can we make this little harder for attacker? for example can we hide banner, so attacker don't know what software running in the http port.
Let's go to the apache2.conf file to put some hardening, like this:
```
root@3f55c4081b93:~# cd /etc/apache2
root@3f55c4081b93:/etc/apache2# ls
apache2.conf  conf-available  conf-enabled  envvars  magic  mods-available  mods-enabled  ports.conf  sites-available  sites-enabled
```
Open the file using nano, like this:
```
root@3f55c4081b93:/etc/apache2# nano apache2.conf
```
Go to the end of file using your arrow key and add the following configuration:
```
ServerTokens Prod
ServerSignature Off
```
Don't forget to save it with ctrl+o and ctrl+x to exit the nano. Finally restart apache server like this:
```
root@3f55c4081b93:/etc/apache2# service apache2 restart
```
Once it got up again , try to scan it using nmap:
```
~# nmap -A 172.17.0.2 -p 80
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-07 11:13 EDT
Nmap scan report for 172.17.0.2 (172.17.0.2)
Host is up (0.00015s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache
MAC Address: 02:42:AC:11:00:02 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.15 ms 172.17.0.2 (172.17.0.2)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.01 seconds

```
As you can see the server version is gone. Now let's move on into something more interesting. Lets setup a vulnerable FTP server, in this case we will be using vsftpd.
```
~# apt install vsftpd
```
If the installation asking you about country and location just pick any area you like!

Next is to create ftp user for you to access:
```
root@3f55c4081b93:/etc/apache2/conf-available# useradd -m ftpuser
root@3f55c4081b93:/etc/apache2/conf-available# passwd ftpuser
```
Use any password that you like. Next is to create the following folder:

```
root@3f55c4081b93:~# mkdir -p /var/ftp/share
```
Finally open the vsftpd.conf using nano:
```
root@3f55c4081b93:~# nano /etc/vsftpd.conf
```
Add the following changes:
```
anonymous_enable=YES
local_enable=YES
```
Finally you can start the vsftpd:
```
root@3f55c4081b93:~# service vsftpd start
 * Starting FTP server vsftpd                                                                                                          [ OK ] 
root@3f55c4081b93:~# netstat -aptn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      4157/apache2        
tcp6       0      0 :::21                   :::*                    LISTEN      5091/vsftpd         
```

Now you can nmap the port and see what the nmap pick up:
```
~# nmap 172.17.0.2 -p 21 -A    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-07 11:49 EDT
Nmap scan report for 172.17.0.2 (172.17.0.2)
Host is up (0.00015s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
MAC Address: 02:42:AC:11:00:02 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Unix

TRACEROUTE
HOP RTT     ADDRESS
1   0.15 ms 172.17.0.2 (172.17.0.2)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.46 seconds
```
### Notes
Several docker command that can be useful:

1. For example, if you want to restart your ubuntu docker that has been exited, like this:
```
~# sudo docker container ps -a  
CONTAINER ID   IMAGE                  COMMAND       CREATED             STATUS                          PORTS                                       NAMES
428fce953623   ubuntu:plucky          "/bin/bash"   About an hour ago   Exited (0) About a minute ago                                               eloquent_lamport

```
You can do the following command, first is to start the docker again by providing the container id as in this case it would be 428fce953623:
```
~# sudo docker container start 428fce953623
```
Once it starts you can attach to the docker again, using the following command:
```
~# sudo docker attach 428fce953623 
root@428fce953623:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
2. After this you will doing a lot of setup in docker and it may cause your memory storage, thus, we recommend you to periodically prune the docker. This means to remove all of the cache, docker image and container.
```
~# sudo docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - unused build cache

Are you sure you want to continue? [y/N] y

```
This will remove all the mentioned items in your docker.

### Get your Hands More Dirty
Lets get more serious in the implementation we will try to learn on how to automate build and deploy of the docker environment using dockerfile and docker-compose.yml respectively. Let's build Nginx with the following configurations:
1. Remove version of Nginx
2. Change Nginx port from 80 to 9080
3. Setup HTTPS connection in Nginx 

Quick explanation on dockerfile and docker-compose.yml:
1. Use dockerfile to build your image
2. Use docker-compose yml to deploy your image to container

Let's start with the first two tasks which are removing version from Nginx and Nginx port from 80 to 9080. This two can be done in one go, since we only need to .conf file

According to [Official Documentation](https://nginx.org/en/docs/beginners_guide.html) of nginx, the configuration file is named nginx.conf and placed in the directory /usr/local/nginx/conf, /etc/nginx, or /usr/local/etc/nginx. In my case when you run Nginx docker its located at /etc/nginx

```
root@e2cc91c6cc6b:/etc/nginx# ls -lah
total 44K
drwxr-xr-x 1 root root 4.0K Jul 13 15:36 .
drwxr-xr-x 1 root root 4.0K Jul 13 15:17 ..
drwxr-xr-x 1 root root 4.0K Jul 13 15:42 conf.d
-rw-r--r-- 1 root root 1007 Jun 17 15:15 fastcgi_params
-rw-r--r-- 1 root root 5.3K Jun 17 15:15 mime.types
lrwxrwxrwx 1 root root   22 Jun 17 15:38 modules -> /usr/lib/nginx/modules
-rw-r--r-- 1 root root  644 Jun 17 15:38 nginx.conf
-rw-r--r-- 1 root root  636 Jun 17 15:15 scgi_params
-rw-r--r-- 1 root root  664 Jun 17 15:15 uwsgi_params
```
Take a quick look at the content of nginx.conf file:

```

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
For now it only host 1 http site, in the future we can host many more http site by just adding "server" block in http but as for now we don't need it. Looking at the end of the file the nginx.conf also include configuration on conf.d/ which have only:

```
root@e2cc91c6cc6b:/etc/nginx/conf.d# ls -lah
total 16K
drwxr-xr-x 1 root root 4.0K Jul 13 15:42 .
drwxr-xr-x 1 root root 4.0K Jul 13 15:36 ..
-rw-r--r-- 1 root root 1.1K Jun 17 15:38 default.conf
```
Take a quick look again on default.conf

```
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
Sweet we found port configuration, we will replace it with number 9080 in our dockerfile. What about masking Nginx version? quick google search and reading documentation reveal that all we need to do is just add server_tokens into the http block.

```
Syntax: 	server_tokens on | off | build | string;
Default: 	

server_tokens on;

Context: 	http, server, location

Enables or disables emitting nginx version on error pages and in the “Server” response header field.

The build parameter (1.11.10) enables emitting a build name along with nginx version.

Additionally, as part of our commercial subscription, starting from version 1.9.13 the signature on error pages and the “Server” response header field value can be set explicitly using the string with variables. An empty string disables the emission of the “Server” field. 
```

by doing this two tasks manually and restart nginx. when we doing nmap the service response and port is changed

```
nmap 172.17.0.2 -p 9080 -sV
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-14 18:43 +0700
Nmap scan report for 172.17.0.2 (172.17.0.2)
Host is up (0.000031s latency).

PORT     STATE SERVICE VERSION
9080/tcp open  http    nginx

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.24 seconds
```

Now, that we got all of required logic and files, lets build dockerfile and docker-compose.yml:

1. For dockerfile, you need to make sure you have the hardened default.conf and nginx.conf in one directory with dockerfile. You can check docker-files/01-intro folder to be used for references.

```
FROM nginx:stable-trixie-perl

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf
```

We will filled our dockerfile with this command pretty simple tho, since we only need to copy the two files. No need for running apt-update because it will just bloated the image. Furthermore, we also not need to run CMD command in dockerfile since if you look at the [Image Layer](https://hub.docker.com/layers/library/nginx/stable-trixie-perl/images/sha256-344b23332cf3fbbaec57ffb2d648eca36cbb1ac200ab0230cb8f5f43f2af4392) the last command is 

```
CMD ["nginx" "-g" "daemon off;"]
```

this means it will run nginx web server automatically

Finally create the docker-compose.yml with the following specification

```
name: nginx-server
services:
  web-server-nginx:
    build: ./
    ports:
      - "9080:9080"
```
this docker-compose file will tell docker to build based on the current dockerfile and we want to expose the port 9080 to outsider. Build and run the docker using this two command

```
~# sudo docker compose build
~# sudo docker compose up
```

for compose build command you only need to run it once. If you successfully run it will host the dockerfile, like this:

```
~# sudo docker compose build
[+] Building 3.1s (10/10) FINISHED                                                                                                                                                                                                    
 => [internal] load local bake definitions                                                                                                                                                                                       0.0s
 => => reading from stdin 642B                                                                                                                                                                                                   0.0s
 => [internal] load build definition from Dockerfile                                                                                                                                                                             0.1s
 => => transferring dockerfile: 154B                                                                                                                                                                                             0.0s
 => [internal] load metadata for docker.io/library/nginx:stable-trixie-perl                                                                                                                                                      0.1s
 => [internal] load .dockerignore                                                                                                                                                                                                0.1s
 => => transferring context: 2B                                                                                                                                                                                                  0.0s
 => [1/3] FROM docker.io/library/nginx:stable-trixie-perl@sha256:ba77770cb5d1868226a6a67e5b3c4d5bbc1e6116e2c71e51075863bbaf13e828                                                                                                0.7s
 => => resolve docker.io/library/nginx:stable-trixie-perl@sha256:ba77770cb5d1868226a6a67e5b3c4d5bbc1e6116e2c71e51075863bbaf13e828                                                                                                0.1s
 => [internal] load build context                                                                                                                                                                                                0.1s
 => => transferring context: 1.82kB                                                                                                                                                                                              0.0s
 => [2/3] COPY nginx.conf /etc/nginx/nginx.conf                                                                                                                                                                                  0.1s
 => [3/3] COPY default.conf /etc/nginx/conf.d/default.conf                                                                                                                                                                       0.1s
 => exporting to image                                                                                                                                                                                                           1.0s
 => => exporting layers                                                                                                                                                                                                          0.4s
 => => exporting manifest sha256:763f299990b150eb980cf1a94a73f129616adb2330162005b1ea3f68545a1083                                                                                                                                0.1s
 => => exporting config sha256:dba753af989df45c92137300bc1bd46c57c535ce04d7388f65143f926d7f1770                                                                                                                                  0.0s
 => => exporting attestation manifest sha256:6032a7a5edc5b346dad8f88a792294d9d9b56671885196cb4340cdff050a760b                                                                                                                    0.1s
 => => exporting manifest list sha256:a4a5ca8e6bb175adca95542fb68ed1dd41218aa7bcbebe4b28056bb01ca4a2c2                                                                                                                           0.1s
 => => naming to docker.io/library/nginx-server-web-server-nginx:latest                                                                                                                                                          0.0s
 => => unpacking to docker.io/library/nginx-server-web-server-nginx:latest                                                                                                                                                       0.1s
 => resolving provenance for metadata file                                                                                                                                                                                       0.0s
[+] build 1/1
 ✔ Image nginx-server-web-server-nginx Built                            
```

```
sudo docker compose up
[+] up 2/2
 ✔ Network nginx-server_default              Created                                                                                                                                                                              0.1s
 ✔ Container nginx-server-web-server-nginx-1 Created                                                                                                                                                                              0.2s
Attaching to web-server-nginx-1
web-server-nginx-1  | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
web-server-nginx-1  | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
web-server-nginx-1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
web-server-nginx-1  | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
web-server-nginx-1  | 10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
web-server-nginx-1  | /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
web-server-nginx-1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
web-server-nginx-1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
web-server-nginx-1  | /docker-entrypoint.sh: Configuration complete; ready for start up
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: using the "epoll" event method
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: nginx/1.30.3
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19) 
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: OS: Linux 7.0.0-27-generic
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:524288
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: start worker processes
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: start worker process 28
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: start worker process 29
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: start worker process 30
web-server-nginx-1  | 2026/07/14 12:20:35 [notice] 1#1: start worker process 31

```

if you want to make run in background just add -d parameter.

Finally let's enable SSL into nginx!

First step is to create public key and private key using openssl like this:

```
~# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./nginx-selfsigned.key -out ./nginx-selfsigned.crt
....+...............+...........+.+.....+....+......+.....+...+.+..............+...+...+.+...+...+...............+++++++++++++++++++++++++++++++++++++++*.+..+.......+........+......+....+......+..+...+.+...+..+...+.+.....+......+......+....+......+..+............+.+...+..+.......+..+..........+.....+.........+....+...+........+....+++++++++++++++++++++++++++++++++++++++*......+......................+..................+..+.+............+..+.............+.....+...+......+....+..+.......+.....+.+......+........+......+.+........+.+..+....+.....+.+........+.+...............+......+............+...............+.....+............+...+.+............+..+...+..................+.+...........+.......+.....+......+.......+...+...........+...+.+.........+..+....+..+..........+...+..+.............+.....+.+.....+.+...+..+.........+.............+..+.+.....+............+...+.........+.........+....+...+.................+......+......+.+..+...+...+............+.............+.....+....+...+..+..................+....+.........+......+...+...........+.+........+.+...........+......+...+......+.+............+........+.......+..+....+......+...+........+...+....+...+..+...............+.+.....................+.....+.............++++++
..........+........+...+...+......+.+.................+++++++++++++++++++++++++++++++++++++++*......+.....+......+....+...+............+.....+....+..+..........+..............+.........+.......+..+......+.+...+...........+.+.....+.+...+..+.......+.....+...+...+............+...+......+.+...+......+.....+...+......+....+++++++++++++++++++++++++++++++++++++++*......+.......+...............+..+.+..+.+...............+.........+...+..++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ID
State or Province Name (full name) [Some-State]:Jakarta
Locality Name (eg, city) []:Jakarta
Organization Name (eg, company) [Internet Widgits Pty Ltd]:blackhat
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:blackhat
Email Address []:IT@blackhat.com
root@e2cc91c6cc6b:/etc/nginx/ssl# ls
nginx-selfsigned.crt  nginx-selfsigned.key
```

Next is to add some changes into default.conf so its load the ssl certs, like this:
```
server {
    listen       9080 ssl;
    server_name  blackhat.com;
    ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    ....
}
```

Its pretty easy changes, all you have to do is just add ssl after the port and add ssl_certificate and ssl_certificate_key to refer it to the ssl certs, you can use any location but make sure its consistent. Finally, the last touch is to adjust the dockerfile.

```
FROM nginx:stable-trixie-perl

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf
COPY nginx-selfsigned.crt /etc/nginx/ssl/nginx-selfsigned.crt
COPY nginx-selfsigned.key /etc/nginx/ssl/nginx-selfsigned.key

RUN chmod 644 /etc/nginx/ssl/nginx-selfsigned.crt && chmod 600 /etc/nginx/ssl/nginx-selfsigned.key
```

we copy the public key and private key to the docker and change the mod of the file to be more restrictive. Remove the old docker image from previous activity and run the sudo docker compose build and up again. You now have setup a https nginx with basic hardening configuration.