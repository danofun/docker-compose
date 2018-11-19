# Automated Usenet Pipeline

[![Build Status](https://travis-ci.org/duhio/docker-compose-usenet.svg?branch=master)](https://travis-ci.org/duhio/docker-compose-usenet)

An automated Usenet pipeline with reverse proxy and auto-updating of services, predominantly using the popular linuxserver Docker images. Includes:

- [NZBGet](https://hub.docker.com/r/linuxserver/nzbget/)
- [Sonarr](https://hub.docker.com/r/linuxserver/sonarr/)
- [Radarr](https://hub.docker.com/r/linuxserver/radarr/)
- [NZBHydra v2](https://hub.docker.com/r/linuxserver/hydra2/)
- [Ombi](https://hub.docker.com/r/linuxserver/ombi/)
- [Plex](https://hub.docker.com/r/linuxserver/plex/)
- [Tautulli](https://hub.docker.com/r/linuxserver/tautulli/)
- [Watchtower](https://hub.docker.com/r/v2tec/watchtower/)
- [UniFi](https://hub.docker.com/r/linuxserver/unifi/)
- [TVheadend](https://hub.docker.com/r/linuxserver/tvheadend/)
- [Bitwarden](https://hub.docker.com/r/mprasil/bitwarden/)
- [Beets](https://hub.docker.com/r/linuxserver/beets/)
- [mysql](https://hub.docker.com/_/mysql/)
- [radicale](https://hub.docker.com/r/tomsquest/docker-radicale/)
- [seafile](https://hub.docker.com/r/foxel/seafile/~/dockerfile/)
- [transmission](https://hub.docker.com/r/haugene/transmission-openvpn/)
- [transmission-rss](https://hub.docker.com/r/haugene/transmission-rss/)

## Requirements

- [Docker](https://store.docker.com/search?type=edition&offering=community)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Domain of your own (usage of a free DDNS service such as DuckDNS is not advised or supported)

## Usage

### Setup

Using `example.env`, create a file called `.env` (in the directory you cloned the repo to) by populating the variables with your desired values (see key below).

| Variable         | Purpose                                                                                   |
|------------------|-------------------------------------------------------------------------------------------|
| CONFIG           | Where the configs for services will live. Each service will have a subdirectory here      |
| MEDIA            | Where your data is stored and where sub-directories for tv, movies, etc will be put       |
| SEAFILE          | Path to your seafile data folder                                                          |
| OVPNSERVER       | The OpenVPN server used by Transmission. [List here.](https://git.io/fpC92)               |
| OVPNUSER         | The OpenVPN server's username used by Transmission.                                       |      
| OVPNPASS         | The OpenVPN server's password used by Transmission.                                       |
| OVPNCONFIG       | The OpenVPN config file used by Transmission. Set with filename minus .ovpn [List here](https://git.io/fpCSF)|
| DOMAIN           | The domain you want to use for access to services from outside your network               |
| TZ               | Your timezone. [List here.](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) |
| HTPASSWD         | HTTP Basic Auth entries in HTPASSWD format ([generate here](http://www.htaccesstools.com/htpasswd-generator/))|

Values for User ID (PUID) and Group ID (PGID) can be found by running `id user` where `user` is the owner of the volume directories on the host.

#### Traefik

1. Create a folder called `traefik` in your chosen config directory. Everything below should be executed inside the `traefik` directory
2. Run `touch acme.json; chmod 600 acme.json`
3. Copy `traefik.toml` to the `traefik` directory in your config folder and replace the example email with your own

### Running

In the directory containing the files, run `docker-compose up -d`. Each service should be accessible (assuming you have port-forwarded on your router) on `<service-name>.<your-domain>`. Heimdall should be accessible on `<your-domain>`, from where you can set it up to provide a convenient homepage with links to services. The Traefik dashboard should be accessible on `monitor.<your-domain>`.

#### Service Configuration

When plumbing each of the services together you can simply enter the service name and port instead of using IP addresses. For example, when configuring a download client in Sonarr/Radarr enter `sabnzbd` in the Host field and `8080` in the Port field. The same applies for other services such as NZBHydra.

#### Customisation

##### Service customisation

To add a new volume mount or otherwise customise an existing service, create a file called `docker-compose.override.yml`.

For example, to add new volume mounts to existing services:

```yaml
version: '3'

services:
  radarr:
    volumes:
      - ${DATA}/documentaries:/media/documentaries

  plex:
    volumes:
      - ${DATA}/documentaries:/media/documentaries
```

You can also add new services to the stack using the same method.

## Notes / Caveats

### Plex

Plex config may not be visible until you SSH tunnel:

- `ssh -L 8080:localhost:32400 user@dockerhost`

Once done you can browse to `localhost:8080/web/index.html` and set up your server.

### UnRAID Usage

Only tested on UnRAID *6.4.1+*.

#### Installing Docker Compose

Add the following to `/boot/config/go` in order to install docker-compose on each boot:

```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### Persisting user-defined networks

By default, UnRAID will not persist user-defined Docker networks such as the one this stack will create. You'll need to enable this setting in order to avoid having to re-run `docker-compose up -d` every time your server is rebooted. It's found in the _Docker_ tab, you'll need to set _Advanced View_ to on and stop the Docker service to make the change.

#### UnRAID UI port conflict

You'll need to either change the HTTPS port specified for the UnRAID WebUI (in _Settings_ -> _Identification_) or change the host port on the Traefik container to something other than 443 and forward 443 to that port on your router (eg 443 on router forwarded to 444 on Docker host) in order to allow Traefik to work properly.

## Help / Contributing

If you need assistance, please file an issue. Please do read the [existing closed issues](https://github.com/duhio/docker-compose-usenet/issues?q=is%3Aissue+is%3Aclosed) as they may contain the answer to your question.

Pull requests for bugfixes/improvements are very much welcomed. As are suggestions of new/replacement services.
