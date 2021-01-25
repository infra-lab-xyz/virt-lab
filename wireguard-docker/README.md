### Wireguard setup and route manage

Demo of securing connection between two hosts with docker containers that can be accessible from both hosts

```
It is kind of legacy pattern. Be cause it is hard to keep up to date list of IP addreses and subnets
Also, there is no access by FQDN only by IP.
``` 

### Useful docs

* [Arch wiki: wireguard](https://wiki.archlinux.org/index.php/WireGuard)
* [Wireguard official site](https://www.wireguard.com/netns/)

### How to use

* Execute `vagrant up`
* Check if wireguard enabled `lsmod | grep wireguard`

    ```
    lsmod | grep wireguard
    wireguard             212992  0
    ip6_udp_tunnel         16384  1 wireguard
    udp_tunnel             16384  1 wireguard
    ```

* Enable wireguard module if not enabled `sudo modprobe wireguard`
* Write config from examples

    | Host | Config | 
    | --- | --- |
    | PeerA | [/etc/wireguard/wg0.conf](wireguard-docker/files/peer-a/etc/wireguard/wg0.conf) |
    | PeerB | [/etc/wireguard/wg0.conf](wireguard-docker/files/peer-b/etc/wireguard/wg0.conf) |

* Enable connection on both machines `wg-quick up wg0`
* Check connection status `wg show`

    ```
    # wg show
    interface: wg0
      public key: bux10xMgV/5htpNL1ctJXKwG/3xpidPlAwvaZ1+BLHY=
      private key: (hidden)
      listening port: 51820
    
    peer: PiySe7E0At4O1/SGg9Xo7i2yNqVlwtIjDrlo1SMA8zg=
      endpoint: 172.28.128.4:51820
      allowed ips: 10.200.0.2/32, 192.168.128.0/19
      latest handshake: 8 seconds ago
      transfer: 3.68 KiB received, 11.23 KiB sent
      persistent keepalive: every 25 seconds
    ```

* Check connection
    * PeerA `nc -l 10.200.0.1 8000`
    * PeerB `nc -vzw 2 10.200.0.1 8000`

* Put docker-compose.yml and .env on server

    | Host | Config | Env file |
    |---|---|---|
    | PeerA | [/opt/docker-compose.yml](wireguard-docker/files/docker-compose.yml) | [/opt/.env](wireguard-docker/files/peer-a/.env) |
    | PeerB | [/opt/docker-compose.yml](wireguard-docker/files/docker-compose.yml) | [/opt/.env](wireguard-docker/files/peer-b/.env) |

* Launch balancers `docker-compose up -d`
* Check connections

    | Host | Command |
    |---|---|
    | PeerA | `curl --connect-timeout 2 -I http://192.168.96.2` |
    | PeerB | `curl --connect-timeout 2 -I http://192.168.128.2` |

    ```
    curl --connect-timeout 2 -I http://192.168.128.2
    HTTP/1.1 200 OK
    Server: nginx/1.19.6
    Date: Mon, 25 Jan 2021 01:27:11 GMT
    Content-Type: text/html
    Content-Length: 612
    Last-Modified: Tue, 15 Dec 2020 13:59:38 GMT
    Connection: keep-alive
    ETag: "5fd8c14a-264"
    Accept-Ranges: bytes
    ```

### TODO

* Automate steps via Terraform or Ansible (Move provision from Vagrantfile)
* Make sure VPN launches after reboot
* Make sure VPN connects after network failure between hosts
* Integrate Consul DNS
* Automate service discovery with Consul
* Check [Consul connect](https://www.consul.io/docs/connect/connect-internals)
