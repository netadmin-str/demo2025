# Настраиваем IP адреса

## ISP

```
configure

int gi1/0/3
ip address 172.16.4.1/28
ip firewall disable
no shutdown

int gi1/0/2
ip address 172.16.5.1/28
ip firewall disable
no shutdown

commit
confirm
```

<img src="01.png" width='600'>

## HQ-RTR - EcoRouter

Создаем сущность интерфейса и назначаем IP

```
int TO-ISP
ip address 172.16.4.2/28
no shutdown
```

Привязываем созданный интерфейс к физическому протоколу

1. Заходим в `port ge0`
2. Создаем service-instance `service-instance SI-ISP`
3. Указываем, что кадры на этом интерфейсе будут без тега `encapsulation untagged`
4. Привязываем сущьность интрефейса к порту `connect ip interface TO-ISP`

```
port ge0
service-instance SI-ISP
encapsulation untagged
connect ip interface TO-ISP
```

Создаем интерфейсы, которые будут обрабатывать трафик vlan 100, 200, 999

```
interface HQ-SRV
 ip mtu 1500
 ip address 192.168.0.1/26
!
interface HQ-CLI
 ip mtu 1500
 ip address 192.168.0.65/28
!
interface HQ-MGMT
 ip mtu 1500
 ip address 192.168.0.81/29
!
```

Заходим на порт и создаем для каждой `vlan` свой `service-instance`.

1. Заходим в `port ge1`.
2. Создаем service-instance `service-instance ge1/vlan100`
3. Указываем инкапсуляцию для `100 vlan`
4. Чтобы кадры из этого интерфейса выходили с тегом задаем настройку `rewrite pop 1`
5. Привязываем сущьность интрефейса к порту `connect ip interface HQ-SRV`

Проделываем эти действия для vlan 200 и 999

```
port ge1
 mtu 9234
 service-instance ge1/vlan100
  encapsulation dot1q 100
  rewrite pop 1
  connect ip interface HQ-SRV
 service-instance ge1/vlan200
  encapsulation dot1q 200
  rewrite pop 1
  connect ip interface HQ-CLI
 service-instance ge1/vlan999
  encapsulation dot1q 999
  rewrite pop 1
  connect ip interface HQ-MGMT
```

<img src="02.png" width='600'>

Создаем GRE туннель

```
interface tunnel.1
 ip mtu 1400
 ip address 172.16.1.1/30
 ip tunnel 172.16.4.2 172.16.5.2 mode gre
```

Задаем маршрут по умолчанию в сторону ISP

```
ip route 0.0.0.0/0 172.16.4.1
```

## HQ-SRV

```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options
```

```
echo 192.168.0.2/26 > /etc/net/ifaces/ens192/ipv4address
```

```
echo default via 192.168.0.1 > /etc/net/ifaces/ens192/ipv4route
```

```
systemctl restart network
```

## HQ-CLI

<img src="04.png" width='600'>
<img src="05.png" width='600'>
<img src="06.png" width='600'>
<img src="07.png" width='600'>

## BR-RTR

```
configure

interface gigabitethernet 1/0/1
  ip firewall disable
  ip address 172.16.5.2/28
  no shutdown
exit
interface gigabitethernet 1/0/2
  ip firewall disable
  ip address 192.168.1.1/27
  no shutdown
exit
tunnel gre 1
  ttl 16
  mtu 1400
  ip firewall disable
  local address 172.16.5.2
  remote address 172.16.4.2
  ip address 172.16.1.2/30
  enable
exit
ip route 0.0.0.0/0 172.16.5.1

commit
confirm
```

## BR-SRV

```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options
```

```
echo 192.168.1.2/26 > /etc/net/ifaces/ens192/ipv4address
```

```
echo default via 192.168.1.1 > /etc/net/ifaces/ens192/ipv4route
```

```
systemctl restart network
```

```
ip address
```

<img src="03.png" width='600'>

