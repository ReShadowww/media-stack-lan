# media-stack

A stack of self-hosted media managers:
- qbittorrent -> http://localhost:5080
- Radarr -> http://localhost:7878
- Sonarr -> http://localhost:8989
- Prowlarr -> http://localhost:9696
- Jellyseerr -> http://localhost:5055
- Jellyfin -> http://localhost:8096
- Bazarr -> http://localhost:6767

## Requirements

- Docker version 23.0.5 and above
- Docker compose version v2.17.3 and above
- It may also work on some of lower versions, but its not tested.

## Install media stack and create media-stack-network

```
git clone https://github.com/ReShadowww/media-stack.git
cd media-stack-lan
sudo docker network create media-stack-network
sudo docker compose up -d
```

## Get Host Addresses 

```
text="A stack of self-hosted media managers:
- qbittorrent -> http://$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+\.\d+\.\d+\.\d+'):5080
- Radarr -> http://$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+\.\d+\.\d+\.\d+'):7878
- Sonarr -> http://$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+\.\d+\.\d+\.\d+'):8989
- Prowlarr -> http://$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+\.\d+\.\d+\.\d+'):9696
- Jellyseerr -> http://$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+\.\d+\.\d+\.\d+'):5055
- Jellyfin -> http://$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+\.\d+\.\d+\.\d+'):8096
- Bazarr -> http://$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+\.\d+\.\d+\.\d+'):6767"
echo "$text"
```


## Configure qBittorrent

- Open qBitTorrent at http://localhost:5080. Default username:password is admin:adminadmin
```
# If username:password is not admin:adminadmin check qbittorent logs

sudo docker logs qbittorrent
```
- Go to Tools --> Options --> WebUI --> Change password
- Run below commands

```
sudo docker exec -it qbittorrent bash

chown 1000:1000 /downloads/movies /downloads/tv-series
```

## Configure Radarr and Sonarr

- Open Radarr at http://localhost:7878 or Sonarr at http://localhost:8989/
- Settings --> Media Management -->
  - Check mark "Movies deleted from disk are automatically unmonitored in Radarr" under File management section
  - Add Root Folder -> /downloads/tv-series for Sonarr and /downloads/movies for Radarr
  - Save Changes
- Settings --> Download clients --> qbittorrent -->
  - Add Host (qbittorrent) (use Get Host Addresses command from above e.g. 172.17.227.100)
  - Port 5080
  - Username and password (admin:adminadmin is the default)
  - Tick box "Sequential Order"
  - Tick box "First and Last First"
  - Test --> Save

## Configure Prowlarr

- Open Prowlarr at http://localhost:9696
- Add application, Settings --> Apps --> Add application --> Choose Sonarr or Radarr or any apps to link -->
  - Prowlarr server (http://localhost:9696)
  - Radarr server (use Get Host Addresses command from above e.g. 172.17.227.100:7878) --> API Key (found in Radarr settings -> General) --> Test and Save
  - Sonarr server (use Get Host Addresses command from above e.g. 172.17.227.100:) --> API Key (found in Sonarr settings -> General) --> Test and Save
- Add Indexers, Indexers --> Add Indexer --> Search for indexer --> Choose base URL --> Test and Save
- This will add indexers in respective apps automatically.

## Add a movie

- Movies --> Search for a movie --> Quality profile --> Add movie
- Go to qbittorrent (http://localhost:5080) and see if movie is getting downloaded.


## CAUTION! Deletes EVERYTHING!, used only for test reseting
```
sudo docker stop $(sudo docker ps -a -q)
sudo docker system prune -a
sudo docker volume rm $(sudo docker volume ls -q)
sudo docker network prune
```
