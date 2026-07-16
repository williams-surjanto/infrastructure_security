# Introduction to Docker

> **Note:** This repository assumes you are using Docker on Kali Linux or a Linux machine with an Intel architecture, along with some familiarity with Linux. If you use an M1/M2/M3 chip, you'll have to find your own way to adjust the labs.

But wait — what's Intel arch? M1? M2? M3? Kali Linux? If you can't answer these basic questions, it's better to start with more fundamental material first. Don't be lazy! The internet has plenty of good, free reading that covers the basics of cyber security — use your brain! If you can't handle these simple things, you'll certainly be replaced by AI in no time.

**References:**

- *Docker Deep Dive: Zero to Docker in a Single Book*, by Nigel Poulton
- Nginx Documentation
- Google

## Why Docker?

| Point | Detail |
|:-----:|--------|
| 1 | In the past, one app equaled one server. |
| 2 | New business ⇒ new app ⇒ new server ⇒ cost goes up. |
| 3 | Then came VMware, to solve this problem. |
| 4 | But it introduced a new problem — it's slow to boot and not portable. |

Then containers came along! They take the same approach as VMs, but they don't require a full-blown OS — so they're portable and easy to set up.

## Installing Docker

To install Docker, open a terminal and use the following commands:

```bash
~# sudo apt install docker.io
~# sudo apt install docker-compose
```

Once the installation is complete, you can check that Docker was installed correctly:

```bash
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

Be aware that your result might be different, but that's not going to be a problem for now.

## Get Your Hands Dirty

Run the following command in the terminal:

```bash
~# sudo docker images           
REPOSITORY                 TAG       IMAGE ID       CREATED       SIZE
```

To use Docker, you need to use `sudo`! As the name implies, this shows your Docker images. An **image** basically contains all the necessary components, including dependencies, to run an application.

Okay, now let's try to download our first image. Run the following command in the terminal:

```bash
~# sudo docker pull ubuntu:plucky
plucky: Pulling from library/ubuntu
60fb2420030a: Pull complete 
Digest: sha256:95a416ad2446813278ec13b7efdeb551190c94e12028707dd7525632d3cec0d1
Status: Downloaded newer image for ubuntu:plucky
docker.io/library/ubuntu:plucky
```

Once the download is done, you can check again by entering this command:

```bash
~# sudo docker images                           
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       plucky    92598a7a70c7   3 weeks ago   77MB
```

Let's create the container (think of the image as the raw material and the container as the product made from that raw material). Now that the container is ready, it's time to start it:

```bash
~# sudo docker run -it ubuntu:plucky /bin/bash   
root@7b6c74e6fd4d:/# 
```

The container is a very minimalistic environment, so we need to do some installation. Run the following commands inside the container:

```bash
root@7b6c74e6fd4d:/# apt update
root@7b6c74e6fd4d:/# apt install net-tools
root@7b6c74e6fd4d:/# apt install nano
```

Once the installation is complete, we can use the standard Linux network CLI tools, like:

```bash
root@7b6c74e6fd4d:/# netstat -aptn
root@7b6c74e6fd4d:/# ifconfig
```

And so on and so forth. Now let's try to install Apache (a web server) into our container for testing. Enter the following command:

```bash
root@7b6c74e6fd4d:/# apt install apache2 
```

Once it's done, let's start the service:

```bash
root@7b6c74e6fd4d:/# service apache2 start
```

We can check whether the HTTP port is open using the command:

```bash
root@3f55c4081b93:/# netstat -aptn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      3573/apache2      
```

Let's visit the web server via a browser from Kali Linux. As you can see, it works!

> **Note:** You can only visit the Ubuntu instance because you are the host. If you want another user to visit the same Apache page, you have to run the container with the `-p` parameter so it maps to a host port.

Now that we have a very minimalistic Ubuntu server to play with, let's do some network scanning and enumeration using `nmap` from Kali Linux.

```bash
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

As you can see, with `nmap` we can get information about the open ports and the software running on them.

Let's now switch sides and act as the operations team. Can we make this a little harder for an attacker? For example, can we hide the banner so the attacker doesn't know what software is running on the HTTP port? Let's go to the `apache2.conf` file to apply some hardening:

