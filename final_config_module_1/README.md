# Финальная конфигурация модуля 1

## ISP

```
hostname ISP

object-group network LOCAL_NET
  ip address-range 172.16.5.1-172.16.5.2
  ip address-range 172.16.4.1-172.16.4.2
exit

syslog max-files 3
syslog file-size 512
syslog file tmpsys:syslog/default
  severity info
exit
syslog console
  virtual-serial
exit

username admin
  password encrypted $6$3f2x9RmHkMX7Rv3u$g5RMUcviJt7ERyUTLVc6JKDeSXEqYqiXgscIPb/mrWFf/6KMATYZvuFksLvp4NJw5n3ivVN7zTQWsxoQDh7Fq0
exit

domain lookup enable

interface gigabitethernet 1/0/1
  ip firewall disable
  ip address dhcp
exit
interface gigabitethernet 1/0/2
  ip firewall disable
  ip address 172.16.5.1/28
exit
interface gigabitethernet 1/0/3
  ip firewall disable
  ip address 172.16.4.1/28
exit
security passwords default-expired
nat source
  ruleset SNAT
    to interface gigabitethernet 1/0/1
    rule 1
      match source-address LOCAL_NET
      action source-nat interface
      enable
    exit
  exit
exit

ip ssh server

ntp enable
ntp broadcast-client enable

licence-manager
  host address elm.eltex-co.ru
exit
```

## HQ-RTR

```
hostname HQ-RTR
!
hw mgmt ip 192.168.255.1/24
!
ip pool HQ-NET200 1
 range 192.168.0.66-192.168.0.70
!
dhcp-server 1
 lease 86400
 mask 255.255.255.0
 pool HQ-NET200 1
  dns 192.168.0.2
  domain-name au-team.irpo
  gateway 192.168.0.65
  mask 255.255.255.240
!
router ospf 1
 ospf router-id 1.1.1.1
 network 172.16.1.0 0.0.0.3 area 0.0.0.0
 network 192.168.0.0 0.0.0.255 area 0.0.0.0
!
ip route 0.0.0.0/0 172.16.4.1
!
ntp timezone utc+5
!
port ge0
 mtu 9234
 service-instance SI-ISP
  encapsulation untagged
!
port ge1
 mtu 9234
 service-instance ge1/vlan100
  encapsulation dot1q 100
  rewrite pop 1
 service-instance ge1/vlan200
  encapsulation dot1q 200
  rewrite pop 1
 service-instance ge1/vlan999
  encapsulation dot1q 999
  rewrite pop 1
!
interface tunnel.1
 ip mtu 1400
 ip address 172.16.1.1/30
 ip tunnel 172.16.4.2 172.16.5.2 mode gre
 ip ospf network point-to-point
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 0x638a577c1da08438
!
interface TO-ISP
 ip mtu 1500
 connect port ge0 service-instance SI-ISP
 ip nat outside
 ip address 172.16.4.2/28
!
interface HQ-SRV
 ip mtu 1500
 connect port ge1 service-instance ge1/vlan100
 ip nat inside
 ip address 192.168.0.1/26
!
interface HQ-CLI
 ip mtu 1500
 connect port ge1 service-instance ge1/vlan200
 dhcp-server 1
 ip nat inside
 ip address 192.168.0.65/28
!
interface HQ-MGMT
 ip mtu 1500
 connect port ge1 service-instance ge1/vlan999
 ip nat inside
 ip address 192.168.0.81/29
!
ip nat pool NAT_POOL 192.168.0.1-192.168.0.254
!
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface TO-ISP
!
end
```

## BR-RTR

```
hostname BR-RTR

object-group network LOCAL_NET
  ip address-range 192.168.1.1-192.168.1.30
exit

username admin
  password encrypted $6$g3LiECReGo5iYXz4$J5jU54LCgu0MURzF1nNkrKUqXQ6fIFx9hATS8FEv8dvMv.ocw/1/xnc7lv4AzL8w3HlqY6vyTaAyps4ciwLki/
exit
username net_admin
  password encrypted $6$ESlbJmEXyb3CWuDM$CuAk7QHUO11C7fAC6kVYjr59vjNTWOg/WhZBPRV7v5FzCPif762QVO/9lU1u.zR6Vm95P.g5Z7SIC04Mi9QXs1
  privilege 15
exit

domain lookup enable

key-chain ospfkey
  key 1
    key-string ascii-text encrypted B8B10E62E25F1AAE
  exit
exit

router ospf 1
  router-id 2.2.2.2
  area 0.0.0.0
    enable
  exit
  enable
exit

interface gigabitethernet 1/0/1
  ip firewall disable
  ip address 172.16.5.2/28
exit
interface gigabitethernet 1/0/2
  ip firewall disable
  ip address 192.168.1.1/27
  ip ospf instance 1
  ip ospf
exit
tunnel gre 1
  ttl 64
  mtu 1400
  ip firewall disable
  local address 172.16.5.2
  remote address 172.16.4.2
  ip address 172.16.1.2/30
  ip ospf instance 1
  ip ospf authentication key-chain ospfkey
  ip ospf authentication algorithm md5
  ip ospf network point-to-point
  ip ospf
  enable
exit

security passwords default-expired
nat source
  ruleset SNAT
    to interface gigabitethernet 1/0/1
    rule 1
      match source-address LOCAL_NET
      action source-nat interface
      enable
    exit
  exit
exit

ip route 0.0.0.0/0 172.16.5.1

ip ssh server

clock timezone gmt +5

```

## HQ-SRV

### hostname

```
hostnamectl set-hostname HQ-SRV
```

### IP
```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options

echo 192.168.0.2/26 > /etc/net/ifaces/ens192/ipv4address

echo default via 192.168.0.1 > /etc/net/ifaces/ens192/ipv4route

systemctl restart network
```

