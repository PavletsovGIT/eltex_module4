# Лабораторная работа 3

## Первоначальная настройка

Сначала удалим лишение vlan'ы, настроим stp и поменяем название коммутатора в консоли. Корневым коммутатором назначим Layer2Switch-1.
Удаление лишних vlan'ов:
```
en
conf t
no vlan 100-300
```

Назначение корневого коммутатора (только на Layer2Switch-1):
```
en
conf t
sp v 1 r p
sp v 1 p 0
```

Изменение названия коммутатора в консоли:
```
en
conf t
hostname swX
```

Для проверки настроек, посмотрим таблицы vlan и spanning-tree на случайном коммутаторе, к примеру Layer2Switch-4:
```
sw4(config)#do sh vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi1/0, Gi1/1
1002 fddi-default                     act/unsup
1003 trcrf-default                    act/unsup
1004 fddinet-default                  act/unsup
1005 trbrf-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0
1003 trcrf 101003     4472  1005   3276   -        -    srb      0      0
1004 fdnet 101004     1500  -      -      -        ieee -        0      0
1005 trbrf 101005     4472  -      -      15       ibm  -        0      0


VLAN AREHops STEHops Backup CRF
---- ------- ------- ----------
1003 7       7       off

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------

sw4(config)#do sh sp

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    1
             Address     0c9e.245d.0000
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0c43.6cbf.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    Shr
Gi0/1               Altn BLK 4         128.2    Shr
Gi0/2               Desg FWD 4         128.3    Shr
Gi0/3               Desg FWD 4         128.4    Shr
Gi1/0               Desg FWD 4         128.5    Shr
Gi1/1               Desg FWD 4         128.6    Shr


sw4(config)#

```

## Настройка LACP

Для настройки LACP заранее уточним что все порты на L2Sw-3, L2Sw-4, L2Sw-5 будут работать в режиме агрегирования passive, все порты на L2Sw-1 в режиме агрегирования active, на L2Sw-2 все порты кроме g0/0-1, g2/0  будут работать в режиме active, а порты g0/0-1, g2/0 будут работать в режиме passive.

Сначала переходим в режим глобальной конфигурации:
```
en
conf t
```

Далее переходим в режим конфигурации интерфейсов(`g` - gigabitEth, `X`,`Y`,`Z` - числа для обозначения порта):
```
int r gX/Y-Z
```

Для настройки одного интерфейса используем(`g` - gigabitEth, `X`,`Y` - числа для обозначения порта):
```
int gX/Y
```

Для настройки работы коммутаторов по протоколу lacp используем:
```
channel-protocol lacp
```

Для настройки группы в режиме active (`X` - номер группы):
```
channel-group X mode active
```

Для настройки группы в режиме passive (`X` - номер группы):
```
channel-group X mode passive
```

Настройка L2Switch-1:
```
sw1(config)#int r g0/0-1
sw1(config-if-range)#channel-p lacp
sw1(config-if-range)#channel-g 1 m ac
Creating a port-channel interface Port-channel 1

sw1(config-if-range)#
*Dec 15 06:42:42.580: %EC-5-L3DONTBNDL2: Gi0/1 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:42:42.770: %EC-5-L3DONTBNDL2: Gi0/0 suspended: LACP currently not enabled on the remote port.
sw1(config-if-range)#
sw1(config-if-range)#int g2/0
sw1(config-if)#channel-p lacp
sw1(config-if)#channel-g 1 m ac
sw1(config-if)#int r 0/2-3
*Dec 15 06:48:25.970: %EC-5-L3DONTBNDL2: Gi2/0 suspended: LACP currently not enabled on the remote port.
sw1(config)#int r g0/2-3
sw1(config-if-range)#channel-p lacp
sw1(config-if-range)#channel-g 2 m ac
Creating a port-channel interface Port-channel 2

sw1(config-if-range)#int r g1/0-3
*Dec 15 06:49:07.708: %EC-5-L3DONTBNDL2: Gi0/2 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:49:08.261: %EC-5-L3DONTBNDL2: Gi0/3 suspended: LACP currently not enabled on the remote port.
sw1(config-if-range)#int r g1/0-1
sw1(config-if-range)#channel-p lacp
sw1(config-if-range)#channel-g 3 m ac
Creating a port-channel interface Port-channel 3

sw1(config-if-range)#int r g1/2-
*Dec 15 06:49:52.869: %EC-5-L3DONTBNDL2: Gi1/1 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:49:52.993: %EC-5-L3DONTBNDL2: Gi1/0 suspended: LACP currently not enabled on the remote port.
sw1(config-if-range)#int r g1/2-3
sw1(config-if-range)#channel-p lacp
sw1(config-if-range)#channel-g 4 m ac
Creating a port-channel interface Port-channel 4

sw1(config-if-range)#exit
sw1(config)#do
*Dec 15 06:50:27.264: %EC-5-L3DONTBNDL2: Gi1/2 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:50:27.375: %EC-5-L3DONTBNDL2: Gi1/3 suspended: LACP currently not enabled on the remote port.
sw1(config)#do wr
Building configuration...
Compressed configuration from 5846 bytes to 2232 bytes[OK]
sw1(config)#
*Dec 15 06:50:34.020: %GRUB-5-CONFIG_WRITING: GRUB configuration is being updated on disk. Please wait...
*Dec 15 06:50:34.765: %GRUB-5-CONFIG_WRITTEN: GRUB configuration was written to disk successfully.
sw1(config)#
sw1#
*Dec 15 06:50:44.159: %SYS-5-CONFIG_I: Configured from console by console
sw1#
```

