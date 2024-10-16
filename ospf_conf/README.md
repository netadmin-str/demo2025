# Настраиваем OSPF

## HQ-RTR

```
router ospf 1
 network 172.16.1.0 0.0.0.3 area 0.0.0.0
 network 192.168.0.0 0.0.0.255 area 0.0.0.0
!
interface tunnel.1
 ip ospf network point-to-point
```

## BR-RTR

```
router ospf 1
  area 0.0.0.0
    enable
  exit
  enable
exit

interface gigabitethernet 1/0/2
  ip ospf instance 1
  ip ospf
exit
tunnel gre 1
  ip ospf instance 1
  ip ospf network point-to-point
  ip ospf
exit

commit
confirm
```

## Проверка

### HQ-RTR

```
sh ip ospf neighbor 
sh ip ospf interface brief
sh ip route
```

<img src="01.png" width='600'>

### BR-RTR

```
sh ip ospf neighbor
sh ip route
```

<img src="02.png" width='600'>

```
sh ip ospf interface
```

<img src="03.png" width='600'>