# Docker Network

Now that you have a good understanding of how Docker works, it's time to learn some of the fundamentals of how networking is done in Docker. Keep in mind that this file is **not** an exhaustive list of how to configure Docker networking, so please read and check the official documentation yourself.

**References:**

1. [Network Chunk – Docker Network](https://www.youtube.com/watch?v=bKFMS5C4CG0)
2. [Official Documentation – Docker Network](https://docs.docker.com/engine/network/)
3. [Understanding NAT](https://medium.com/@aditya.raut24/understanding-network-address-translation-nat-ae9b64c2e923)

## The Default Bridge Network

If you want to get a similar result as me, you first need to run `docker system prune` to remove the network interfaces created by previous labs, then run `ifconfig`:

```text
~# ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::c897:feff:fe4c:9dcb  prefixlen 64  scopeid 0x20<link>
        ether ca:97:fe:4c:9d:cb  txqueuelen 0  (Ethernet)
        RX packets 1834  bytes 115963 (115.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3197  bytes 11634099 (11.6 MB)
        TX errors 0  dropped 468 overruns 0  carrier 0  collisions 0

enp2s0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 18:66:da:26:64:69  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 120048  bytes 13007312 (13.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 120048  bytes 13007312 (13.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

As you can see, we have `docker0` — what is it? It's called the **default bridge**. By default, Docker containers are attached to this built-in network interface, which has access to network services outside the Docker host. So with that explained, every container you have created so far has been connected to this interface, and this is also why we can install packages and pull updates from remote repositories inside our containers.

We can deep dive into the default bridge by spinning up two containers, one Linux and one nginx:

```bash
~# sudo docker images
IMAGE                      ID             DISK USAGE   CONTENT SIZE   EXTRA
nginx:stable-trixie-perl   ba77770cb5d1        311MB         79.5MB    U   
ubuntu:plucky              27771fb7b40a        118MB         31.6MB    U   
~$ sudo docker run -itd --name linux-host ubuntu:plucky 
da47f01106b2b08d8af39bc885835c347ce3101a7de4b4cf90d936e9d9a0410a
~$ sudo docker run -itd --name nginx-host nginx:stable-trixie-perl 
2bba4764bbd387bc824d5ae3ba0e090e566bb2b2b9964ad7f1504e755d6113ff
~$ sudo docker container ls 
CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS              PORTS     NAMES
2bba4764bbd3   nginx:stable-trixie-perl   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    nginx-host
da47f01106b2   ubuntu:plucky              "/bin/bash"              About a minute ago   Up About a minute             linux-host
```

Here we add the `-d` flag so we can detach from the container after creating it and run it in the background. Check the IPs again using `ifconfig` and `bridge link`:

```text
~# ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::c897:feff:fe4c:9dcb  prefixlen 64  scopeid 0x20<link>
        ether ca:97:fe:4c:9d:cb  txqueuelen 0  (Ethernet)
        RX packets 1840  bytes 116131 (116.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3215  bytes 11648063 (11.6 MB)
        TX errors 0  dropped 478 overruns 0  carrier 0  collisions 0

....
veth0636a58: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::1807:f1ff:fe87:b11b  prefixlen 64  scopeid 0x20<link>
        ether 9e:af:b1:36:13:85  txqueuelen 0  (Ethernet)
        RX packets 3  bytes 126 (126.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 60  bytes 28540 (28.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethdcacc06: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::80a6:abff:fe8f:7c90  prefixlen 64  scopeid 0x20<link>
        ether 82:a6:ab:8f:7c:90  txqueuelen 0  (Ethernet)
        RX packets 3  bytes 126 (126.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 43  bytes 17826 (17.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
~# bridge link
20: veth0636a58@enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master docker0 state forwarding priority 32 cost 2 
21: vethdcacc06@enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master docker0 state forwarding priority 32 cost 2 
```

You'll notice we now have two new virtual interfaces connected to the default bridge `docker0`, and each has been assigned its own IP address. You can confirm this with the `inspect` command:

```bash
~# sudo docker inspect bridge
.....
"Containers": {
            "2bba4764bbd387bc824d5ae3ba0e090e566bb2b2b9964ad7f1504e755d6113ff": {
                "Name": "nginx-host",
                "EndpointID": "bd2163f39c66c03d86b72a01c015775d04a14817954c1d67311bf11f22fc3e49",
                "MacAddress": "72:64:39:42:89:10",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "da47f01106b2b08d8af39bc885835c347ce3101a7de4b4cf90d936e9d9a0410a": {
                "Name": "linux-host",
                "EndpointID": "921abaaeab7a212a5cd782c16e001e55eff68806ea96d3d7a5b7d05bafe47b19",
                "MacAddress": "d2:07:ec:5c:e4:96",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        }
.....
```

### How Docker Reaches the Outside World: NAT

The way Docker connects its containers to the outside world is through a mechanism called **NAT (Network Address Translation)**. Basically, it's a method to translate a private IP address (home Wi-Fi) into a public IP address (internet) and vice versa. The way NAT works is really simple, and we can summarize it in three easy steps:

1. When sending a packet to its destination, the router replaces the device's private IP address with the router's public IP.
2. It records the connection in a **NAT table**, mapping the internal IP address to the external IP and port.
3. When the response comes back, the router uses the NAT table to forward it to the correct internal device.

To make this concrete, let's map those three steps onto the exact setup we just built. Our `linux-host` (`172.17.0.2`) wants to reach a server out on the internet, but `172.17.0.x` is a private range that the internet can't route back to. The Docker host performs NAT (via iptables masquerading) on the way out, and reverses it on the way back in:

```text
        Docker Host (172.17.0.1 / enp2s0: 203.0.113.10)
   ┌───────────────────────────────────────────────────────────┐
   │                                                           │
   │   ┌───────────────┐        docker0 bridge                 │
   │   │  linux-host   │        172.17.0.1                     │
   │   │  172.17.0.2   │──veth──┐                              │
   │   └───────────────┘        │      ┌──────────────────────┐│
   │                            ├──────│  NAT / iptables      ││
   │   ┌───────────────┐        │      │  (MASQUERADE)        ││
   │   │  nginx-host   │──veth──┘      └──────────┬───────────┘│
   │   │  172.17.0.3   │                          │            │
   │   └───────────────┘                     enp2s0 (physical) │
   │                                         203.0.113.10      │
   └───────────────────────────────────────────────┼───────────┘
                                                   │
                                                   ▼
                                             ┌───────────┐
                                             │  Internet │
                                             │ 93.184.x  │
                                             └───────────┘


  OUTBOUND  ── linux-host sends a packet to 93.184.216.34:443
  ─────────────────────────────────────────────────────────────

  ┌──────────────────────────┐   ┌──────────────────────────┐
  │ src: 172.17.0.2:52418    │   │ src: 203.0.113.10:61000  │  ← ① src rewritten
  │ dst: 93.184.216.34:443   │ ▶ │ dst: 93.184.216.34:443   │     to host public IP
  └──────────────────────────┘   └──────────────────────────┘
        (leaves container)              (leaves enp2s0)

                        ② mapping saved in NAT table
        ┌───────────────────────────────────────────────────┐
        │ 203.0.113.10:61000  <->  172.17.0.2:52418         │
        └───────────────────────────────────────────────────┘


  INBOUND   ── reply comes back from 93.184.216.34:443
  ─────────────────────────────────────────────────────────────

  ┌──────────────────────────┐   ┌──────────────────────────┐
  │ src: 93.184.216.34:443   │   │ src: 93.184.216.34:443   │
  │ dst: 203.0.113.10:61000  │ ▶ │ dst: 172.17.0.2:52418    │  ← ③ NAT table used to
  └──────────────────────────┘   └──────────────────────────┘     restore real dst
        (arrives at enp2s0)           (delivered to container)
```

Notice the port `52418` in the example — this is the key that makes it work. If both `linux-host` and `nginx-host` reach out at the same time, the host distinguishes their two connections by the source port it recorded in the NAT table, so replies never get delivered to the wrong container. This is why the mechanism is more precisely called **PAT (Port Address Translation)**, or "NAT overload": many private addresses hiding behind a single public IP.

## The User-Defined Network

This type of network uses the same mechanism as the default bridge, but the catch is that we create our own bridged network. Let's create a new network with this command:

```bash
~# sudo docker network create home-lab-01
[sudo: authenticate] Password:            
9f5e8070351dc6039bc8de0bde7f941e7a0fa1ce669373c0a1d0e0746f1214f6
~# sudo docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
3b2aeca3fa0f   bridge        bridge    local
9f5e8070351d   home-lab-01   bridge    local
3c83947eac33   host          host      local
0f12908211d3   none          null      local
```

As you can see, the new network has the same type as the default network. Checking with `ifconfig` shows a new bridge adapter with a new subnet:

```text
~# ifconfig
br-9f5e8070351d: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether ee:a8:cb:06:27:93  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::c897:feff:fe4c:9dcb  prefixlen 64  scopeid 0x20<link>
        ether ca:97:fe:4c:9d:cb  txqueuelen 0  (Ethernet)
        RX packets 1840  bytes 116131 (116.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3505  bytes 11864537 (11.8 MB)
        TX errors 0  dropped 478 overruns 0  carrier 0  collisions 0
```

Now that you have the new network, you can spin up a container in it like this:

```bash
~# sudo docker run -itd --rm --network home-lab-01 --name linux-orphan busybox
latest: Pulling from library/busybox
b05093807bb0: Pull complete 
7270b3e1860c: Download complete 
Digest: sha256:fd8d9aa63ba2f0982b5304e1ee8d3b90a210bc1ffb5314d980eb6962f1a9715d
Status: Downloaded newer image for busybox:latest
c62942806545c92c6c730b03710fc3c647c375b529a6be14fb1146cc557d268e
~# $ sudo docker container ls 
CONTAINER ID   IMAGE                      COMMAND                  CREATED          STATUS          PORTS     NAMES
c62942806545   busybox                    "sh"                     45 seconds ago   Up 45 seconds             linux-orphan
2bba4764bbd3   nginx:stable-trixie-perl   "/docker-entrypoint.…"   24 hours ago     Up 24 hours     80/tcp    nginx-host
da47f01106b2   ubuntu:plucky              "/bin/bash"              24 hours ago     Up 24 hours               linux-host
~# bridge link
20: veth0636a58@enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master docker0 state forwarding priority 32 cost 2 
21: vethdcacc06@enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master docker0 state forwarding priority 32 cost 2 
23: vethbe10318@enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br-9f5e8070351d state forwarding priority 32 cost 2 
```

Once that's done, Docker automatically creates a virtual ethernet interface for your container. However, with these settings, containers on different subnets **cannot** talk to each other — so if you try to ping `linux-orphan` from `linux-host`, it won't work. To fix this, we need to explicitly connect the container to the other bridge network:

```bash
~# sudo docker network connect home-lab-01 linux-host
~# sudo docker inspect home-lab-01
....
"Containers": {
            "c62942806545c92c6c730b03710fc3c647c375b529a6be14fb1146cc557d268e": {
                "Name": "linux-orphan",
                "EndpointID": "e2b6335498fd3937efb2943bf772da2b3ac762590387114f6fccb5ddd8991b1d",
                "MacAddress": "72:bf:9d:64:4e:d9",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "da47f01106b2b08d8af39bc885835c347ce3101a7de4b4cf90d936e9d9a0410a": {
                "Name": "linux-host",
                "EndpointID": "5f8edddaf197a60598455dd7fc7bc47030aa4a557fa8a27708b9ea96a7c20786",
                "MacAddress": "62:e8:c8:b9:1a:b8",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        }
....
```

The result shows that `linux-host` now has two virtual ethernet interfaces. You'll see the same result if you attach to the container:

```bash
~# sudo docker container ls
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS     NAMES
c62942806545   busybox                    "sh"                     6 minutes ago   Up 6 minutes             linux-orphan
2bba4764bbd3   nginx:stable-trixie-perl   "/docker-entrypoint.…"   24 hours ago    Up 24 hours    80/tcp    nginx-host
da47f01106b2   ubuntu:plucky              "/bin/bash"              24 hours ago    Up 24 hours              linux-host
~# sudo docker container attach da47f01106b2
root@da47f01106b2:/# ifconfig
bash: ifconfig: command not found
root@da47f01106b2:/# apt update        
9 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@da47f01106b2:/# apt install net-tools
Installing:                     
  net-tools

Summary:
  Upgrading: 0, Installing: 1, Removing: 0, Not Upgrading: 9
  Download size: 210 kB
  Space needed: 848 kB / 452 GB available

Get:1 http://archive.ubuntu.com/ubuntu plucky-updates/main amd64 net-tools amd64 2.10-1.1ubuntu1.25.04.4 [210 kB]
Unpacking net-tools (2.10-1.1ubuntu1.25.04.4) ...
Setting up net-tools (2.10-1.1ubuntu1.25.04.4) ...
root@da47f01106b2:/# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether d2:07:ec:5c:e4:96  txqueuelen 0  (Ethernet)
        RX packets 13786  bytes 26739161 (26.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10727  bytes 992185 (992.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.3  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 62:e8:c8:b9:1a:b8  txqueuelen 0  (Ethernet)
        RX packets 46  bytes 18302 (18.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3  bytes 126 (126.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...
```

At this stage, you can send a connection from `linux-host` to `linux-orphan` (`172.18.0.2`) using `ping`:

```bash
root@da47f01106b2:/# apt install iputils-ping
Installing:                     
  iputils-ping

Installing dependencies:
  libidn2-0  libunistring5  linux-sysctl-defaults

Summary:
  Upgrading: 0, Installing: 4, Removing: 0, Not Upgrading: 9
  Download size: 735 kB
  Space needed: 2601 kB / 452 GB available

Continue? [Y/n] y
....
root@da47f01106b2:/# ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.235 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.097 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.130 ms
^C
--- 172.18.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2072ms
rtt min/avg/max/mdev = 0.097/0.154/0.235/0.058 ms
```

Let's visualize what we just built. We now have **two separate bridge networks**, each its own subnet, living side by side on the same Docker host. The important idea is that each user-defined bridge is an isolated segment: `nginx-host` on `docker0` and `linux-orphan` on `home-lab-01` have no path to each other by default. The only reason `linux-host` can reach `linux-orphan` is that we gave it a second foot (`eth1`) inside `home-lab-01`:

```text
                      Docker Host (172.17.0.1 / 172.18.0.1)
 ┌───────────────────────────────────────────────────────────────────────────┐
 │                                                                           │
 │    docker0  (default bridge)              br-9f5e8070351d  (home-lab-01)  │
 │    172.17.0.0/16                          172.18.0.0/16                   │
 │  ┌────────────────────────────┐         ┌────────────────────────────┐    │
 │  │                            │         │                            │    │
 │  │   ┌────────────────┐       │    ✗    │       ┌────────────────┐   │    │
 │  │   │   nginx-host   │       │ ◀─────▶ │       │  linux-orphan  │   │    │
 │  │   │   172.17.0.3   │       │ blocked │       │   172.18.0.2   │   │    │
 │  │   └────────────────┘       │ (isolated)      └────────────────┘   │    │
 │  │                            │         │                            │    │
 │  │   ┌────────────────┐       │         │   ┌────────────────┐       │    │
 │  │   │   linux-host   │       │         │   │   linux-host   │       │    │
 │  │   │ eth0:172.17.0.2│       │         │   │ eth1:172.18.0.3│       │    │
 │  │   └──────┬─────────┘       │         │   └───────┬────────┘       │    │
 │  └──────────┼─────────────────┘         └───────────┼────────────────┘    │
 │             │                                       │                     │
 │             └──────────── same container ───────────┘                     │
 │              (dual-homed after `docker network connect`)                  │
 │                                                                           │
 └───────────────────────────────────────────────────────────────────────────┘
```

So `linux-host` is now **dual-homed**: it has one interface (`eth0`) on `docker0` and another (`eth1`) on `home-lab-01`, exactly as the `ifconfig` output above showed. Because it has a leg in each subnet, it can talk to `nginx-host` on one side *and* `linux-orphan` on the other. Neither of those two can reach across to each other, though — the isolation between the bridges still holds. This is the same trick a real network uses: a host with interfaces in two subnets sits on the boundary between them, and it is exactly this property (a machine bridging two otherwise-isolated segments) that matters when we start thinking about **network segmentation and lateral movement** later on.