Настройка L2Switch-2:
```
sw2(config)#int r g0/0-1
sw2(config-if-range)#channel-p lacp
sw2(config-if-range)#channel-g 1 m p
Creating a port-channel interface Port-channel 1

sw2(config-if-range)#int
*Dec 15 06:53:42.169: %EC-5-L3DONTBNDL2: Gi0/1 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:53:42.305: %EC-5-L3DONTBNDL2: Gi0/0 suspended: LACP currently not enabled on the remote port.
sw2(config-if-range)#int g2/0
sw2(config-if)#channe-
*Dec 15 06:53:54.226: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
sw2(config-if)#channel-p lacp
sw2(config-if)#channel-g 1 m p
sw2(config-if)#int r g0/2-3
sw2(config-if-range)#ch
*Dec 15 06:54:17.033: %EC-5-L3DONTBNDL2: Gi2/0 suspended: LACP currently not enabled on the remote port.
sw2(config-if-range)#channel-p lacp
sw2(config-if-range)#channel-g 5 m ac
Creating a port-channel interface Port-channel 5

sw2(config-if-range)#int r g1
*Dec 15 06:54:55.353: %EC-5-L3DONTBNDL2: Gi0/2 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:54:55.634: %EC-5-L3DONTBNDL2: Gi0/3 suspended: LACP currently not enabled on the remote port.
sw2(config-if-range)#int r g1/0-1
sw2(config-if-range)#channel-p lacp
sw2(config-if-range)#channel-g 6 m ac
Creating a port-channel interface Port-channel 6

sw2(config-if-range)#int r g1/2-
*Dec 15 06:55:38.677: %EC-5-L3DONTBNDL2: Gi1/0 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:55:38.812: %EC-5-L3DONTBNDL2: Gi1/1 suspended: LACP currently not enabled on the remote port.
sw2(config-if-range)#int r g1/2-3
sw2(config-if-range)#channel-p lacp
sw2(config-if-range)#channel-g 7 m ac
Creating a port-channel interface Port-channel 7

sw2(config-if-range)#exit
sw2(config)#do wr
Building configuration...
Compressed configuration from 5856 bytes to 2245 bytes[OK]
*Dec 15 06:56:15.848: %EC-5-L3DONTBNDL2: Gi1/3 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:56:15.848: %EC-5-L3DONTBNDL2: Gi1/2 suspended: LACP currently not enabled on the remote port.
*Dec 15 06:56:18.249: %GRUB-5-CONFIG_WRITING: GRUB configuration is being updated on disk. Please wait...
sw2(config)#
*Dec 15 06:56:18.998: %GRUB-5-CONFIG_WRITTEN: GRUB configuration was written to disk successfully.
sw2(config)#
```

