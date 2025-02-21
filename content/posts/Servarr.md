+++
title = 'An Imbeciles Guide to Servarr'
date = 2025-02-08
draft = false
+++
## Introduction

About two years ago, a friend found half a dozen ThinkStation P330 Tinys for sale by a local CAD company that was upgrading their machines. $130 and a fresh install of Ubuntu Server later I was the proud owner of my first home server and, as I would soon come to find out, constant source of stress.

My first attempt at a media stack included Jellyfin, Radarr, Transmission, and Jackett. It was outdated, unreliable, and slow—and it lacked a VPN. Frustrated with downloads taking two weeks for a 2GB movie, I eventually gave up. After moving apartments, my little ThinkStation ended up sitting in a closet for nearly a year and a half. That is, until I recently decided to take a break from doomscrolling social media. Boredom turned into productivity, and so I gave it a second shot -- this time with a more up-to-date stack and a bit more patience.

After a couple weeks of reading docs, reddit posts, blogs, and forums I am finally satisified with my Servarr setup and thought I would share my experience in case anyone runs into similar roadblocks while setting up their own media stack.

You can find my entire `docker-compose.yml` and any other configs I used in the [appendix](#appendix) at the end of this article.

## Media Stack

Not knowing exactly where to begin, I did some snooping and eventually found a popular guide by <a href="https://github.com/navilg" target="_blank">navilg</a>. This easy to follow guide got the ball rolling with a dockerized media stack consisting of:
- Jellyfin,
- Jellyseerr,
- Radarr,
- Sonarr,
- Prowlarr,
- qBittorrent,
- and VPN config.

The guide also has instructions for how to configure Ngnix if you wish to host Jellyfin externally, but I chose to just have a local setup.

## Getting Started

Following the guide by navilg, I got my Docker network set up, cloned the `docker-compose.yml`, filled out the necessary environment variables and followed the inline comments to configure the settings for the VPN profile. Most guides recommend **against** using the latest version of the Docker images to avoid incompatibility, but I have never ran into and issue with this while using this stack, so I changed all the image versions to `:latest`.

For my VPN I chose <a href="https://protonvpn.com/free-vpn" target="_blank">Proton VPN's free tier</a> which does the job well as long as you don't care about port forwarding (we will cover this later). Just make sure you have uncommented `FREE_ONLY=on` in the `docker-compose.yml` and set your country to either Netherlands, United States, or Japan.

After finishing the initial setup, I now had all my services configured and running locally. We can now get into some of the aforementioned roadblocks.

## Keep it Simple, Stupid!

My first mistake was wanting to access as many indexers as I could to increase my chances of getting my grubby little hands on as many free movies as possible. This approach required a Flaresolverr proxy configuration as many indexers are protected by Cloudflare DDoS Protection. So, I added Flaresolverr to my `docker-compose.yml` and tried to traffic it through my VPN.

```yaml
services:
  flaresolverr:
    profiles: ["vpn", "no-vpn"]
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=info
      - LOG_HTML=false
      - CAPTCHA_SOLVER=none
      - TZ=UTC
    ports:
      - "8191:8191"
    restart: unless-stopped
    network_mode: service:vpn
```

I connected it to Prowlarr, and tried to add an indexer that required it. This started a multi-day hair pulling exercise in learning about DNS settings and Docker networking as I fought for my life to try to solve the error `Unable to connect to indexer, please check your DNS settings and ensure IPv6 is working or disabled`. After questioning friends with more Docker and networking experience than myself to no avail, I finally joined the <a href="https://discord.com/invite/8Dbsx35rrx" target="_blank">Servarr Discord</a> where I was informed that Flaresolverr has been in shambles since Cloudflare developers have been keeping an eye on it.

>Flaresolverr is currently BROKEN. The Cloudflare devs are watching the repo and so it is very likely permanently dead.
>
>Also, we are not the Flaresolverr support team and cannot help you with questions that don't relate specifically to adding Flaresolverr to Prowlarr.
>
>Flaresolverr is a 3rd party program which solves cloudflare captchas for some indexers.
>They use Github for support, and you should go there to ask them questions or catch up on the current status of the program.
>
>The current open issue on their github is: FlareSolverr/FlareSolverr#1253"

So after all my efforts, I decided to scrap all hopes of using Flaresolverr tagged indexers. This turned out to be perfectly fine, as sometimes less is more, and I have found having a few highly rated indexers just as good as having them all.

My next hiccup was having Prowlarr passed to my VPN. Although this is the default setting in navilg's guide, I do not believe it to be correct. All of my indexers would fail almost immediately after restarting the container, so I started digging into logs. After inspecting both Docker and *arr logs there was _a lot_ of `HTTP request timed out` and `HTTP 429: TooManyRequests` errors. 

I knew this was likely due to my VPN configuration, but no matter what I did (e.g. changing DNS to `1.1.1.1` or `8.8.8.8` in the VPN, containers, dropping the firewall all together, and becoming all too familiar with <a href="https://github.com/qdm12/gluetun" target="_blank">Gluetun</a> documentation) I just couldn't get it. 

I finally just decided I had to get rid of the VPN and risk getting some threatening mail from my ISP when a friend mentioned that you dont _need_ to have Prowlarr networked through your VPN. It turns out he was right! Although interacting with indexers is a bit shady, its not really illegal. As long as you have your download client handled by the VPN you should be fine. So, I got rid of `network_mode: service:vpn` for `mynetwork`, assigned it a static ipv4 address, et voilà, I had functioning indexers!

```yaml
  prowlarr:
    profiles: ["vpn", "no-vpn"]
    container_name: prowlarr
    image: linuxserver/prowlarr:latest

    networks:
      mynetwork:
        ipv4_address: 172.20.0.40
```

## Improvements

You should now have a functioning media stack, but if your experience is anything like mine, your torrents are downloading slower than molasses in winter. 

I found a <a href="https://www.reddit.com/r/qBittorrent/comments/fvpeib/comment/fmk3yh6/">reddit post</a> by u/WankWankNudgeNudge and after following their suggestions and giving qBittorrent 4GB of ram instead of the default 512MB, my downloading speeds went from about 1% _per day_ to a finishing whole 30GB file in under an hour.

- Tools>Options>Connection:

  + Enabled Protocol - set to `TCP` (just `TCP`, not `TCP and uTP`)

  + Uncheck all boxes under `Listening Port` and `Connections Limits`.

- Tools>Options>Speed: 
  + Uncheck all boxes under `Rate Limits Settings`.

You can improve your speeds more by port forwarding either via your router (if you aren't using the VPN profile) or through your VPN. Port forwarding is not _required_ but if you do not do it, you are restricting yourself to only connecting to peers who _do_. 

I recently upgraded from Proton VPN free tier to Proton VPN Plus to do this, but have not got around to it yet. Apparently my motivation comes from frustration, and my downloads are fast enough for me at the moment. However, if you choose to go this route Proton has a <a href="https://protonvpn.com/support/port-forwarding#qbittorrent" target="_blank">guide to enable port forwarding in your qBittorrent client</a>.

This may not belong under improvements, but for myself, I dislike the default mounting points that the `docker-compose.yml` provided by navilg has. I opted to migrate these to bind mounts just for ease of access. If you set your mounts from the get-go, just set your path to be whatever you like (e.g. `./volumes`). However if you are migrating like I did you will need to change the host mounting point in your `docker-compose.yml` to something like `/home/user/jellyfin/volumes/media-stack_{volumename}/_data`. Note the `_data` directory. Since you are migrating with, I assume, something like `cp -r /var/lib/docker/volumes/media-stack_* /home/user/jellyfin/volumes`, all of your files will be nested behind `_data`. This is just the default location used by Docker.

You will also need to delete the volumes from the bottom of your `docker-compose.yml`.

```yaml
volumes:
 torrent-downloads:
 radarr-config:
 sonarr-config:
 prowlarr-config:
 jellyfin-config:
 qbittorrent-config:
 jellyseerr-config:
```

### Using Ubuntu?

If you are using Ubuntu you may be facing a lack of storage space. I only had 98gb free which, at first, I thought was some limit on the `torrent-downloads` mount or a restriction in my root partition.

I ran `lsblk` to find out Ubuntu only allocates 100GB to your root filesystem.

```bash
nvme0n1                   259:0    0 476.9G  0 disk 
├─nvme0n1p1               259:1    0     1G  0 part /boot/efi
├─nvme0n1p2               259:2    0     2G  0 part /boot
└─nvme0n1p3               259:3    0 473.9G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   100G  0 lvm  /
```

I know, I need a bigger hard drive. Anyways, putting all my trust in ChatGPT I simply ran the following commands and gave `/` 100% of nvme0n1p3:

```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

## Afterthoughts

I am now _mostly_ satisified with my media stack, however I still need to make a few tweaks such as port forwarding my VPN, and maybe one day implementing Ngnix so I can access Jellyfin outside of my home network.

Overall, if you avoid overcomplicating and learn from my mistakes you should be watching your favourite shows and movies in no time.

**Disclaimer**: This article is for informational and educational purposes only. I do not condone or encourage copyright infringement, piracy, or any illegal activities. Users are solely responsible for ensuring that their actions comply with all applicable laws in their jurisdiction. I assume no liability for any misuse of the information provided. Proceed at your own risk.

Have fun sailing the seven seas!

## Appendix

```yaml
name: media-stack
services:
  vpn:
    profiles: ["vpn"]
    container_name: vpn
    image: qmcgaw/gluetun:latest
    cap_add:
      - NET_ADMIN

    environment:
      - VPN_SERVICE_PROVIDER=protonvpn # Valid values: nordvpn, expressvpn, protonvpn, surfshark or custom
      - OPENVPN_USER=""
      - OPENVPN_PASSWORD=""
      - SERVER_COUNTRIES=Canada
      #- FREE_ONLY=on

    networks:
      - mynetwork

    devices:
      - /dev/net/tun:/dev/net/tun

    ports:
      - 5080:5080
      - 6881:6881
      - 6881:6881/udp

    restart: "unless-stopped"

  qbittorrent:
    profiles: ["vpn", "no-vpn"]
    container_name: qbittorrent
    image: lscr.io/linuxserver/qbittorrent:latest
    network_mode: service:vpn

    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
      - WEBUI_PORT=5080

    volumes:
      - /home/user/jellyfin/volumes/media-stack_qbittorrent-config/_data:/config
      - /home/user/jellyfin/volumes/media-stack_torrent-downloads/_data:/downloads

    restart: "unless-stopped"

  radarr:
    profiles: ["vpn", "no-vpn"]
    container_name: radarr
    image: lscr.io/linuxserver/radarr:latest
    networks:
      mynetwork:
        ipv4_address: 172.20.0.20 # It should be available IPv4 address in range of docker network `mynetwork` e.g. 172.20.0.2  

    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC

    ports:
      - 7878:7878

    volumes:
      - /home/user/jellyfin/volumes/media-stack_radarr-config/_data:/config
      - /home/user/jellyfin/volumes/media-stack_torrent-downloads/_data:/downloads

    restart: "unless-stopped"

  sonarr:
    profiles: ["vpn", "no-vpn"]
    image: linuxserver/sonarr:latest
    container_name: sonarr
    networks:
      mynetwork:
        ipv4_address: 172.20.0.30 # It should be available IPv4 address in range of docker network `mynetwork` e.g. 172.20.0.2

    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC

    volumes:
      - /home/user/jellyfin/volumes/media-stack_sonarr-config/_data:/config
      - /home/user/jellyfin/volumes/media-stack_torrent-downloads/_data:/downloads

    ports:
      - 8989:8989

    restart: unless-stopped

  prowlarr:
    profiles: ["vpn", "no-vpn"]
    container_name: prowlarr
    image: linuxserver/prowlarr:latest
    networks:
      mynetwork:
        ipv4_address: 172.20.0.40

    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC

    volumes:
      - /home/user/jellyfin/volumes/media-stack_prowlarr-config/_data:/config

    ports:
      - 9696:9696

    restart: unless-stopped

  jellyseerr:
    profiles: ["vpn", "no-vpn"]
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    networks:
      - mynetwork
    dns:
      - 8.8.8.8
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
      - FORCE_IPV4_FIRST=true

    volumes:
      - /home/user/jellyfin/volumes/media-stack_jellyseerr-config/_data:/app/config

    ports:
      - 5055:5055

    restart: unless-stopped

  jellyfin:
    profiles: ["vpn", "no-vpn"]
    image: linuxserver/jellyfin:10.10.3
    container_name: jellyfin
    networks:
      - mynetwork
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC

    volumes:
      - /home/user/jellyfin/volumes/media-stack_jellyfin-config/_data:/config
      - /home/user/jellyfin/volumes/media-stack_torrent-downloads/_data:/data

    ports:
      - 8096:8096
      - 7359:7359/udp
      - 8920:8920

    restart: unless-stopped

networks:
  mynetwork:
    external: true
```

You can optionally add Readarr and/or Calibre if you enjoy books!

```yaml
  readarr:
    profiles: ["vpn", "no-vpn"]
    image: lscr.io/linuxserver/readarr:develop
    container_name: readarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC

    networks:
      mynetwork:
        ipv4_address: 172.20.0.60 # It should be available IPv4 address in range of docker network `mynetwork` e.g. 172.20.0.2

    volumes:
      - /home/keenan/jellyfin/volumes/media-stack_readarr-config/_data:/config
      - /home/keenan/jellyfin/volumes/media-stack_torrent-downloads/_data:/downloads #optional
    ports:
      - 8787:8787
    restart: unless-stopped

 calibre:
   profiles: ["vpn", "no-vpn"]
   image: lscr.io/linuxserver/calibre:latest
   container_name: calibre
   environment:
     - PUID=1000
     - PGID=1000
      - TZ=Etc/UTC

    networks:
     - mynetwork
    volumes:
     - /home/keenan/jellyfin/volumes/media-stack_calibre-config/_data:/config
     - /home/keenan/jellyfin/volumes/media-stack_torrent-downloads/_data:/data
    ports:
      - 8080:8080
      - 8181:8181
      - 8081:8081
   restart: unless-stopped
```

Just throw that at the bottom of your `docker-compose.yml`, but be careful, Calibre's default webUI port is `8080` which is commonly used, so just make sure you don't have anything else running on it.