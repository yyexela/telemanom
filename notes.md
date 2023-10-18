# Summary:
Problem:
- Container not added to network bridge
- Show network bridge `docker0`: `brctl show`
- Show container interface: `ip addr` should be something like `vethedbea25@if15
- Add manually with: `sudo brctl addif docker0 'vethedbea25'`

Solution:
- Added container to bridge
- ip addr: `vethedbea25@if15`
- Really, what I did was `sudo mv /etc/systemd/network/20-wired.network /etc/systemd/network/20-wired.network_bak`
- Since I match `ether` instead of something specific, systemd was messing with docker interfaces somehow

# Helpful commands: 
Move docker install location:
- https://www.ibm.com/docs/en/z-logdata-analytics/5.1.0?topic=compose-relocating-docker-root-directory

Prune docker:
- `docker system prune -a`

Start docker container:
- `docker container run -it <image:ubuntu> <command:bash>`

Check current DNS:
- `resolvectl status`

Telemanom build and run:
- `docker build -t telemanom .`
- `docker run telemanom`

Check DNS:
- `docker run --rm busybox nslookup google.com`

Wrote to:
- `/etc/sysctl.d/30-ipforward.conf`
- `sysctl -a | grep forward`
- Source: https://wiki.archlinux.org/title/Internet_sharing#Enable_packet_forwarding

iptables:
- Print all rules: `sudo iptabled -S | grep docker0`
- Print nat rules: `sudo iptables -S -t nat`
- Remove config: `iptables -t nat -D INPUT -i docker0 -j ACCEPT`
- Add config: `iptables -t nat -A INPUT -i docker0 -j ACCEPT`

brctl:
- `brctl show`

Check:
- `cat /etc/sysctl.conf`

# Error
```
[alexey@yyexela telemanom]$ docker build -t telemanom .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon   2.11MB
Step 1/12 : FROM ubuntu:latest
 ---> e4c58958181a
Step 2/12 : RUN apt-get update   && apt-get install -y python3-pip python3-dev   && cd /usr/local/bin   && ln -s /usr/bin/python3 python   && pip3 install --upgrade pip
 ---> Running in 6fbbdf6f27e2
Ign:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Ign:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
Ign:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Ign:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
Ign:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Ign:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
Ign:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Err:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
  Temporary failure resolving 'security.ubuntu.com'
Ign:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Ign:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Ign:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Ign:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Ign:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Err:1 http://archive.ubuntu.com/ubuntu jammy InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Err:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Err:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/jammy/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/jammy-updates/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/jammy-backports/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://security.ubuntu.com/ubuntu/dists/jammy-security/InRelease  Temporary failure resolving 'security.ubuntu.com'
W: Some index files failed to download. They have been ignored, or old ones used instead.
Reading package lists...
Building dependency tree...
Reading state information...
E: Unable to locate package python3-pip
E: Unable to locate package python3-dev
The command '/bin/sh -c apt-get update   && apt-get install -y python3-pip python3-dev   && cd /usr/local/bin   && ln -s /usr/bin/python3 python   && pip3 install --upgrade pip' returned a non-zero code: 100
```