Настройка Layer2Switch-3:
```
sw3(config)#int r g0/0-1
sw3(config-if-range)#channel-p lacp
sw3(config-if-range)#channel-g 2 m p
Creating a port-channel interface Port-channel 2

sw3(config-if-range)#int r g0/2
*Dec 15 07:00:46.820: %EC-5-L3DONTBNDL2: Gi0/0 suspended: LACP currently not enabled on the remote port.
*Dec 15 07:00:46.845: %EC-5-L3DONTBNDL2: Gi0/1 suspended: LACP currently not enabled on the remote port.
sw3(config-if-range)#int r g0/2-3
sw3(config-if-range)#channel-p lacp
sw3(config-if-range)#channel-g g
*Dec 15 07:01:08.381: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to up
sw3(config-if-range)#channel-g 5 m p
Creating a port-channel interface Port-channel 5

sw3(config-if-range)#exit
sw3(config)#do wr
Building configuration...
Compressed configuration from 5323 bytes to 2058 bytes[OK]
*Dec 15 07:01:26.019: %EC-5-L3DONTBNDL2: Gi0/3 suspended: LACP currently not enabled on the remote port.
*Dec 15 07:01:28.686: %GRUB-5-CONFIG_WRITING: GRUB configuration is being updated on disk. Please wait...
sw3(config)#
```

Подобным образом был настроен протокол LACP на Layer2Switch-4 и Layer2Switch-5.

## Проверка настройки LACP
```
sw1#sh etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator

        M - not in use, minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 4
Number of aggregators:           4

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Gi0/0(s)    Gi0/1(s)    Gi2/0(P)
2      Po2(SU)         LACP      Gi0/2(P)    Gi0/3(P)
3      Po3(SD)         LACP      Gi1/0(H)    Gi1/1(H)
4      Po4(SU)         LACP      Gi1/2(P)    Gi1/3(P)

sw1#show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode

Channel group 1 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SP      32768     0cc4.036c.0000  13s    0x0    0x1    0x1     0x3C
Gi0/1     SP      32768     0cc4.036c.0000  13s    0x0    0x1    0x2     0x3C
Gi2/0     SP      32768     0cc4.036c.0000   3s    0x0    0x1    0x201   0x3C

Channel group 2 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SP      32768     0cec.7140.0000   3s    0x0    0x2    0x1     0x3C
Gi0/3     SP      32768     0cec.7140.0000  10s    0x0    0x2    0x2     0x3C

Channel group 3 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi1/0     SP      32768     0ce5.73c4.0000  23s    0x0    0x3    0x1     0x3C
Gi1/1     SP      32768     0ce5.73c4.0000  24s    0x0    0x3    0x2     0x3C

Channel group 4 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi1/2     SP      32768     0c7e.1449.0000   7s    0x0    0x4    0x1     0x3C
Gi1/3     SP      32768     0c7e.1449.0000   5s    0x0    0x4    0x2     0x3C
sw1#

```

### Проверка с помощью ping
PC1:
```
VPCS> ip 192.168.0.1
Checking for duplicate address...
VPCS : 192.168.0.1 255.255.255.0

VPCS> ping 192.168.0.3

84 bytes from 192.168.0.3 icmp_seq=1 ttl=64 time=8.993 ms
84 bytes from 192.168.0.3 icmp_seq=2 ttl=64 time=5.558 ms
84 bytes from 192.168.0.3 icmp_seq=3 ttl=64 time=4.391 ms
84 bytes from 192.168.0.3 icmp_seq=4 ttl=64 time=9.890 ms
84 bytes from 192.168.0.3 icmp_seq=5 ttl=64 time=6.899 ms

VPCS>
```

PC3:
```
VPCS> ip 192.168.0.3
Checking for duplicate address...
VPCS : 192.168.0.3 255.255.255.0

VPCS>
```

## Изменение режима работы групп портов произвольных коммутаторов

