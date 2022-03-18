# How to setup icmptunnel

### Prerequisites:

* server = 1.2.3.4
* tunnel = 10.1.2.0/24
* preshared key = VERY_SECRET_PASSWORD

### Binary (client and server):

* cd /usr/local/sbin
* wget https://github.com/albertzak/hanstunnel/raw/master/bin/hans-ubuntu
* chmod +x hans-ubuntu

### Service on server side:

* nano /etc/systemd/system/icmptunnel.service
* systemctl daemon-reload
* systemctl start icmptunnel
* systemctl enable icmptunnel
* journalctl -xeu icmptunnel

### icmptunnel.service:

```
[Unit]
Description = ICMP tunnel
Docs  = https://github.com/albertzak/hanstunnel
Docs  = https://github.com/ilyaevseev/icmptunnel-setup-docs/
After = network-online.target
Wants = network-online.target

[Service]
ExecStartPre = /bin/sh -c 'echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all'
ExecStart    = /usr/local/sbin/hans-ubuntu -s 10.1.2.0 -r -p VERY_SECRET_PASSWORD -u nobody -f -v
ExecStopPost = /bin/sh -c 'echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all'
Restart = on-failure

[Install]
WantedBy = multi-user.target
```

### Prepare client side:

* nano /sbin/ifconfig
* chmod +x /sbin/ifconfig

### Fake /sbin/ifconfig:

```
#!/bin/sh

case "$2" in
   mtu    ) ip link set "$1" mtu "$3" ;;
   [1-9]* ) ip addr add "$2/24" dev "$1" && ip link set "$1" up ;;
   *      ) echo "Bad params" ;;
esac
```

### Run and test client:

* sudo hans-ubuntu -c 1.2.3.4 -p VERY_SECRET_PASSWORD -u nobody -f -v
* ip addr list dev tun0
* ip route list dev tun0
* ping 10.1.2.1
* ssh 10.1.2.1
