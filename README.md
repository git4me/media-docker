Cloned from https://github.com/joshuhn/media-docker

Tweaked for my needs, start with https://github.com/joshuhn/media-docker

# media-docker
a short and sweet way to get a full-blown media stack running on a server in minutes. four files and sudo access are all you need to get started.

## what's included
with this package, you'll get a media server environment capable of finding, grabbing, downloading, and presenting: movies, tv, books, music, and comics. it does this (relatively) securely, prioritizing usenet and torrenting-over-VPN. traefik acting as a reverse proxy manages access to everything via nice URL's, and watchtower keeps all of your docker containers up-to-date.

### services and where to access them
| service | purpose | url / ports |
| ------- | ------- | :---------: |
| plex | movie / tv / music interface | https://plex.${DOMAIN} <br> :32400 |
| ubooquity | book / comic interface | https://ubooquity.${DOMAIN}/ubooquity <br> https://admin-ubooquity.${DOMAIN}/ubooquity/admin |
| traefik | reverse proxy | https://traefik.${DOMAIN} <br> :80 <br> :443|
| heimdall | dashboard | https://heimdall.${DOMAIN} |
| sabnzbd | nzb download | https://sab.${DOMAIN} |
| transmission | torrent download | https://transmission.${DOMAIN} |
| hydra | nzb searcher | https://hydra.${DOMAIN} |
| jackett | torznab searcher | https://jackett.${DOMAIN} |
| sonarr | tv management | https://sonarr.${DOMAIN} |
| radarr | movie management | https://radarr.${DOMAIN} |
| lidarr | music management | https://lidarr.${DOMAIN} |
| lazylibrarian | book management | https://lazylibrarian.${DOMAIN} |
| mylar | comic management | https://mylar.${DOMAIN} |
| ombi | plex requests | https://ombi.${DOMAIN} |
| tautulli | plex statistics | https://tautulli.${DOMAIN} |
| cockpit | server statistics & management | :9090 |

## installation
installation is omega-easy!

1. have an Ubuntu machine available
2. be root
3. run `sudo git clone https://github.com/joshuhn/media-docker/ /media-docker/ && cd /media-docker/`
4. open `.env` in your favorite text editor and set the variables it contains
5. make sure that the script is executable (`chmod +x ./deploy.sh`) and run `./deploy.sh`
6. wait a few minutes, watch the color-coded status messages scroll by
7. use your newly configured media stack! hooray!

note: there is an additional step required to link the Plex server to your account. Plex requires this step to be done from localhost, so you'll need to set up an ssh tunnel to make Plex think you're localhost. 

if you're on \*nix, this can be done like this: 
```
ssh -L 8080:localhost:32400 user@host
```
if you're on Windows, i'd recommend doing the same via puTTY.

### non-apt systems
if you're running on a system that doesn't use the apt package manager, you unfortunately can't use the `./deploy.sh` script. it relies heavily on apt, so you'll need to perform the configuration manually. refer to the deploy.sh section below for details on what the script actually does.

if you're confident your system is configured appropriately, run `docker-compose up --force-recreate -d` from `/media-docker`.

## what are those?
### deploy.sh
a straightforward shell script that ensures your environment is configured as needed to ensure a solid media server. the process it takes is as follows:

1. perform local server configuration (sudo user setup, timezone)
2. cleanup legacy Docker packages
3. install and enable apt over HTTPS
4. grab and install Docker's official GPG key, add their managed repository to apt
5. install Docker CE & Docker Compose
6. install Cockpit for remote management
7. pre-configure Traefik (set your email & put files in place)
8. build the Docker container environment via docker-compose.yml
9. everything's functional!

all of the customizable bits (users, directories, email) are pulled from the dotenv file. if anything fails, the shell script will tell you in big red text.

if you're feeling saucy and confident that your system is properly configured (or if you're running a non-apt system), you can bypass `.\deploy.sh` and run `docker-compose up --force-recreate -d` on your own.

### docker-compose.yml
this file contains all of the instructions Docker Compose needs to pull, build, and configure the entire media environment. as with `.\deploy.sh`, all user-configurable items are exposed through the dotenv file.

all services are accessed via a Traefik reverse proxy for security. unfortunately, due to the complexity or poor design (or both) of Plex, it's also able to be reached directly. there's light at the end of that tunnel, though, as the Traefik team are currently working on a method of applying multiple routing labels to a single container. once that's implemented, we'll apply it here to make up for the needlessly many ports those services use.