Для примера изменим режим работы группы каналов 2 на L2Switch-3:
```
sw3>en
sw3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
sw3(config)#int r g0/0-1
sw3(config-if-range)#channel-g 2 m ac
sw3(config-if-range)#
*Dec 15 07:22:35.769: %LINK-3-UPDOWN: Interface Port-channel2, changed state to down
*Dec 15 07:22:36.769: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to down
sw3(config-if-range)#
*Dec 15 07:22:39.696: %IDBMAN-3-INVALIDAGGPORTBANDWIDTH: Port-channel2(16 / 0) has an invalid bandwidth value of 0
*Dec 15 07:22:39.901: %IDBMAN-3-INVALIDAGGPORTBANDWIDTH: Port-channel2(16 / 0) has an invalid bandwidth value of 0
sw3(config-if-range)#
*Dec 15 07:22:41.696: %LINK-3-UPDOWN: Interface Port-channel2, changed state to up
*Dec 15 07:22:42.696: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to up
sw3(config-if-range)#show lacp neighbor
                       ^
% Invalid input detected at '^' marker.

sw3(config-if-range)#do show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode

Channel group 2 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     0c33.458e.0000  28s    0x0    0x2    0x3     0x3D
Gi0/1     SA      32768     0c33.458e.0000  15s    0x0    0x2    0x4     0x3D

Channel group 5 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     0cc4.036c.0000   6s    0x0    0x5    0x3     0x3D
Gi0/3     SA      32768     0cc4.036c.0000  19s    0x0    0x5    0x4     0x3D
sw3(config-if-range)#

```

```
sw1>en
sw1#show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode

Channel group 1 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SP      32768     0cc4.036c.0000  16s    0x0    0x1    0x1     0x3C
Gi0/1     SP      32768     0cc4.036c.0000  19s    0x0    0x1    0x2     0x3C
Gi2/0     SP      32768     0cc4.036c.0000   9s    0x0    0x1    0x201   0x3C

Channel group 2 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     0cec.7140.0000  18s    0x0    0x2    0x1     0x3D
Gi0/3     SA      32768     0cec.7140.0000  18s    0x0    0x2    0x2     0x3D

Channel group 3 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi1/0     SP      32768     0ce5.73c4.0000  26s    0x0    0x3    0x1     0x3C
Gi1/1     SP      32768     0ce5.73c4.0000   8s    0x0    0x3    0x2     0x3C

Channel group 4 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi1/2     SP      32768     0c7e.1449.0000   7s    0x0    0x4    0x1     0x3C
Gi1/3     SP      32768     0c7e.1449.0000  24s    0x0    0x4    0x2     0x3C
sw1#
```

PC1:
```
VPCS> ping 192.168.0.3

host (192.168.0.3) not reachable

VPCS>
```

PC3:
```
VPCS> ping 192.168.0.1

192.168.0.1 icmp_seq=1 timeout
192.168.0.1 icmp_seq=2 timeout
192.168.0.1 icmp_seq=3 timeout
192.168.0.1 icmp_seq=4 timeout
192.168.0.1 icmp_seq=5 timeout

VPCS>
```

## Статистика после ping
PC1:
```
VPCS> ip 192.168.0.1
Checking for duplicate address...
VPCS : 192.168.0.1 255.255.255.0

VPCS> ping 192.168.0.3

84 bytes from 192.168.0.3 icmp_seq=1 ttl=64 time=9.093 ms
84 bytes from 192.168.0.3 icmp_seq=2 ttl=64 time=8.115 ms
84 bytes from 192.168.0.3 icmp_seq=3 ttl=64 time=5.109 ms
84 bytes from 192.168.0.3 icmp_seq=4 ttl=64 time=1.847 ms
84 bytes from 192.168.0.3 icmp_seq=5 ttl=64 time=12.443 ms

VPCS>

```

PC3:
```
VPCS> ip 192.168.0.3
Checking for duplicate address...
VPCS : 192.168.0.3 255.255.255.0

VPCS> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=64 time=7.810 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=64 time=9.324 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=64 time=4.072 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=64 time=7.726 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=64 time=8.120 ms

VPCS> ping 192.168.0.5

84 bytes from 192.168.0.5 icmp_seq=1 ttl=64 time=4.142 ms
84 bytes from 192.168.0.5 icmp_seq=2 ttl=64 time=6.935 ms
84 bytes from 192.168.0.5 icmp_seq=3 ttl=64 time=5.206 ms
84 bytes from 192.168.0.5 icmp_seq=4 ttl=64 time=3.547 ms
84 bytes from 192.168.0.5 icmp_seq=5 ttl=64 time=10.997 ms
```

PC5:
```
VPCS> ip 192.168.0.5
Checking for duplicate address...
VPCS : 192.168.0.5 255.255.255.0

VPCS> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=64 time=7.587 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=64 time=7.048 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=64 time=8.186 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=64 time=2.559 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=64 time=1.564 ms

VPCS>

```