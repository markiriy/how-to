# how-to
![image](https://github.com/markiriy/how-to/assets/124806098/ca7024c1-aadb-48f0-9707-e317974d8094)


# БАЗА
sysctl -w net.ipv4.ip_forward=1 >> /etc/sysctl.conf
sysctl -p

# Routing
root@isp:~# nano /etc/network/interfaces
up ip route add 172.16.10.0/26 via 5.5.5.6
up ip route add 192.168.10.0/28 via 6.6.6.7

# GRE
br-r  gre            hq-r  gre
      gre1                 gre1
      ens192               ens192
      6.6.6.7              5.5.5.6
      5.5.5.6              6.6.6.7
      10.5.5.2/30          10.5.5.1/30
nmcli connection modify gre ip-tunnel.ttl 64

# IPSEC
apt install libreswan -y
hq: nano /etc/ipsec.d/wsr.conf
    conn wsr
      auto=start
      authby=secret
      type=tunnel
      ike=aes-sha1;modp2048
      left=10.5.5.1
      leftprotoport=gre
      right=10.5.5.2
      rightprotoport=gre
nano /etc/ipsec.d/wsr.secrets
10.5.5.1 10.5.5.2 : PSK "P@$$w0rd"

br: nano /etc/ipsec.d/wsr.conf
conn wsr
      auto=start
      authby=secret
      type=tunnel
      ike=aes-sha1;modp2048
      left=10.5.5.2
      leftprotoport=gre
      right=10.5.5.1
      rightprotoport=gre
nano /etc/ipsec.d/wsr.secrets
10.5.5.2 10.5.5.1 : PSK "P@$$w0rd"

# DHCP
![image](https://github.com/markiriy/how-to/assets/124806098/ddb15c86-78bc-4d6d-9706-1c4044c3f5cd)
![image](https://github.com/markiriy/how-to/assets/124806098/61338d70-7a95-4914-bb5d-dde2c3ab781e)