### traefik.toml
you don't need to worry about this one, the only thing to change is your email address and `.\deploy.sh` takes care of this for you.

### .env
a simple dotenv file containing the variables necessary to configure and install all necessary components for the project.

| variable | function | example |
| -------- | -------- | ------- |
| USER_NAME | username for account that will be created by `./deploy.sh` | joshuhn |
| PASSWORD | password for account that will be created by `./deploy.sh` | iM@gr8Password! |
| BASE_DIR | base directory path for container storage | /mnt/meda/data/ |
| MEDIA_DIR | base directory path for media library storage | /mnt/media/data/media-library |
| DOWNLOAD_DIR | base directory path for downloads | /mnt/media/data/downloads |
| TIMEZONE | local timezone, set by `./deploy.sh` | America/Montevideo |
| DOMAIN | domain name for the media stack, used by traefik | media.com |
| COMPOSE_VERSION | version of docker-compose to pull from GitHub | 1.20.0-rc2 |
| VPN_PROVIDER | openvpn provider, supported values in the table below | NORDVPN |
| VPN_USER | openvpn username | joshuhn |
| VPN_PASS | openvpn password | iM@gr8Password! |
| EMAIL_ADDRESS | email used for let's encrypt | josh@email.me |

### vpn providers
the following is lifted wholesale from the base package's readme. the full text can be found [here.](https://github.com/haugene/docker-transmission-openvpn)

This is a list of providers that are bundled within the image. Feel free to create an issue if your provider is not on the list, but keep in mind that some providers generate config files per user. This means that your login credentials are part of the config an can therefore not be bundled. In this case you can use the custom provider setup described later in this readme. The custom provider setting can be used with any provider.

| Provider Name                | Config Value (`OPENVPN_PROVIDER`) |
|:-----------------------------|:-------------|
| Anonine | `ANONINE` |
| AnonVPN | `ANONVPN` |
| BlackVPN | `BLACKVPN` |
| BTGuard | `BTGUARD` |
| Cryptostorm | `CRYPTOSTORM` |
| Cypherpunk | `CYPHERPUNK` |
| FrootVPN | `FROOT` |
| FrostVPN | `FROSTVPN` |
| Giganews | `GIGANEWS` |
| HideMe | `HIDEME` |
| HideMyAss | `HIDEMYASS` |
| IntegrityVPN | `INTEGRITYVPN` |
| IPredator | `IPREDATOR` |
| IPVanish | `IPVANISH` |
| Ivacy | `IVACY` |
| IVPN | `IVPN` |
| Mullvad | `MULLVAD` |
| Newshosting | `NEWSHOSTING` |
| NordVPN | `NORDVPN` |
| OVPN | `OVPN` |
| Perfect Privacy | `PERFECTPRIVACY` |
| Private Internet Access | `PIA` |
| PrivateVPN | `PRIVATEVPN` |
| proXPN | `PROXPN` |
| PureVPN | `PUREVPN` |
| RA4W VPN | `RA4W` |
| SaferVPN | `SAFERVPN` |
| SlickVPN | `SLICKVPN` |
| Smart DNS Proxy | `SMARTDNSPROXY` |
| SmartVPN | `SMARTVPN` |
| TigerVPN | `TIGER` |
| TorGuard | `TORGUARD` |
| UsenetServerVPN | `USENETSERVER` |
| Windscribe | `WINDSCRIBE` |
| VPNArea.com | `VPNAREA` |
| VPN.AC | `VPNAC` |
| VPN.ht | `VPNHT` |
| VPNBook.com | `VPNBOOK` |
| VPNTunnel | `VPNTUNNEL` |
| VyprVpn | `VYPRVPN` |

## thanks!
this very much stands on the shoulders of those who came before, all this has done is made the deployment process simple.

this project makes use of the following Docker containers:
- linuxserver\plex
- linuxserver\ubooquity
- linuxserver\sabnzbd
- linuxserver\hydra
- linuxserver\jackett
- linuxserver\sonarr
- linuxserver\radarr
- linuxserver\lidarr
- linuxserver\lazylibrarian
- linuxserver\mylar
- linuxserver\heimdall
- linuxserver\ombi
- tautulli\tautulli
- haugene\transmission-openvpn
- haugene\transmission-openbpn-proxy
- traefik
- watchtower
