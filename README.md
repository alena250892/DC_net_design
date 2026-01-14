Цели:

I: Собрать топологию CLOS

II: Распределить адресное пространство для базовой сети

III: Зафиксировать в документации план работ, адресное пространство, схему сети, настройки

Схема сети
<img width="736" height="492" alt="image" src="https://github.com/user-attachments/assets/7a1e447c-6c3d-419b-ae9b-769594d80846" />

Адресное пространство:

interfaces Loopback:
SPINE1 10.0.1.1/32
SPINE2 10.0.1.2/32
leaf1 10.1.1.1/32
leaf2 10.1.1.2/32
leaf3 10.1.1.3/32

P2P 
SPINE1-LEAF1 10.2.1.0/31
SPINE1-LEAF2 10.2.1.2/31
SPINE1-LEAF3 10.2.1.4/31
SPINE2-LEAF1 10.2.2.0/31
SPINE2-LEAF2 10.2.2.2/31
SPINE2-LEAF3 10.2.2.4/31

Настройки и проверка связности:
Spine 1

interface Ethernet1
   no switchport
   ip address 10.2.1.0/31
!
interface Ethernet2
   no switchport
   ip address 10.2.1.2/31
!
interface Ethernet3
   no switchport
   ip address 10.2.1.4/31
!
interface Loopback0
   ip address 10.0.1.1/32

   checking 
    to leaf1
   SPINE1#ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 72(100) bytes of data.
80 bytes from 10.2.1.1: icmp_seq=1 ttl=64 time=241 ms
80 bytes from 10.2.1.1: icmp_seq=2 ttl=64 time=234 ms
80 bytes from 10.2.1.1: icmp_seq=3 ttl=64 time=228 ms
80 bytes from 10.2.1.1: icmp_seq=4 ttl=64 time=230 ms
80 bytes from 10.2.1.1: icmp_seq=5 ttl=64 time=209 ms

--- 10.2.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 61ms
rtt min/avg/max/mdev = 209.333/228.782/241.236/10.634 ms, pipe 5, ipg/ewma 15.274/234.278 ms
    
    to leaf2

    SPINE1#ping 10.2.1.3
PING 10.2.1.3 (10.2.1.3) 72(100) bytes of data.
80 bytes from 10.2.1.3: icmp_seq=1 ttl=64 time=15.8 ms
80 bytes from 10.2.1.3: icmp_seq=2 ttl=64 time=20.0 ms
80 bytes from 10.2.1.3: icmp_seq=3 ttl=64 time=14.1 ms
80 bytes from 10.2.1.3: icmp_seq=4 ttl=64 time=32.1 ms
80 bytes from 10.2.1.3: icmp_seq=5 ttl=64 time=36.2 ms

--- 10.2.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 70ms
rtt min/avg/max/mdev = 14.143/23.675/36.212/8.872 ms, pipe 2, ipg/ewma 17.513/20.369 ms

to leaf3

SPINE1#ping 10.2.1.5
PING 10.2.1.5 (10.2.1.5) 72(100) bytes of data.
80 bytes from 10.2.1.5: icmp_seq=1 ttl=64 time=80.3 ms
80 bytes from 10.2.1.5: icmp_seq=2 ttl=64 time=73.0 ms
80 bytes from 10.2.1.5: icmp_seq=3 ttl=64 time=71.2 ms
80 bytes from 10.2.1.5: icmp_seq=4 ttl=64 time=63.2 ms
80 bytes from 10.2.1.5: icmp_seq=5 ttl=64 time=59.0 ms

--- 10.2.1.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 59.059/69.391/80.304/7.499 ms, pipe 5, ipg/ewma 12.698/74.313 ms

Spine 2
hostname SPINE2
!
interface Ethernet1
   no switchport
   ip address 10.2.2.0/31
!
interface Ethernet2
   no switchport
   ip address 10.2.2.2/31
!
interface Ethernet3
   no switchport
   ip address 10.2.2.4/31
!
interface Loopback0
   ip address 10.0.1.2/32
!
checking
to leaf1