```bash
root@3f55c4081b93:~# cd /etc/apache2
root@3f55c4081b93:/etc/apache2# ls
apache2.conf  conf-available  conf-enabled  envvars  magic  mods-available  mods-enabled  ports.conf  sites-available  sites-enabled
```

Open the file using nano:

```bash
root@3f55c4081b93:/etc/apache2# nano apache2.conf
```

Go to the end of the file using your arrow keys and add the following configuration:

```apache
ServerTokens Prod
ServerSignature Off
```

Don't forget to save it with `Ctrl+O`, then `Ctrl+X` to exit nano. Finally, restart the Apache server:

```bash
root@3f55c4081b93:/etc/apache2# service apache2 restart
```

Once it's back up, try scanning it again with nmap:

```bash
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

As you can see, the server version is gone. Now let's move on to something more interesting. Let's set up a vulnerable FTP server — in this case we'll use vsftpd.

```bash
~# apt install vsftpd
```

If the installation asks you about country and location, just pick any area you like.

Next, create an FTP user for you to access:

```bash
root@3f55c4081b93:/etc/apache2/conf-available# useradd -m ftpuser
root@3f55c4081b93:/etc/apache2/conf-available# passwd ftpuser
```

Use any password you like. Next, create the following folder:

```bash
root@3f55c4081b93:~# mkdir -p /var/ftp/share
```

Finally, open `vsftpd.conf` using nano:

```bash
root@3f55c4081b93:~# nano /etc/vsftpd.conf
```

Add the following changes:

```ini
anonymous_enable=YES
local_enable=YES
```

Finally, you can start vsftpd:

```bash
root@3f55c4081b93:~# service vsftpd start
 * Starting FTP server vsftpd                                                                                                          [ OK ] 
root@3f55c4081b93:~# netstat -aptn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      4157/apache2        
tcp6       0      0 :::21                   :::*                    LISTEN      5091/vsftpd         
```

Now you can nmap the port and see what nmap picks up:

```bash
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

## Notes

Several Docker commands that can be useful:

**1. Restarting an exited container.** For example, if you want to restart your Ubuntu container that has been exited:

```bash
~# sudo docker container ps -a  
CONTAINER ID   IMAGE                  COMMAND       CREATED             STATUS                          PORTS                                       NAMES
428fce953623   ubuntu:plucky          "/bin/bash"   About an hour ago   Exited (0) About a minute ago                                               eloquent_lamport
```

First, start the container again by providing the container ID — in this case, `428fce953623`:

```bash
~# sudo docker container start 428fce953623
```

Once it starts, you can attach to it again with the following command:

```bash
~# sudo docker attach 428fce953623 
root@428fce953623:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

**2. Pruning Docker.** After this you'll be doing a lot of setup in Docker, and it may fill up your storage. We recommend periodically pruning Docker — this removes all cache, dangling images, and stopped containers.

```bash
~# sudo docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - unused build cache

Are you sure you want to continue? [y/N] y
```

This will remove all of the mentioned items from your Docker environment.

## Get Your Hands More Dirty

Let's get more serious about the implementation. We'll learn how to automate building and deploying a Docker environment using a Dockerfile and docker-compose.yml, respectively. Let's build Nginx with the following configurations:

1. Remove the Nginx version from the banner.
2. Change the Nginx port from 80 to 9080.
3. Set up an HTTPS connection in Nginx.

Quick explanation of the Dockerfile and docker-compose.yml:

1. Use the **Dockerfile** to build your image.
2. Use **docker-compose.yml** to deploy your image to a container.

Let's start with the first two tasks: removing the version from Nginx and changing the Nginx port from 80 to 9080. These two can be done in one go, since we only need the `.conf` file.

According to the [official Nginx documentation](https://nginx.org/en/docs/beginners_guide.html), the configuration file is named `nginx.conf` and placed in the directory `/usr/local/nginx/conf`, `/etc/nginx`, or `/usr/local/etc/nginx`. In my case, when you run the Nginx container it's located at `/etc/nginx`:

```bash
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

Take a quick look at the contents of the `nginx.conf` file:

