## Домашнее задание №2. Проектирование адресного пространства:

### Цели:
* Собрать схему CLOS;
* Распределить адресное пространство.

### Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

#### 1) Соберете топологию CLOS, как на схеме:
![This is an alt text.](Ex_Scheme1.PNG "This is a network topology.")
#### 2) Распределите адресное пространство для Underlay сети;
#### 3) Зафиксируете в документации план работ, адресное пространство, схему сети, настройки (если перенесли на оборудование).

# Выполнение:

## План работ:

1) Выделить адресное пространство для интерфейсов Loopback;
2) Выделить адресное пространство для интерфейсов P-2-P;
3) Собрать схему сети согласно топологии представленной в задании;
4) Назначить адреса на сооветствующие интерфейсы;
5) Настроить протокол динамической маршрутизации Underlay (учесть рекомендации);
6) Проверить связность между адресами на Loopback-интерфейсах.
7) Листинг команд для проверки корректной работы сети.
8) Конфигурации устройств.

## Адресное пространство:

На этот раз будем использовать протокол IPv6. Количество адресов для интерфейсов Loopback берем из расчета максимального рекомендованного количества устройств на один Site - 1024.
Количество адресов для линков между устройствами берез из расчета максимально возможного их количества - 8 на каждый Leaf-коммутатор.

| Назначение   | IP Network      | IP Range      | Number of Hosts |
| ------------ |:---------------:|:---------------:|:---------------:|
| Loopbacks    | fd00::x/118  | fd00::0000 - fd00::03ff  | 1024 |
| P-2-P links  | fd00::8000/113  | fd00::8000 - fd00::ffff  | 32768 |
| Link-local  | fe80::8000/113(64) | fe80::8000 - fe80::ffff | 32768 |

* Номер DC в адресный план не заложено, для последующих инсталляций берутся следующий по порядку диапазон адресов;
* x - порядковый номер коммутатора, при этом младшие адреса используются Spine-коммутаторами, последующие - Leaf-коммутаторами;
* Адреса для P-2-P интерфейсов между Spine- и Leaf-коммутаторами вибираются из диапазона с маской /127 и назначаются в порядке следования линков по топологии сети. При этом младший адрес назначается со стороны Spine-коммутатора, а старший - со стороны Leaf-коммутаторы;
* Адреса Link-local назначаются по из соображений назначенных адресов на P-2-P линках по последнему хекстету.


## Cхема сети:
![This is an alt text.](Scheme2.PNG "This is a network topology.")
## Настройки сети:
```
S1#show running-config
! Command: show running-config
! device: S1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname S1
!
spanning-tree mode mstp
!
interface Ethernet1
   no switchport
   ip address 169.254.11.0/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet2
   no switchport
   ip address 169.254.21.0/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet3
   no switchport
   ip address 169.254.31.0/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 169.254.1.1/32
!
interface Management1
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0000.0000.0001.00
   is-type level-2
   redistribute connected
   !
   address-family ipv4 unicast
!
```
```
S2#show running-config
! Command: show running-config
! device: S2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname S2
!
spanning-tree mode mstp
!
interface Ethernet1
   no switchport
   ip address 169.254.12.0/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet2
   no switchport
   ip address 169.254.22.0/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet3
   no switchport
   ip address 169.254.32.0/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 169.254.1.2/32
!
interface Management1
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0000.0000.0002.00
   is-type level-2
   redistribute connected
   !
   address-family ipv4 unicast
!
end
```
```
L1#show running-config
! Command: show running-config
! device: L1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname L1
!
spanning-tree mode mstp
!
interface Ethernet1
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   no switchport
   ip address 169.254.11.1/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet8
   no switchport
   ip address 169.254.12.1/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Loopback0
   ip address 169.254.1.3/32
!
interface Management1
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0000.0000.0011.00
   is-type level-2
   redistribute connected
   !
   address-family ipv4 unicast
!
end
```
```
L2#show running-config
! Command: show running-config
 ! device: L2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname L2
!
spanning-tree mode mstp
!
interface Ethernet1
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   no switchport
   ip address 169.254.21.1/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet8
   no switchport
   ip address 169.254.22.1/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Loopback0
   ip address 169.254.1.4/32
!
interface Management1
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0000.0000.0012.00
   is-type level-2
   redistribute connected
   !
   address-family ipv4 unicast
!
end
```
```
L3#show running-config
! Command: show running-config
! device: L3 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname L3
!
spanning-tree mode mstp
!
interface Ethernet1
!
interface Ethernet2
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   no switchport
   ip address 169.254.31.1/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Ethernet8
   no switchport
   ip address 169.254.32.1/31
   isis enable UNDERLAY
   isis network point-to-point
!
interface Loopback0
   ip address 169.254.1.5/32
!
interface Management1
!
ip routing
!
router isis UNDERLAY
   net 49.0001.0000.0000.0013.00
   is-type level-2
   redistribute connected
   !
   address-family ipv4 unicast
!
end
```

## Проверка работы сети:

```
L1#show ip route | begin Gateway
Gateway of last resort is not set

 I L2     169.254.1.1/32 [115/11] via 169.254.11.0, Ethernet7
 I L2     169.254.1.2/32 [115/11] via 169.254.12.0, Ethernet8
 C        169.254.1.3/32 is directly connected, Loopback0
 I L2     169.254.1.4/32 [115/21] via 169.254.11.0, Ethernet7
                                  via 169.254.12.0, Ethernet8
 I L2     169.254.1.5/32 [115/21] via 169.254.11.0, Ethernet7
                                  via 169.254.12.0, Ethernet8
 C        169.254.11.0/31 is directly connected, Ethernet7
 C        169.254.12.0/31 is directly connected, Ethernet8
 I L2     169.254.21.0/31 [115/20] via 169.254.11.0, Ethernet7
 I L2     169.254.22.0/31 [115/20] via 169.254.12.0, Ethernet8
 I L2     169.254.31.0/31 [115/20] via 169.254.11.0, Ethernet7
 I L2     169.254.32.0/31 [115/20] via 169.254.12.0, Ethernet8

L1#ping 169.254.1.4 repeat 1
PING 169.254.1.4 (169.254.1.4) 72(100) bytes of data.
80 bytes from 169.254.1.4: icmp_seq=1 ttl=63 time=20.4 ms

--- 169.254.1.4 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 20.424/20.424/20.424/0.000 ms

L1#ping 169.254.1.5 repeat 1
PING 169.254.1.5 (169.254.1.5) 72(100) bytes of data.
80 bytes from 169.254.1.5: icmp_seq=1 ttl=63 time=21.9 ms

--- 169.254.1.5 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 21.967/21.967/21.967/0.000 ms
```
