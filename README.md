# Athena OS Docker Image

![athena-banner](https://user-images.githubusercontent.com/83867734/221656804-51b13a4f-876b-4ca8-856e-288d2209a949.png)

This repository provides the Docker image that runs a [GNOME](https://gnome.org/)
desktop on top of [Athena OS](https://github.com/Athena-OS/athena-iso). The desktop is accessible via
[RDP (Remote Desktop Protocol)](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol)
clients such as [Remmina](https://remmina.org/), [FreeRDP](https://www.freerdp.com),
[Microsoft Remote Desktop](https://www.microsoft.com/en-us/p/microsoft-remote-desktop/9wzdncrfj3ps)
([for macOS](https://itunes.apple.com/app/microsoft-remote-desktop/id1295203466)).

Find us at:

* [Discord](https://discord.gg/DNjvQkb5Ad) - realtime support / chat with the community and the team.
* [GitHub](https://github.com/Athena-OS) - view the source for all of our repositories.

## Usage

Athena OS container has been developed in order to be run also by podman. The choice to use podman comes from its advantages over docker, one of most important: security.

According to your preference, install `docker` and `docker-compose` packages or `podman` package for your Linux environment.

In case you are using podman, edit `/etc/containers/registries.conf` and add:
```
[registries.search]
registries = ['docker.io']
```
in order to allow podman to search for images in Docker Hub.

### Hack The Box API Token

Athena OS container allows you to learn and play on Hack The Box platform. It is possible to access to Hack The Box by using your App Token. Retrieve your App Token from the Hack The Box website in your Profile Settings.

### Docker

You can run the container by `docker-compose` (recommended) or `docker run`.

#### docker-compose

The `docker-compose.yml` file should have the following content:
```yaml
services:
  athena-rdp:
    image: athenaos/rdp
    environment:
      - DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
    cap_add:
      - cap_sys_admin
      - ipc_lock
      - net_admin
    cgroup: host
    devices:
      - /dev/net/tun
    ports:
      - "127.0.0.1:23389:3389"
      - "127.0.0.1:8022:22"
    shm_size: '2gb'
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=0
    volumes:
       - /sys/fs/cgroup:/sys/fs/cgroup
    tmpfs:
      - /run
      - /run/user/1000:uid=1000,gid=1000,mode=700
      - /tmp
    restart: unless-stopped
```

Run the container by:
```
sudo docker-compose run athena-rdp
```

#### docker run

```bash
docker run -ti \
  --name athena-rdp \
  --cap-add CAP_SYS_ADMIN \
  --cap-add IPC_LOCK \
  --cap-add NET_ADMIN \
  --cgroupns=host \
  --device /dev/net/tun \
  --shm-size 2G \
  --sysctl net.ipv6.conf.all.disable_ipv6=0 \
  --volume /sys/fs/cgroup:/sys/fs/cgroup \
  --publish 23389:3389 \
  --publish 8022:22 \
  --restart unless-stopped \
  docker.io/athenaos/rdp:latest
```
or
```
docker run -ti --name athena-rdp --cap-add CAP_SYS_ADMIN --cap-add IPC_LOCK --cap-add NET_ADMIN --cgroupns=host --device /dev/net/tun --shm-size 2G --sysctl net.ipv6.conf.all.disable_ipv6=0 --volume /sys/fs/cgroup:/sys/fs/cgroup --publish 23389:3389 --publish 8022:22 --restart unless-stopped docker.io/athenaos/rdp:latest
```

In case you exit the container and need to re-enter, run:
```
docker exec --user athena -ti athena-rdp /bin/zsh
```
In case the container is not running, run:
```
docker start athena-rdp
```

For stopping the container, run:
```
docker stop athena-rdp
```

For deleting the container, run:
```
docker container rm athena-rdp
```

### Podman (Untested)

You can run the container by `podman run`.

#### podman run

```bash
podman run -ti \
  --name athena-rdp \
  --cap-add NET_RAW \
  --cap-add NET_ADMIN \
  --device /dev/net/tun \
  --restart unless-stopped \
  docker.io/athenaos/rdp:latest
```
or
```
podman run -ti --name athena-rdp --cap-add NET_RAW --cap-add NET_ADMIN --device /dev/net/tun --restart unless-stopped docker.io/athenaos/rdp:latest
```

Podman will automatically replicate `/etc/hosts` and `/etc/hostname` files of your host. For preventing this, add `--no-hosts` argument to the `podman run` command above.

In case you exit the container and need to re-enter, run:
```
podman exec --user athena -ti athena-rdp /bin/zsh
```
In case the container is not running, run:
```
podman start athena-rdp
```

For stopping the container, run:
```
podman stop athena-rdp
```

For deleting the container, run:
```
podman container rm athena-rdp
```

### Default Credentials

```
athena:athena
```

### Setting HTB API Token

Once you retrieved your HTB API Token from your Hack The Box website profile settings, you can set the HTB API Token in two ways:
* By connecting by RDP to the GUI as explained in the next section, open terminal and run `htb-update` (or clicking **HTB Update** button in Athena Welcome)
* If you don't want to connect to the GUI environment, run the following commands:
  ```
  sudo mkdir -p /run/secrets
  sudo nano /run/secrets/htb-api
  <store your HTB API Token in htb-api file and save it>
  htb-play
  ```
  
## Connect to the desktop by RDP

You should now be able to access your full-featured GNOME desktop using
the RDP client of your choice. For example, using [Remmina](https://remmina.org) along with the install of `freerdp` package:

```
remmina -c rdp://127.0.0.1:23389
```

### Remmina features

When you run Remmina, expand its window and on the left side press the `Toggle dynamic resolution update` icon button for automatically resizing the display.

For making the keyboard shortcut effective on the connected environment, on the left side press the `Grab all keyboard events` icon.

For copy and paste text from the host to the environment, copy a string, then, on the left side, click on `Tools` icon button and select `Keystrokes` and `Send clipboard content as keystrokes`.

## Customizing and building the image

Clone this repository, edit `Dockerfile` and then run `docker build` as usual:

```
git clone https://github.com/Athena-OS/athena-rdp-docker.git
cd athena-rdp-docker
docker buildx build --tag 'athenaos/rdp:latest' .
```

### Invalidating cache

Use `--no-cache` option:

```
docker build --tag 'custom-athena-rdp:latest' --no-cache .
```