SPINE2#ping 10.2.2.1
PING 10.2.2.1 (10.2.2.1) 72(100) bytes of data.
80 bytes from 10.2.2.1: icmp_seq=1 ttl=64 time=75.7 ms
80 bytes from 10.2.2.1: icmp_seq=2 ttl=64 time=64.4 ms
80 bytes from 10.2.2.1: icmp_seq=3 ttl=64 time=62.1 ms
80 bytes from 10.2.2.1: icmp_seq=4 ttl=64 time=58.8 ms
80 bytes from 10.2.2.1: icmp_seq=5 ttl=64 time=51.7 ms

--- 10.2.2.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 53ms
rtt min/avg/max/mdev = 51.784/62.621/75.785/7.854 ms, pipe 5, ipg/ewma 13.450/68.687 ms
to leaf2
SPINE2#ping 10.2.2.3
PING 10.2.2.3 (10.2.2.3) 72(100) bytes of data.
80 bytes from 10.2.2.3: icmp_seq=1 ttl=64 time=47.1 ms
80 bytes from 10.2.2.3: icmp_seq=2 ttl=64 time=38.3 ms
80 bytes from 10.2.2.3: icmp_seq=3 ttl=64 time=33.9 ms
80 bytes from 10.2.2.3: icmp_seq=4 ttl=64 time=27.6 ms
80 bytes from 10.2.2.3: icmp_seq=5 ttl=64 time=10.1 ms

--- 10.2.2.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 82ms
rtt min/avg/max/mdev = 10.161/31.470/47.138/12.394 ms, pipe 4, ipg/ewma 20.678/38.396 ms
to leaf3
SPINE2#ping 10.2.2.5
PING 10.2.2.5 (10.2.2.5) 72(100) bytes of data.
80 bytes from 10.2.2.5: icmp_seq=1 ttl=64 time=36.7 ms
80 bytes from 10.2.2.5: icmp_seq=2 ttl=64 time=26.8 ms
80 bytes from 10.2.2.5: icmp_seq=3 ttl=64 time=24.0 ms
80 bytes from 10.2.2.5: icmp_seq=4 ttl=64 time=20.1 ms
80 bytes from 10.2.2.5: icmp_seq=5 ttl=64 time=12.1 ms

--- 10.2.2.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 72ms
rtt min/avg/max/mdev = 12.176/23.990/36.739/8.068 ms, pipe 4, ipg/ewma 18.206/29.810 ms

LEAF1

!
hostname LEAF1
!
interface Ethernet2
   no switchport
   ip address 10.2.2.1/31
!
interface Ethernet3
   no switchport
   ip address 10.2.1.1/31
!
interface Loopback0
   ip address 10.1.1.1/32

CHEKING
TO SPINE 2
LEAF1#ping 10.2.2.0
PING 10.2.2.0 (10.2.2.0) 72(100) bytes of data.
80 bytes from 10.2.2.0: icmp_seq=1 ttl=64 time=11.4 ms
80 bytes from 10.2.2.0: icmp_seq=2 ttl=64 time=8.10 ms
80 bytes from 10.2.2.0: icmp_seq=3 ttl=64 time=12.8 ms
80 bytes from 10.2.2.0: icmp_seq=4 ttl=64 time=9.53 ms
80 bytes from 10.2.2.0: icmp_seq=5 ttl=64 time=8.31 ms

--- 10.2.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 8.100/10.046/12.834/1.837 ms, pipe 2, ipg/ewma 13.068/10.702 ms
TO SPINE 1
LEAF1#ping 10.2.1.0
PING 10.2.1.0 (10.2.1.0) 72(100) bytes of data.
80 bytes from 10.2.1.0: icmp_seq=1 ttl=64 time=19.0 ms
80 bytes from 10.2.1.0: icmp_seq=2 ttl=64 time=18.2 ms
80 bytes from 10.2.1.0: icmp_seq=3 ttl=64 time=12.4 ms
80 bytes from 10.2.1.0: icmp_seq=4 ttl=64 time=19.1 ms
80 bytes from 10.2.1.0: icmp_seq=5 ttl=64 time=16.0 ms

