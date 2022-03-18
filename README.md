# How to setup icmptunnel

## Prerequisites:

* server = 1.2.3.4
* tunnel = 10.1.2.0/24
* preshared key = VERY_SECRET_PASSWORD

## Binary (client and server):

* cd /usr/local/sbin
* wget https://github.com/albertzak/hanstunnel/raw/master/bin/hans-ubuntu
* chmod +x hans-ubuntu

## Service on server side:

* nano /etc/systemd/system/icmptunnel.service
* systemctl start icmptunnel
* systemctl enable icmptunnel
* journalctl -xeu icmptunnel

## icmptunnel.service

```
[Unit]
Description = https://github.com/albertzak/hanstunnel
After = network-online.target
Wants = network-online.target

[Service]
ExecStart = /usr/local/sbin/hans-ubuntu -s 10.1.2.0 -r -p VERY_SECRET_PASSWORD -u nobody -f -v
Restart = on-failure

[Install]
WantedBy = multi-user.target
```

## Client side:

* nano /sbin/ifconfig
* chmod +x /sbin/ifconfig

## Fake /sbin/ifconfig:

```
#!/bin/sh

case "$2" in
   mtu    ) ip link set "$1" mtu "$3" ;;
   [1-9]* ) ip addr add "$2/24" dev "$1" && ip link set "$1" up ;;
   *      ) echo "Bad params" ;;
esac
```

## Run client:

* sudo hans-ubuntu -c 1.2.3.4 -p VERY_SECRET_PASSWORD -u nobody -f -v
* ip addr list dev tun0
* ip route list dev tun0
* ping 10.1.2.1