### SSH
```
useradd -m -u 1010 sshuser
passwd sshuser

echo sshuser ALL=(ALL) NOPASSWD:ALL > /etc/sudoers.d/sshuser

cp -a /etc/openssh/sshd_config /etc/openssh/sshd_config.bak

rm /etc/openssh/sshd_config

echo "
Port 2024
MaxAuthTries 3

ssh-rsa-cert-v01@openssh.com,ssh-rsa,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp521,ecdsa-sha2-nistp384,ecdsa-sha2-nistp256

authorized_keys .ssh/authorized_keys2

Banner /etc/openssh/banner

Subsystem       sftp    /usr/lib/openssh/sftp-server

AcceptEnv LANG LANGUAGE LC_ADDRESS LC_ALL LC_COLLATE LC_CTYPE
AcceptEnv LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES LC_MONETARY
AcceptEnv LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE LC_TIME
" > /etc/openssh/sshd_config

echo "
******************************************************
*                     WARNING!                       *
*                                                    *
*           Access only authorized users!            *
*                                                    *
*  If you not authorized user, close this session!   *
*                                                    *
******************************************************
" > /etc/openssh/banner

systemctl restart sshd
```

### BIND
```
apt-get update
apt-get install bind bind-utils
```

```
cp -a /var/lib/bind/etc/options.conf /var/lib/bind/etc/options.conf.bak

rm /var/lib/bind/etc/options.conf

echo "
options {
        version \"unknown\";
        directory \"/etc/bind/zone\";
        dump-file \"/var/run/named/named_dump.db\";
        statistics-file \"/var/run/named/named.stats\";
        recursing-file \"/var/run/named/named.recursing\";
        secroots-file \"/var/run/named/named.secroots\";
        pid-file none;

        forwarders { 77.88.8.8; };
        recursion yes;

        allow-query { 127.0.0.1; 192.168.0.0/24; 192.168.1.0/24; };


        allow-query-cache { 127.0.0.1; 192.168.0.0/24; 192.168.1.0/24; };


        allow-recursion { 127.0.0.1; 192.168.0.0/24; 192.168.1.0/24; };


};

logging {


};
" > /var/lib/bind/etc/options.conf

rm /etc/net/ifaces/ens192/resolv.conf 

cat "search au-team.irpo
nameserver 127.0.0.1" > /etc/net/ifaces/ens192/resolv.conf 

systemctl restart network
systemctl enable --now bind

cp -a /var/lib/bind/etc/local.conf /var/lib/bind/etc/local.conf.bak
rm /var/lib/bind/etc/local.conf

echo "include \"/etc/bind/rfc1912.conf\";

zone \"au-team.irpo\" {
        type master;
        file \"au-team.irpo.db\";
};

zone \"0.168.192.in-addr.arpa\" {
        type master;
        file \"au-team.irpo_rev.db\";
};" > /var/lib/bind/etc/local.conf


echo "$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                                2024102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
        IN      NS      au-team.irpo.
        IN      A       192.168.0.2
hq-rtr  IN      A       192.168.0.1
br-rtr  IN      A       192.168.1.1
hq-srv  IN      A       192.168.0.2
hq-cli  IN      A       192.168.0.66
br-srv  IN      A       192.168.1.2
moodle  IN      CNAME   hq-rtr
wiki    IN      CNAME   hq-rtr" > /var/lib/bind/etc/zone/au-team.irpo.db

chown named. /var/lib/bind/etc/zone/au-team.irpo.db

chmod 600 /var/lib/bind/etc/zone/au-team.irpo.db

echo "$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                                2024102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
        IN      NS      au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.
2       IN      PTR     hq-srv.au-team.irpo.
66      IN      PTR     hq-cli.au-team.irpo." > /var/lib/bind/etc/zone/au-team.irpo_rev.db

chown named. /var/lib/bind/etc/zone/au-team.irpo_rev.db

chmod 600 /var/lib/bind/etc/zone/au-team.irpo_rev.db

systemctl restart bind

```

### timezone

```
timedatectl set-timezone Asia/Yekaterinburg
```

## BR-SRV

### hostname

```
hostnamectl set-hostname BR-SRV
```

### IP
```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options

echo 192.168.1.2/26 > /etc/net/ifaces/ens192/ipv4address

echo default via 192.168.1.1 > /etc/net/ifaces/ens192/ipv4route

echo nameserver 192.168.0.2 > /etc/net/ifaces/ens192/resolv.conf

systemctl restart network
```

### SSH
```
useradd -m -u 1010 sshuser
passwd sshuser

echo sshuser ALL=(ALL) NOPASSWD:ALL > /etc/sudoers.d/sshuser

cp -a /etc/openssh/sshd_config /etc/openssh/sshd_config.bak

rm /etc/openssh/sshd_config

echo "
Port 2024
MaxAuthTries 3

ssh-rsa-cert-v01@openssh.com,ssh-rsa,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp521,ecdsa-sha2-nistp384,ecdsa-sha2-nistp256

authorized_keys .ssh/authorized_keys2

Banner /etc/openssh/banner

Subsystem       sftp    /usr/lib/openssh/sftp-server

AcceptEnv LANG LANGUAGE LC_ADDRESS LC_ALL LC_COLLATE LC_CTYPE
AcceptEnv LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES LC_MONETARY
AcceptEnv LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE LC_TIME
" > /etc/openssh/sshd_config

echo "
******************************************************
*                     WARNING!                       *
*                                                    *
*           Access only authorized users!            *
*                                                    *
*  If you not authorized user, close this session!   *
*                                                    *
******************************************************
" > /etc/openssh/banner

systemctl restart sshd
```