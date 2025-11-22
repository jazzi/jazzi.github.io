---
layout: post
---

[CryMPD](https://github.com/mamantoha/cryMPD/) is  minimalistic web-based MPD client while [MPD](https://www.musicpd.org) is a powerful yet simple server-client music server. However there is no Crympd package available in Debian, thankfully it offers a standalone binary, just download it and give it excution right.

+ **Operation System**: Debian Bookworm
+ **cryMPD**: 0.4.0

## Download from github

Firstly download [crympd_v0.4.0_linux-amd64](https://github.com/mamantoha/cryMPD/releases/tag/v0.4.0)

```
`sudo mv crympd_v0.4.0_linux-amd64 /usr/local/bin/crympd` # Move package to new location
`sudo chown root /usr/local/bin/crympd` # Change owner to root
`sudo chmod 700 /usr/local/bin/crympd` # Add x
`sudo systemctl edit --force --full crympd.service` # Create service file and modify as below
```

```
[Unit]
Description=A minimalistic web-based MPD client
Wants=network.target

[Service]
ExecStart=/usr/local/bin/crympd
Restart=always

[Install]
WantedBy=multi-user.target
```

## Enable & Start the service

`sudo systemctl enable --now crympd.service`
 
## Debug errors

`sudo systemctl status crympd.service`

`sudo journalctl -u crympd.service -f`


## Change Crympd to Hympd

Crympd doesn't have the function of choosing files, so replace it with [hympd](https://github.com/cortsf/hympd) and add **hympd.service** file into */etc/systemd/system/*.

```
[Unit]
Description=A minimalistic web-based MPD client
Wants=network.target

[Service]
ExecStart=/usr/local/bin/hympd --port 3001
Restart=always

[Install]
WantedBy=multi-user.target
```