--- 10.2.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 68ms
rtt min/avg/max/mdev = 12.496/17.007/19.130/2.513 ms, pipe 2, ipg/ewma 17.245/18.012 ms

leaf2
hostname LEAF2
!
interface Ethernet1
   no switchport
   ip address 10.2.1.3/31
!
interface Ethernet2
   no switchport
   ip address 10.2.2.3/31
!
interface Loopback0
   ip address 10.1.1.2/32
!
CHEKING
TO SPINE1
LEAF2#ping  10.2.1.2
PING 10.2.1.2 (10.2.1.2) 72(100) bytes of data.
80 bytes from 10.2.1.2: icmp_seq=1 ttl=64 time=11.2 ms
80 bytes from 10.2.1.2: icmp_seq=2 ttl=64 time=9.95 ms
80 bytes from 10.2.1.2: icmp_seq=3 ttl=64 time=7.60 ms
80 bytes from 10.2.1.2: icmp_seq=4 ttl=64 time=7.53 ms
80 bytes from 10.2.1.2: icmp_seq=5 ttl=64 time=10.1 ms

--- 10.2.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 63ms
rtt min/avg/max/mdev = 7.537/9.291/11.209/1.469 ms, ipg/ewma 15.788/10.225 ms
TO SPINE2
LEAF2#ping 10.2.2.2
PING 10.2.2.2 (10.2.2.2) 72(100) bytes of data.
80 bytes from 10.2.2.2: icmp_seq=1 ttl=64 time=10.1 ms
80 bytes from 10.2.2.2: icmp_seq=2 ttl=64 time=6.11 ms
80 bytes from 10.2.2.2: icmp_seq=3 ttl=64 time=6.35 ms
80 bytes from 10.2.2.2: icmp_seq=4 ttl=64 time=8.06 ms
80 bytes from 10.2.2.2: icmp_seq=5 ttl=64 time=10.3 ms

--- 10.2.2.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 6.118/8.218/10.382/1.818 ms, ipg/ewma 11.544/9.268 ms

LEAF3

hostname LEAF3
!
interface Ethernet1
   no switchport
   ip address 10.2.1.5/31
!
interface Ethernet2
   no switchport
   ip address 10.2.2.5/31
!
interface Loopback0
   ip address 10.1.1.3/32
!
CHEKING
TO SPINE1
LEAF3#ping 10.2.1.4
PING 10.2.1.4 (10.2.1.4) 72(100) bytes of data.
80 bytes from 10.2.1.4: icmp_seq=1 ttl=64 time=15.6 ms
80 bytes from 10.2.1.4: icmp_seq=2 ttl=64 time=12.9 ms
80 bytes from 10.2.1.4: icmp_seq=3 ttl=64 time=11.7 ms
80 bytes from 10.2.1.4: icmp_seq=4 ttl=64 time=8.11 ms
80 bytes from 10.2.1.4: icmp_seq=5 ttl=64 time=8.46 ms

--- 10.2.1.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 58ms
rtt min/avg/max/mdev = 8.110/11.384/15.642/2.829 ms, pipe 2, ipg/ewma 14.562/13.323 ms
TO SPINE2
LEAF3#ping 10.2.2.4
PING 10.2.2.4 (10.2.2.4) 72(100) bytes of data.
80 bytes from 10.2.2.4: icmp_seq=1 ttl=64 time=10.5 ms
80 bytes from 10.2.2.4: icmp_seq=2 ttl=64 time=13.5 ms
80 bytes from 10.2.2.4: icmp_seq=3 ttl=64 time=14.2 ms
80 bytes from 10.2.2.4: icmp_seq=4 ttl=64 time=8.21 ms
80 bytes from 10.2.2.4: icmp_seq=5 ttl=64 time=10.2 ms

--- 10.2.2.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 56ms
rtt min/avg/max/mdev = 8.212/11.378/14.222/2.227 ms, pipe 2, ipg/ewma 14.126/10.887 ms
