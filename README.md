[![wireguard](https://www.wireguard.com/img/wireguard.svg)](https://www.wireguard.com/)



# What is WireGuard?

[WireGuard®](https://www.wireguard.com/) is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more useful than IPsec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances. Initially released for the Linux kernel, it is now cross-platform (Windows, macOS, BSD, iOS, Android) and widely deployable. It is currently under heavy development, but already it might be regarded as the most secure, easiest to use, and simplest VPN solution in the industry.

## Supported Architectures

Our images support multiple architectures such as `x86-64`, `arm64` and `armhf`. We utilise the docker manifest for multi-platform awareness. More information is available from docker [here](https://github.com/docker/distribution/blob/master/docs/spec/manifest-v2-2.md#manifest-list) and our announcement [here](https://blog.linuxserver.io/2019/02/21/the-lsio-pipeline-project/).

Simply pulling `ghcr.io/linuxserver/wireguard` should retrieve the correct image for your arch, but you can also pull specific arch images via tags.

The architectures supported by this image are:

| Architecture | Tag |
| :----: | --- |
| x86-64 | amd64-latest |
| arm64 | arm64v8-latest |
| armhf | arm32v7-latest |


## Usage

Here are some example snippets to help you get started creating a container.

### docker-compose ([recommended](https://docs.linuxserver.io/general/docker-compose))

Compatible with docker-compose v2 schemas.

```yaml
---
version: "2.1"
services:
  wireguard:
    image: ghcr.io/linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - SERVERURL=wireguard.domain.com #optional
      - SERVERPORT=51820 #optional
      - PEERS=1 #optional
      - PEERDNS=auto #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
    volumes:
      - /path/to/appdata/config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

### docker cli

```
docker run -d \
  --name=wireguard \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e SERVERURL=wireguard.domain.com `#optional` \
  -e SERVERPORT=51820 `#optional` \
  -e PEERS=1 `#optional` \
  -e PEERDNS=auto `#optional` \
  -e INTERNAL_SUBNET=10.13.13.0 `#optional` \
  -e ALLOWEDIPS=0.0.0.0/0 `#optional` \
  -p 51820:51820/udp \
  -v /path/to/appdata/config:/config \
  -v /lib/modules:/lib/modules \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --restart unless-stopped \
  ghcr.io/linuxserver/wireguard
```


## Parameters

Container images are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 8080:80` would expose port `80` from inside the container to be accessible from the host's IP on port `8080` outside the container.

| Parameter | Function |
| :----: | --- |
| `-p 51820/udp` | wireguard port |
| `-e PUID=1000` | for UserID - see below for explanation |
| `-e PGID=1000` | for GroupID - see below for explanation |
| `-e TZ=Europe/London` | Specify a timezone to use EG Europe/London |
| `-e SERVERURL=wireguard.domain.com` | External IP or domain name for docker host. Used in server mode. If set to `auto`, the container will try to determine and set the external IP automatically |
| `-e SERVERPORT=51820` | External port for docker host. Used in server mode. |
| `-e PEERS=1` | Number of peers to create confs for. Required for server mode. Can be a list of names too: myPC,myPhone,myTablet... |
| `-e PEERDNS=auto` | DNS server set in peer/client configs (can be set as `8.8.8.8`). Used in server mode. Defaults to `auto`, which uses wireguard docker host's DNS via included CoreDNS forward. |
| `-e INTERNAL_SUBNET=10.13.13.0` | Internal subnet for the wireguard and server and peers (only change if it clashes). Used in server mode. |
| `-e ALLOWEDIPS=0.0.0.0/0` | The IPs/Ranges that the peers will be able to reach using the VPN connection. If not specified the default value is: '0.0.0.0/0, ::0/0' This will cause ALL traffic to route through the VPN, if you want split tunneling, set this to only the IPs you would like to use the tunnel AND the ip of the server's WG ip, such as 10.13.13.1. |
| `-v /config` | Contains all relevant configuration files. |
| `-v /lib/modules` | Maps host's modules folder. |
| `--sysctl=` | Required for client mode. |

## Environment variables from files (Docker secrets)

You can set any environment variable from a file by using a special prepend `FILE__`.

As an example:

```
-e FILE__PASSWORD=/run/secrets/mysecretpassword
```

Will set the environment variable `PASSWORD` based on the contents of the `/run/secrets/mysecretpassword` file.

## Umask for running applications

For all of our images we provide the ability to override the default umask settings for services started within the containers using the optional `-e UMASK=022` setting.
Keep in mind umask is not chmod it subtracts from permissions based on it's value it does not add. Please read up [here](https://en.wikipedia.org/wiki/Umask) before asking for support.

## User / Group Identifiers

When using volumes (`-v` flags) permissions issues can arise between the host OS and the container, we avoid this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id user` as below:

```
  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```


&nbsp;
## Application Setup

This image is designed for Ubuntu and Debian based systems mainly (it works on some others, but ymmv). During container start, it will first check if the wireguard module is already installed and loaded. If not, it will then check if the kernel headers are already installed (in `/usr/src`) and if not, attempt to download the necessary kernel headers from the ubuntu/debian/raspbian repos; then will compile and install the kernel module.

If you're on a debian/ubuntu based host with a custom or downstream distro provided kernel (ie. Pop!_OS), the container won't be able to install the kernel headers from the regular ubuntu and debian repos. In those cases, you can try installing the headers on the host via `sudo apt install linux-headers-$(uname -r)` (if distro version) and then add a volume mapping for `/usr/src:/usr/src`, or if custom built, map the location of the existing headers to allow the container to use host installed headers to build the kernel module (tested successful on Pop!_OS, ymmv).

With regards to arm32/64 devices, Raspberry Pi 2-4 running the [official ubuntu images prior to focal](https://ubuntu.com/download/raspberry-pi) or Raspbian Buster are supported out of the box. For all other devices and OSes, you can try installing the kernel headers on the host, and mapping `/usr/src:/usr/src` and it may just work (no guarantees).

This can be run as a server or a client, based on the parameters used. 

## Server Mode
If the environment variable `PEERS` is set to a number or a list of strings separated by comma, the container will run in server mode and the necessary server and peer/client confs will be generated. The peer/client config qr codes will be output in the docker log. They will also be saved in text and png format under `/config/peerX` in case `PEERS` is a variable and an integer or `/config/peer_X` in case a list of names was provided instead of an integer.

Variables `SERVERURL`, `SERVERPORT`, `INTERNAL_SUBNET` and `PEERDNS` are optional variables used for server mode. Any changes to these environment variables will trigger regeneration of server and peer confs. Peer/client confs will be recreated with existing private/public keys. Delete the peer folders for the keys to be recreated along with the confs.

To add more peers/clients later on, you increment the `PEERS` environment variable or add more elements to the list and recreate the container.

To display the QR codes of active peers again, you can use the following command and list the peer numbers as arguments: `docker exec -it wireguard /app/show-peer 1 4 5` or `docker exec -it wireguard /app/show-peer myPC myPhone myTablet` (Keep in mind that the QR codes are also stored as PNGs in the config folder).

The templates used for server and peer confs are saved under `/config/templates`. Advanced users can modify these templates and force conf generation by deleting `/config/wg0.conf` and restarting the container.

## Client Mode
Do not set the `PEERS` environment variable. Drop your client conf into the config folder as `/config/wg0.conf` and start the container. 

If you get IPv6 related errors in the log and connection cannot be established, edit the `AllowedIPs` line in your peer/client wg0.conf to include only `0.0.0.0/0` and not `::/0`; and restart the container.

## Road warriors, roaming and returning home
If you plan to use Wireguard both remotely and locally, say on your mobile phone, you will need to consider routing. Most firewalls will not route ports forwarded on your WAN interface correctly to the LAN out of the box. This means that when you return home, even though you can see the Wireguard server, the return packets will probably get lost.

This is not a Wireguard specific issue and the two generally accepted solutions are NAT reflection (setting your edge router/firewall up in such a way as it translates internal packets correctly) or split horizon DNS (setting your internal DNS to return the private rather than public IP when connecting locally).

Both of these approaches have positives and negatives however their setup is out of scope for this document as everyone's network layout and equipment will be different.

## Maintaining local access to attached services

** Note: This is not a supported configuration by Linuxserver.io - use at your own risk.

When routing via Wireguard from another container using the `service` option in docker, you might lose access to the containers webUI locally. To avoid this, exclude the docker subnet from being routed via Wireguard by modifying your `wg0.conf` like so (modifying the subnets as you require):

  ```
  [Interface]
  PrivateKey = <private key>
  Address = 9.8.7.6/32
  DNS = 8.8.8.8
  PostUp = DROUTE=$(ip route | grep default | awk '{print $3}'); HOMENET=192.168.0.0/16; HOMENET2=10.0.0.0/8; HOMENET3=172.16.0.0/12; ip route add $HOMENET3 via $DROUTE;ip route add $HOMENET2 via $DROUTE; ip route add $HOMENET via $DROUTE;iptables -I OUTPUT -d $HOMENET -j ACCEPT;iptables -A OUTPUT -d $HOMENET2 -j ACCEPT; iptables -A OUTPUT -d $HOMENET3 -j ACCEPT;  iptables -A OUTPUT ! -o %i -m mark ! --mark $(wg show %i fwmark) -m addrtype ! --dst-type LOCAL -j REJECT
  PreDown = HOMENET=192.168.0.0/16; HOMENET2=10.0.0.0/8; HOMENET3=172.16.0.0/12; ip route del $HOMENET3 via $DROUTE;ip route del $HOMENET2 via $DROUTE; ip route del $HOMENET via $DROUTE; iptables -D OUTPUT ! -o %i -m mark ! --mark $(wg show %i fwmark) -m addrtype ! --dst-type LOCAL -j REJECT; iptables -D OUTPUT -d $HOMENET -j ACCEPT; iptables -D OUTPUT -d $HOMENET2 -j ACCEPT; iptables -D OUTPUT -d $HOMENET3 -j ACCEPT
  ```


## Docker Mods
[![Docker Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=wireguard&query=%24.mods%5B%27wireguard%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=wireguard "view available mods for this container.") [![Docker Universal Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=universal&query=%24.mods%5B%27universal%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=universal "view available universal mods.")

We publish various [Docker Mods](https://github.com/linuxserver/docker-mods) to enable additional functionality within the containers. The list of Mods available for this image (if any) as well as universal mods that can be applied to any one of our images can be accessed via the dynamic badges above.


## Support Info

* Shell access whilst the container is running: `docker exec -it wireguard /bin/bash`
* To monitor the logs of the container in realtime: `docker logs -f wireguard`
* container version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' wireguard`
* image version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/wireguard`

## Updating Info

Most of our images are static, versioned, and require an image update and container recreation to update the app inside. With some exceptions (ie. nextcloud, plex), we do not recommend or support updating apps inside the container. Please consult the [Application Setup](#application-setup) section above to see if it is recommended for the image.

Below are the instructions for updating containers:

### Via Docker Compose
* Update all images: `docker-compose pull`
  * or update a single image: `docker-compose pull wireguard`
* Let compose update all containers as necessary: `docker-compose up -d`
  * or update a single container: `docker-compose up -d wireguard`
* You can also remove the old dangling images: `docker image prune`

### Via Docker Run
* Update the image: `docker pull ghcr.io/linuxserver/wireguard`
* Stop the running container: `docker stop wireguard`
* Delete the container: `docker rm wireguard`
* Recreate a new container with the same docker run parameters as instructed above (if mapped correctly to a host folder, your `/config` folder and settings will be preserved)
* You can also remove the old dangling images: `docker image prune`

### Via Watchtower auto-updater (only use if you don't remember the original parameters)
* Pull the latest image at its tag and replace it with the same env variables in one run:
  ```
  docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --run-once wireguard
  ```
* You can also remove the old dangling images: `docker image prune`

**Note:** We do not endorse the use of Watchtower as a solution to automated updates of existing Docker containers. In fact we generally discourage automated updates. However, this is a useful tool for one-time manual updates of containers where you have forgotten the original parameters. In the long term, we highly recommend using [Docker Compose](https://docs.linuxserver.io/general/docker-compose).

### Image Update Notifications - Diun (Docker Image Update Notifier)
* We recommend [Diun](https://crazymax.dev/diun/) for update notifications. Other tools that automatically update containers unattended are not recommended or supported.

## Building locally

If you want to make local modifications to these images for development purposes or just to customize the logic:
```
git clone https://github.com/linuxserver/docker-wireguard.git
cd docker-wireguard
docker build \
  --no-cache \
  --pull \
  -t ghcr.io/linuxserver/wireguard:latest .
```

The ARM variants can be built on x86_64 hardware using `multiarch/qemu-user-static`
```
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

Once registered you can define the dockerfile to use with `-f Dockerfile.aarch64`.

## Versions

* **06.10.20:** - Disable CoreDNS in client mode, or if port 53 is already in use in server mode.
* **04.10.20:** - Allow to specify a list of names as PEERS and add ALLOWEDIPS environment variable. Also, add peer name/id to each one of the peer sections in wg0.conf. Important: Existing users need to delete `/config/templates/peer.conf` and restart
* **27.09.20:** - Cleaning service binding example to have accurate PreDown script.
* **06.08.20:** - Replace resolvconf with openresolv due to dns issues when a client based on this image is connected to a server also based on this image. Add IPv6 info to readme. Display kernel version in logs.
* **29.07.20:** - Update Coredns config to detect dns loops (existing users need to delete `/config/coredns/Corefile` and restart).
* **27.07.20:** - Update Coredns config to prevent issues with non-user-defined bridge networks (existing users need to delete `/config/coredns/Corefile` and restart).
* **05.07.20:** - Add Debian updates and security repos for headers.
* **25.06.20:** - Simplify module tests, prevent iptables issues from resulting in false negatives.
* **19.06.20:** - Add support for Ubuntu Focal (20.04) kernels. Compile wireguard tools and kernel module instead of using the ubuntu packages. Make module install optional. Improve verbosity in logs.
* **29.05.20:** - Add support for 64bit raspbian.
* **28.04.20:** - Add Buster/Stretch backports repos for Debian. Tested with OMV 5 and OMV 4 (on kernel 4.19.0-0.bpo.8-amd64).
* **20.04.20:** - Fix typo in client mode conf existence check.
* **13.04.20:** - Fix bug that forced conf recreation on every start.
* **08.04.20:** - Add arm32/64 builds and enable multi-arch (rpi4 with ubuntu and raspbian buster tested). Add CoreDNS for `PEERDNS=auto` setting. Update the `add-peer`/`show-peer` scripts to utilize the templates and the `INTERNAL_SUBNET` var (previously missed, oops).
* **05.04.20:** - Add `INTERNAL_SUBNET` variable to prevent subnet clashes. Add templates for server and peer confs.
* **01.04.20:** - Add `show-peer` script and include info on host installed headers.
* **31.03.20:** - Initial Release.