```nginx
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

For now it only hosts one HTTP site. In the future we can host many more HTTP sites by adding more `server` blocks inside `http`, but for now we don't need to. Looking at the end of the file, `nginx.conf` also includes the configuration in `conf.d/`, which contains only:

```bash
root@e2cc91c6cc6b:/etc/nginx/conf.d# ls -lah
total 16K
drwxr-xr-x 1 root root 4.0K Jul 13 15:42 .
drwxr-xr-x 1 root root 4.0K Jul 13 15:36 ..
-rw-r--r-- 1 root root 1.1K Jun 17 15:38 default.conf
```

Take a quick look at `default.conf`:

```nginx
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

Sweet — we found the port configuration, which we'll replace with `9080` in our Dockerfile. What about masking the Nginx version? A quick Google search and some documentation reading reveal that all we need to do is add `server_tokens` to the `http` block:

```text
Syntax: 	server_tokens on | off | build | string;
Default: 	

server_tokens on;

Context: 	http, server, location

Enables or disables emitting nginx version on error pages and in the “Server” response header field.

The build parameter (1.11.10) enables emitting a build name along with nginx version.

Additionally, as part of our commercial subscription, starting from version 1.9.13 the signature on error pages and the “Server” response header field value can be set explicitly using the string with variables. An empty string disables the emission of the “Server” field. 
```

By doing these two tasks manually and restarting Nginx, when we run nmap the service response and port have changed:

```bash
nmap 172.17.0.2 -p 9080 -sV
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-14 18:43 +0700
Nmap scan report for 172.17.0.2 (172.17.0.2)
Host is up (0.000031s latency).

PORT     STATE SERVICE VERSION
9080/tcp open  http    nginx

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.24 seconds
```

Now that we have all of the required logic and files, let's build the Dockerfile and docker-compose.yml.

**1. Dockerfile.** Make sure you have the hardened `default.conf` and `nginx.conf` in the same directory as the Dockerfile. You can check the `docker-files/01-intro` folder for reference.

```dockerfile
FROM nginx:stable-trixie-perl

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf
```

We fill our Dockerfile with this command — pretty simple, since we only need to copy the two files. There's no need to run `apt-update`, because it would just bloat the image. Furthermore, we also don't need to run a `CMD` command in the Dockerfile, because if you look at the [image layers](https://hub.docker.com/layers/library/nginx/stable-trixie-perl/images/sha256-344b23332cf3fbbaec57ffb2d648eca36cbb1ac200ab0230cb8f5f43f2af4392), the last command is:

```dockerfile
CMD ["nginx" "-g" "daemon off;"]
```

This means it will run the Nginx web server automatically.

Finally, create the docker-compose.yml with the following specification:

```yaml
name: nginx-server
services:
  web-server-nginx:
    build: ./
    ports:
      - "9080:9080"
```

This docker-compose file tells Docker to build based on the current Dockerfile, and we want to expose port 9080 to the outside. Build and run the container using these two commands:

```bash
~# sudo docker compose build
~# sudo docker compose up
```

For the `compose build` command, you only need to run it once. If it runs successfully, it will host the Dockerfile, like this:

```bash
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

```bash
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

If you want to run it in the background, just add the `-d` parameter.

Finally, let's enable SSL in Nginx!

The first step is to create a public key and private key using OpenSSL:

```bash
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

Next, add some changes to `default.conf` so it loads the SSL certs:

```nginx
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

These are pretty easy changes — all you have to do is add `ssl` after the port and add `ssl_certificate` and `ssl_certificate_key` to point to the SSL certs. You can use any location, but make sure it's consistent. Finally, the last touch is to adjust the Dockerfile:

```dockerfile
FROM nginx:stable-trixie-perl

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf
COPY nginx-selfsigned.crt /etc/nginx/ssl/nginx-selfsigned.crt
COPY nginx-selfsigned.key /etc/nginx/ssl/nginx-selfsigned.key

RUN chmod 644 /etc/nginx/ssl/nginx-selfsigned.crt && chmod 600 /etc/nginx/ssl/nginx-selfsigned.key
```

We copy the public key and private key into the container and change the file permissions to be more restrictive. Remove the old Docker image from the previous activity, then run `sudo docker compose build` and `up` again. You now have a HTTPS Nginx setup with basic hardening configuration.
