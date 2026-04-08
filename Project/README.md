## Проектирование сетевой фабрики на основе VxLAN EVPN.

### План работ:
#### 1. [Составление IP плана сети](#ip-plan)
#### 2. [Схема сети](#shema)
#### 3. [Настройка Underlay EBGP](#underlay)
#### 4. [Настройка Overlay VXLAN EVPN](#overlay)
##### ⋅⋅* [Настройка оборудования ЦОД1](#cod1)
##### ⋅⋅* [Настройка оборудования ЦОД2](#cod2)
##### ⋅⋅* [Настройка Border Leaf для связности между l2 сегментами ЦОДов](#border)
#### 5. [Настройка Multihoming ethernet сегмента в ЦОД 1](#multihoming)
#### 6. [Настройка инкапсуляции маршрута от EBGP соседа в фабрику](#encapsulation)
#### 7. [Траблшутинг](#troubleshooting)
---
## 1. IP-план фабрики <a id="ip-plan"></a>
### 🏢 ЦОД-1 (DC1)
#### Loopback (VTEP)
| Устройство    | Loopback IP        | Назначение               |
|---------------|--------------------|--------------------------|
| Spine1_1      | 10.10.1.1/32       | Router-ID, underlay loopback |
| Spine1_2      | 10.10.2.1/32       | Router-ID, underlay loopback |
| Leaf1_1       | 10.1.1.1/32        | VTEP, Router-ID           |
| Leaf1_2       | 10.1.2.1/32        | VTEP, Router-ID           |
| BorderLeaf-1  | 10.1.10.1/32       | VTEP, Router-ID           |
#### Underlay P2P-линки (/31)
| Линк (Link)           | Подсеть (Subnet)     | IP устройства (IP Assignment) |
|-----------------------|----------------------|-------------------------------|
| Spine1_1 – Leaf1_1    | 10.0.1.0/31          | Spine: .0, Leaf: .1           |
| Spine1_1 – Leaf1_2    | 10.0.1.2/31          | Spine: .0, Leaf: .1           |
| Spine1_2 – Leaf1_1    | 10.0.1.4/31          | Spine: .0, Leaf: .1           |
| Spine1_2 – Leaf1_2    | 10.0.1.6/31          | Spine: .0, Leaf: .1           |
| Spine1_1 – BorderLeaf-1 | 10.0.1.8/31        | Spine: .0, Border: .1         |
| Spine1_2 – BorderLeaf-1 | 10.0.1.10/31       | Spine: .0, Border: .1         |
#### Клиентские сети (VLAN)
| VLAN | VNI    | Подсеть          | Anycast Gateway | VRF  |
|------|--------|------------------|-----------------|------|
| 10   | 10010  | 192.168.1.0/24   | 192.168.1.1     | VRF1 |
| -    | 10020  | 192.168.2.0/24   | 192.168.2.1     | VRF1 |
#### L3 VNI
| VRF  | L3 VNI   |
|------|----------|
| VRF1 | 111111   |
---
### 🏭 ЦОД-2 (DC2)
#### Loopback (VTEP)
| Устройство    | Loopback IP        | Назначение               |
|---------------|--------------------|--------------------------|
| Spine2_1      | 10.20.1.1/32       | Router-ID, underlay loopback |
| Spine2_2      | 10.20.2.1/32       | Router-ID, underlay loopback |
| Leaf2_1       | 10.2.1.1/32        | VTEP, Router-ID           |
| Leaf2_2       | 10.2.2.1/32        | VTEP, Router-ID           |
| BorderLeaf-2  | 10.2.10.1/32       | VTEP, Router-ID           |
#### Underlay P2P-линки (/31)

| Линк (Link)           | Подсеть (Subnet)     | IP устройства (IP Assignment) |
|-----------------------|----------------------|-------------------------------|
| Spine2_1 – Leaf2_1    | 10.0.2.0/31          | Spine: .0, Leaf: .1           |
| Spine2_1 – Leaf2_2    | 10.0.2.2/31          | Spine: .0, Leaf: .1           |
| Spine2_2 – Leaf2_1    | 10.0.2.4/31          | Spine: .0, Leaf: .1           |
| Spine2_2 – Leaf2_2    | 10.0.2.6/31          | Spine: .0, Leaf: .1           |
| Spine2_1 – BorderLeaf-2 | 10.0.2.8/31        | Spine: .0, Border: .1         |
| Spine2_2 – BorderLeaf-2 | 10.0.2.10/31       | Spine: .0, Border: .1         |
#### Клиентские сети (VLAN)
| VLAN | VNI    | Подсеть          | Anycast Gateway | VRF  |
|------|--------|------------------|-----------------|------|
| 10   | 10010  | 192.168.1.0/24   | 192.168.1.1     | VRF1 |
| 20   | 10020  | 192.168.2.0/24   | 192.168.2.1     | VRF1 |
#### L3 VNI
| VRF  | L3 VNI   |
|------|----------|
| VRF1 | 111111   |

## 2. Схема сети <a id="shema"></a>
### <img width="865" height="643" alt="image" src="https://github.com/user-attachments/assets/7b016234-fc1d-4b15-91da-3c30e38368ce" />

### <img width="1762" height="669" alt="image" src="https://github.com/user-attachments/assets/ed9b3a92-607f-40c6-afeb-afee7c3142c2" />

## 3. Настройка Underlay EBGP <a id="underlay"></a>

## Нумерация AS (Автономных систем)

#### ЦОД-1

| Устройство          | AS Number | Роль                                                   |
|---------------------|-----------|--------------------------------------------------------|
| Spine1_1, Spine1_2  | 65000     | Underlay eBGP (общий AS)                               |
| Leaf1_1             | 65001     | Underlay eBGP, Overlay iBGP                            |
| Leaf1_2             | 65002     | Underlay eBGP, Overlay iBGP                            |
| BorderLeaf-1        | 65003     | Underlay eBGP, Overlay iBGP, DCI eBGP                  |

#### ЦОД-2

| Устройство          | AS Number | Роль                                                   |
|---------------------|-----------|--------------------------------------------------------|
| Spine2_1, Spine2_2  | 65000     | Underlay eBGP (общий AS)                               |
| Leaf2_1             | 65021     | Underlay eBGP, Overlay iBGP                            |
| Leaf2_2             | 65022     | Underlay eBGP, Overlay iBGP                            |
| BorderLeaf-2        | 65023     | Underlay eBGP, Overlay iBGP, DCI eBGP                  |

#### DCI

* eBGP между BorderLeaf-1 (AS 65003) и BorderLeaf-2 (AS 65023) через подсеть `10.100.1.0/30`.

- **Цель:** обеспечить IP-связность между всеми Loopback-адресами устройств, используемыми в качестве VTEP и Router‑ID.
- **Архитектура:**
  - Spine объединены в общую автономную систему (AS 65000), все Leaf и BorderLeaf имеют собственные уникальные AS.
  - На физических P2P-линках между Spine и Leaf поднимаются eBGP‑сессии.
  - Каждый коммутатор анонсирует свой Loopback‑адрес в BGP, формируя единую транспортную сеть.

## 4. Настройка Overlay VXLAN EVPN <a id="overlay"></a>

**Цель:** Построить EVPN control‑plane для VXLAN, обеспечивающую обмен информацией о MAC‑адресах, VNI и IP‑префиксах между Leaf и BorderLeaf.

**Архитектура:**
- Overlay строится на eBGP‑сессиях между Loopback‑адресами (Spine и Leaf) с использованием `ebgp-multihop 2`.
- Для передачи extended communities (Route Target, Router MAC) обязательно включается `send-community extended`.
- Spine передают EVPN‑маршруты между Leaf, сохраняя оригинальный next‑hop (`next-hop-unchanged`).
- На Leaf определены:
  - VLAN 10 → VNI 10010 (L2 VNI)
  - VRF VRF1 → L3 VNI 111111
- EVPN обменивается маршрутами Type‑2 (MAC/IP), Type‑3 (IMET), Type‑5 (IP Prefix).

Проверка:
- Убедиться, что BGP EVPN‑сессии с Spine установлены:

bash
show bgp evpn summary
Проверить наличие Type‑2 маршрутов (MAC/IP клиентов):

bash
show bgp evpn route-type mac-ip
Проверить наличие Type‑3 маршрутов (IMET):

bash
show bgp evpn route-type imet
Проверить, что VXLAN интерфейс поднят и remote VTEP видны:

bash
show vxlan vtep
Убедиться, что клиенты в одном VLAN видят друг друга (ping).

Ожидаемый результат: BGP EVPN‑сессии Established, в таблицах EVPN присутствуют Type‑2, Type‑3 маршруты, VXLAN туннели построены, клиенты в одном VLAN связываются.

### Настройка оборудования ЦОД1 <a id="cod1"></a>

<details>
<summary>Конфигурация Spine1_1</summary>

```bash
Spine1_1#sh run
! Command: show running-config
! device: Spine1-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine1_1
!
spanning-tree mode mstp
no spanning-tree vlan-id 4094
!
interface Ethernet1
   description to-LEAF1_1
   mtu 9000
   no switchport
   ip address 10.0.1.0/31
   spanning-tree link-type point-to-point
!
interface Ethernet2
   description to-LEAF1_2
   mtu 9000
   no switchport
   ip address 10.0.1.2/31
   spanning-tree link-type point-to-point
!
interface Ethernet3
   description to-BORDERLEAF-1
   mtu 9000
   no switchport
   ip address 10.0.1.8/31
   spanning-tree link-type point-to-point
!
interface Ethernet4
   no switchport
!
interface Ethernet5
   no switchport
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 10.10.1.1/32
!
interface Management1
!
ip routing
!
router bgp 65000
   router-id 10.10.1.1
   maximum-paths 4 ecmp 4
   neighbor OVER peer group
   neighbor OVER next-hop-unchanged
   neighbor OVER update-source Loopback0
   neighbor OVER ebgp-multihop 4
   neighbor OVER send-community extended
   neighbor UNDER peer group
   neighbor UNDER send-community extended
   neighbor 10.0.1.1 peer group UNDER
   neighbor 10.0.1.1 remote-as 65001
   neighbor 10.0.1.3 peer group UNDER
   neighbor 10.0.1.3 remote-as 65002
   neighbor 10.0.1.9 peer group UNDER
   neighbor 10.0.1.9 remote-as 65003
   neighbor 10.1.1.1 peer group OVER
   neighbor 10.1.1.1 remote-as 65001
   neighbor 10.1.2.1 peer group OVER
   neighbor 10.1.2.1 remote-as 65002
   neighbor 10.1.10.1 peer group OVER
   neighbor 10.1.10.1 remote-as 65003
   redistribute connected
   !
   address-family evpn
      neighbor OVER activate
   !
   address-family ipv4
      neighbor UNDER activate
!
end

```
</details>


<details>
<summary>Конфигурация Spine1_2</summary>

```bash
Spine1_2#sh run
! Command: show running-config
! device: Spine1-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine1_2
!
spanning-tree mode mstp
no spanning-tree vlan-id 4094
!
interface Ethernet1
   description to-LEAF1_1
   mtu 9000
   no switchport
   ip address 10.0.1.4/31
   spanning-tree link-type point-to-point
!
interface Ethernet2
   description to-LEAF1_2
   mtu 9000
   no switchport
   ip address 10.0.1.6/31
   spanning-tree link-type point-to-point
!
interface Ethernet3
   description to-BORDERLEAF-1
   mtu 9000
   no switchport
   ip address 10.0.1.10/31
   spanning-tree link-type point-to-point
!
interface Ethernet4
   no switchport
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
   ip address 10.10.2.1/32
!
interface Management1
!
interface Vlan4094
   description MLAG_PEER_VLAN
   ip address 172.20.2.2/30
!
ip routing
!
router bgp 65000
   router-id 10.10.2.1
   maximum-paths 4 ecmp 4
   neighbor OVER peer group
   neighbor OVER next-hop-unchanged
   neighbor OVER update-source Loopback0
   neighbor OVER ebgp-multihop 4
   neighbor OVER send-community extended
   neighbor UNDER peer group
   neighbor UNDER send-community extended
   neighbor 10.0.1.5 peer group UNDER
   neighbor 10.0.1.5 remote-as 65001
   neighbor 10.0.1.7 peer group UNDER
   neighbor 10.0.1.7 remote-as 65002
   neighbor 10.0.1.11 peer group UNDER
   neighbor 10.0.1.11 remote-as 65003
   neighbor 10.1.1.1 peer group OVER
   neighbor 10.1.1.1 remote-as 65001
   neighbor 10.1.2.1 peer group OVER
   neighbor 10.1.2.1 remote-as 65002
   neighbor 10.1.10.1 peer group OVER
   neighbor 10.1.10.1 remote-as 65003
   redistribute connected
   !
   address-family evpn
      neighbor OVER activate
   !
   address-family ipv4
      neighbor UNDER activate
!
end

```
</details>

<details>
<summary>Конфигурация Leaf1_1</summary>

```bash
#sh run
 ! Command: show running-config
 ! device: localhost (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf1_1
!
spanning-tree mode mstp
no spanning-tree vlan-id 1-4094
!
vlan 10
   name TENANT-A
!
vrf instance VRF1
!
interface Port-Channel10
   description ESI-LAG-to-CLIENT
   switchport trunk allowed vlan 10
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0001:0012:000a
      route-target import 00:00:00:01:12:0a
   lacp system-id 0201.0012.000a
!
interface Ethernet1
   description to-SPINE1_1
   mtu 9000
   no switchport
   ip address 10.0.1.1/31
   spanning-tree link-type point-to-point
!
interface Ethernet2
   description to-SPINE1_2
   mtu 9000
   no switchport
   ip address 10.0.1.5/31
   spanning-tree link-type point-to-point
!
interface Ethernet3
   no switchport
!
interface Ethernet4
!
interface Ethernet5
   description to-CLIENT
   switchport trunk allowed vlan 10
   switchport mode trunk
   channel-group 10 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description client1
   switchport access vlan 10
!
interface Loopback0
   ip address 10.1.1.1/32
!
interface Management1
!
interface Vlan10
   description TEST-VXLAN
   vrf VRF1
   ip address 192.168.1.2/24
   ip virtual-router address 192.168.1.1
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf VRF1 vni 111111
   vxlan learn-restrict any
!
ip virtual-router mac-address 02:00:00:00:00:00
!
ip routing
ip routing vrf VRF1
!
router bgp 65001
   router-id 10.1.1.1
   maximum-paths 2 ecmp 2
   neighbor OVER peer group
   neighbor OVER remote-as 65000
   neighbor OVER update-source Loopback0
   neighbor OVER ebgp-multihop 3
   neighbor OVER send-community standard extended
   neighbor UNDER peer group
   neighbor UNDER remote-as 65000
   neighbor 10.0.1.0 peer group UNDER
   neighbor 10.0.1.0 description SPINE_1_1_p2p
   neighbor 10.0.1.4 peer group UNDER
   neighbor 10.0.1.4 description SPINE_1_2_p2p
   neighbor 10.10.1.1 peer group OVER
   neighbor 10.10.1.1 description SPINE_1_1_lo
   neighbor 10.10.2.1 peer group OVER
   neighbor 10.10.2.1 description SPINE_1_2_lo
   redistribute connected
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   address-family evpn
      neighbor OVER activate
      no neighbor UNDER activate
   !
   address-family ipv4
      neighbor UNDER activate
   !
   vrf VRF1
      rd 65001:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
      !
      address-family ipv4
         redistribute connected
!
end

```
</details>

<details>
<summary>Конфигурация Leaf1_2</summary>

```bash Leaf1_2#sh run
! Command: show running-config
! device: Leaf1-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf1_2
!
spanning-tree mode mstp
no spanning-tree vlan-id 1-4094
!
vlan 10
   name TENANT-A
!
vrf instance VRF1
!
interface Port-Channel10
   description ESI-LAG-to-CLIENT
   switchport trunk allowed vlan 10
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0001:0012:000a
      route-target import 00:00:00:01:12:0a
   lacp system-id 0201.0012.000a
!
interface Ethernet1
   description to-SPINE1_1
   mtu 9000
   no switchport
   ip address 10.0.1.3/31
   spanning-tree link-type point-to-point
!
interface Ethernet2
   description to-SPINE1_2
   mtu 9000
   no switchport
   ip address 10.0.1.7/31
   spanning-tree link-type point-to-point
!
interface Ethernet3
   no switchport
!
interface Ethernet4
!
interface Ethernet5
   description to-CLIENT
   switchport trunk allowed vlan 10
   switchport mode trunk
   channel-group 10 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   switchport access vlan 10
!
interface Loopback0
   ip address 10.1.2.1/32
!
interface Management1
!
interface Vlan10
   description TEST-VXLAN
   vrf VRF1
   ip address 192.168.1.3/24
   ip virtual-router address 192.168.1.1
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf VRF1 vni 111111
   vxlan learn-restrict any
!
ip virtual-router mac-address 02:00:00:00:00:00
ip virtual-router address subnet-routes
!
ip routing
ip routing vrf VRF1
!
router bgp 65002
   router-id 10.1.2.1
   maximum-paths 2 ecmp 2
   neighbor OVER peer group
   neighbor OVER remote-as 65000
   neighbor OVER update-source Loopback0
   neighbor OVER ebgp-multihop 3
   neighbor OVER send-community standard extended
   neighbor UNDER peer group
   neighbor UNDER remote-as 65000
   neighbor 10.0.1.2 peer group UNDER
   neighbor 10.0.1.2 description SPINE_1_1_p2p
   neighbor 10.0.1.6 peer group UNDER
   neighbor 10.0.1.6 description SPINE_1_2_p2p
   neighbor 10.10.1.1 peer group OVER
   neighbor 10.10.1.1 description SPINE_1_1_lo
   neighbor 10.10.2.1 peer group OVER
   neighbor 10.10.2.1 description SPINE_1_2_lo
   redistribute connected
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   address-family evpn
      neighbor OVER activate
   !
   address-family ipv4
      neighbor UNDER activate
   !
   vrf VRF1
      rd 65002:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
      !
      address-family ipv4
         redistribute connected
!
end
```
</details>


### Настройка оборудования ЦОД2 <a id="cod2"></a>

<details>
<summary>Конфигурация Spine2_1</summary>

```bash
Spine2_1#sh run
! Command: show running-config
! device: Spine2-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine2_1
!
spanning-tree mode mstp
!
interface Ethernet1
   description to-LEAF2_1
   no switchport
   ip address 10.0.2.0/31
!
interface Ethernet2
   description to-LEAF2_2
   no switchport
   ip address 10.0.2.2/31
!
interface Ethernet3
   description to-BORDERLEAF-2
   no switchport
   ip address 10.0.2.8/31
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
   ip address 10.20.1.1/32
!
interface Management1
!
ip routing
!
router bgp 65000
   router-id 10.20.1.1
   maximum-paths 4 ecmp 4
   neighbor 10.0.2.1 remote-as 65021
   neighbor 10.0.2.3 remote-as 65022
   neighbor 10.0.2.9 remote-as 65023
   neighbor 10.2.1.1 remote-as 65021
   neighbor 10.2.1.1 update-source Loopback0
   neighbor 10.2.1.1 ebgp-multihop 2
   neighbor 10.2.1.1 route-reflector-client
   neighbor 10.2.1.1 send-community extended
   neighbor 10.2.2.1 remote-as 65022
   neighbor 10.2.2.1 update-source Loopback0
   neighbor 10.2.2.1 ebgp-multihop 2
   neighbor 10.2.2.1 route-reflector-client
   neighbor 10.2.2.1 send-community extended
   neighbor 10.2.10.1 remote-as 65023
   neighbor 10.2.10.1 update-source Loopback0
   neighbor 10.2.10.1 ebgp-multihop 2
   neighbor 10.2.10.1 route-reflector-client
   neighbor 10.2.10.1 send-community extended
   !
   address-family evpn
      neighbor 10.2.1.1 activate
      neighbor 10.2.1.1 next-hop-unchanged
      neighbor 10.2.2.1 activate
      neighbor 10.2.2.1 next-hop-unchanged
      neighbor 10.2.10.1 activate
      neighbor 10.2.10.1 next-hop-unchanged
   !
   address-family ipv4
      neighbor 10.0.2.1 activate
      neighbor 10.0.2.3 activate
      neighbor 10.0.2.9 activate
      network 10.0.2.0/31
      network 10.0.2.2/31
      network 10.0.2.8/31
      network 10.20.1.1/32
!
end


```
</details>


<details>
<summary>Конфигурация Spine2_2</summary>

```bash
Spine2_2#sh run
! Command: show running-config
! device: Spine2-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine2_2
!
spanning-tree mode mstp
!
interface Ethernet1
   description to-LEAF2_1
   no switchport
   ip address 10.0.2.4/31
!
interface Ethernet2
   description to-LEAF2_2
   no switchport
   ip address 10.0.2.6/31
!
interface Ethernet3
   description to-BORDERLEAF-2
   no switchport
   ip address 10.0.2.10/31
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
   ip address 10.20.2.1/32
!
interface Management1
!
ip routing
!
router bgp 65000
   router-id 10.20.2.1
   maximum-paths 4 ecmp 4
   neighbor 10.0.2.5 remote-as 65021
   neighbor 10.0.2.7 remote-as 65022
   neighbor 10.0.2.11 remote-as 65023
   neighbor 10.2.1.1 remote-as 65021
   neighbor 10.2.1.1 update-source Loopback0
   neighbor 10.2.1.1 ebgp-multihop 2
   neighbor 10.2.1.1 route-reflector-client
   neighbor 10.2.1.1 send-community extended
   neighbor 10.2.2.1 remote-as 65022
   neighbor 10.2.2.1 update-source Loopback0
   neighbor 10.2.2.1 ebgp-multihop 2
   neighbor 10.2.2.1 route-reflector-client
   neighbor 10.2.2.1 send-community extended
   neighbor 10.2.10.1 remote-as 65023
   neighbor 10.2.10.1 update-source Loopback0
   neighbor 10.2.10.1 ebgp-multihop 2
   neighbor 10.2.10.1 route-reflector-client
   neighbor 10.2.10.1 send-community extended
   !
   address-family evpn
      neighbor 10.2.1.1 activate
      neighbor 10.2.1.1 next-hop-unchanged
      neighbor 10.2.2.1 activate
      neighbor 10.2.2.1 next-hop-unchanged
      neighbor 10.2.10.1 activate
      neighbor 10.2.10.1 next-hop-unchanged
   !
   address-family ipv4
      neighbor 10.0.2.5 activate
      neighbor 10.0.2.7 activate
      neighbor 10.0.2.11 activate
      network 10.0.2.4/31
      network 10.0.2.6/31
      network 10.0.2.10/31
      network 10.20.2.1/32
!
end

```
</details>

<details>
<summary>Конфигурация Leaf2_1</summary>

```bash
leaf2_1#sh run
! Command: show running-config
! device: leaf2-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf2_1
!
spanning-tree mode mstp
no spanning-tree vlan-id 1-4094
!
vlan 20
   name TENANT-B
!
vrf instance VRF1
!
interface Port-Channel10
   lacp system-id 0201.0012.000a
!
interface Ethernet1
   description to-SPINE2_1
   no switchport
   ip address 10.0.2.1/31
!
interface Ethernet2
   description to-SPINE2_2
   no switchport
   ip address 10.0.2.5/31
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
   switchport trunk allowed vlan 10
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   description to-CLIENT
   switchport access vlan 20
!
interface Loopback0
   ip address 10.2.1.1/32
!
interface Management1
!
interface Vlan20
   vrf VRF1
   ip address 192.168.2.2/24
   ip virtual-router address 192.168.2.1
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vrf VRF1 vni 111111
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:11:22:33:44:55
ip virtual-router address subnet-routes
!
ip routing
ip routing vrf VRF1
!
router bgp 65021
   router-id 10.2.1.1
   maximum-paths 2 ecmp 2
   neighbor 10.0.2.0 remote-as 65000
   neighbor 10.0.2.4 remote-as 65000
   neighbor 10.20.1.1 remote-as 65000
   neighbor 10.20.1.1 update-source Loopback0
   neighbor 10.20.1.1 ebgp-multihop 2
   neighbor 10.20.1.1 send-community extended
   neighbor 10.20.2.1 remote-as 65000
   neighbor 10.20.2.1 update-source Loopback0
   neighbor 10.20.2.1 ebgp-multihop 2
   neighbor 10.20.2.1 send-community extended
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor 10.20.1.1 activate
      neighbor 10.20.1.1 next-hop-unchanged
      neighbor 10.20.2.1 activate
      neighbor 10.20.2.1 next-hop-unchanged
   !
   address-family ipv4
      neighbor 10.0.2.0 activate
      neighbor 10.0.2.4 activate
      network 10.0.2.0/31
      network 10.0.2.4/31
      network 10.2.1.1/32
   !
   vrf VRF1
      rd 65021:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
      !
      address-family ipv4
         redistribute connected
!
end

```
</details>


<details>
<summary>Конфигурация Leaf2_2</summary>

```bash
Leaf2_2#sh run
! Command: show running-config
! device: Leaf2-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf2_2
!
spanning-tree mode mstp
no spanning-tree vlan-id 1-4094
!
vlan 20
   name TENANT-B
!
vrf instance VRF1
!
interface Ethernet1
   description to-SPINE2_1
   no switchport
   ip address 10.0.2.3/31
!
interface Ethernet2
   description to-SPINE2_2
   no switchport
   ip address 10.0.2.7/31
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
!
interface Ethernet8
   description to-CLIENT
   switchport access vlan 20
!
interface Loopback0
   ip address 10.2.2.1/32
!
interface Management1
!
interface Vlan20
   vrf VRF1
   ip address 192.168.2.3/24
   ip virtual-router address 192.168.2.1
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vrf VRF1 vni 111111
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:11:22:33:44:55
ip virtual-router address subnet-routes
!
ip routing
ip routing vrf VRF1
!
router bgp 65022
   router-id 10.2.2.1
   maximum-paths 2 ecmp 2
   neighbor 10.0.2.2 remote-as 65000
   neighbor 10.0.2.6 remote-as 65000
   neighbor 10.20.1.1 remote-as 65000
   neighbor 10.20.1.1 update-source Loopback0
   neighbor 10.20.1.1 ebgp-multihop 2
   neighbor 10.20.1.1 send-community extended
   neighbor 10.20.2.1 remote-as 65000
   neighbor 10.20.2.1 update-source Loopback0
   neighbor 10.20.2.1 ebgp-multihop 2
   neighbor 10.20.2.1 send-community extended
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor 10.20.1.1 activate
      neighbor 10.20.1.1 next-hop-unchanged
      neighbor 10.20.2.1 activate
      neighbor 10.20.2.1 next-hop-unchanged
   !
   address-family ipv4
      neighbor 10.0.2.2 activate
      neighbor 10.0.2.6 activate
      network 10.0.2.2/31
      network 10.0.2.6/31
      network 10.2.2.1/32
   !
   vrf VRF1
      rd 65022:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
      !
      address-family ipv4
         redistribute connected
!
end

```
</details>

### Настройка Border Leaf для связности между l2 сегментами ЦОДов <a id="border"></a>

1. настраиваем ebgp сессию между p2p линками. У меня линки собраны в Port-channel и интерфейсу дан адрес
### <img width="267" height="86" alt="изображение" src="https://github.com/user-attachments/assets/96b5a3c5-5534-4ef1-ad75-d102354baac3" />
2. настраиваем evpn сессию между loopback 2х бордеров 
- в общей таблице bgp создаем соседей и активируем их в  address-family evpn
### <img width="484" height="756" alt="изображение" src="https://github.com/user-attachments/assets/b33d3082-f918-4753-a35d-da84e9ed6c71" />
3. Создаем общий vni для передачи префиксов из соседнего POD
   
- объявить на обоих бордерах vlan из соседнего POD

- создать vrf instance VRF1, включить маршрутизацию ip routing vrf VRF1, опустить VRF в нужный vlan (внимательно, т.к. собъются прежние настройки интерфейса), добавить VRF в BGP
   
### <img width="205" height="40" alt="изображение" src="https://github.com/user-attachments/assets/880b2b1a-5024-4eef-806e-07d072b01d45" />
### <img width="318" height="79" alt="изображение" src="https://github.com/user-attachments/assets/1381a73d-3e6b-42d9-afc2-7a2ae789687c" />
### <img width="301" height="154" alt="изображение" src="https://github.com/user-attachments/assets/c5d8a9fe-37b9-40e6-b0ee-d6f3bdec89ec" />
### <img width="617" height="936" alt="изображение" src="https://github.com/user-attachments/assets/a4cb9d9c-e9bf-4627-8429-6014eda479a6" />
*** если ранее на leaf не был добавлен общий vni - необходимо добавить
### Проверка
- проверка связности сегментов
#### первые пинги, ожидаемо, проходят не уверенно
### <img width="552" height="452" alt="изображение" src="https://github.com/user-attachments/assets/c7ab68fc-5371-45cb-9613-9d30406844d4" />
#### но затем маршруты 3 типа появляются в таблицах коммутаторов, а на PC появляется арп запись и все идет стабильно:
### <img width="623" height="236" alt="изображение" src="https://github.com/user-attachments/assets/c74cf671-bb4c-4f10-9d5f-9533219fc72e" />
- посмотреть что видно vtep соседнего POD
### <img width="315" height="180" alt="изображение" src="https://github.com/user-attachments/assets/49e02564-3da3-4678-ad76-4f60b5a535a0" />
- посмотреть маршруты типа 3
### <img width="858" height="351" alt="image" src="https://github.com/user-attachments/assets/7e13166b-c3aa-4801-a851-0e0039ea5798" />
- посмотреть маршруты типа 2
### <img width="849" height="486" alt="изображение" src="https://github.com/user-attachments/assets/a8b66728-8cdf-4cba-a70c-ccbf224cd187" />
- посмотреть маршруты для vrf
### <img width="946" height="326" alt="image" src="https://github.com/user-attachments/assets/18b5608b-74f5-40e2-8cb3-ef6bdb10d4b7" />
- проверить что l3vni есть
### <img width="563" height="208" alt="image" src="https://github.com/user-attachments/assets/5bd5fa65-49a7-4750-9039-db0ddce3c08d" />

## 5. Настройка Multihoming ethernet сегмента в ЦОД 1 <a id="multihoming"></a>

**Цель:** Обеспечить отказоустойчивое подключение клиента двумя линками к разным Leaf (Leaf1_1 и Leaf1_2) с использованием стандартного механизма EVPN ESI LAG.

**Архитектура:**
- На Leaf1_1 и Leaf1_2 создан одинаковый Ethernet Segment Identifier (ESI) `0000:0000:0001:0012:000a` и LACP system‑id `0201.0012.000a` для порт‑канала, ведущего к клиенту.
- Клиент подключается через LACP (mode active) к обоим Leaf.
- EVPN генерирует Type‑4 маршруты (Ethernet Segment), которые используются для обнаружения сегмента и выбора Designated Forwarder (DF).
- Режим all‑active позволяет передавать трафик через оба Leaf одновременно.

**Настройка (на Leaf1_1 и Leaf1_2):**
```bash
interface Port-Channel10
   description ESI-LAG-to-CLIENT
   switchport mode trunk
   switchport trunk allowed vlan 10
   !
   evpn ethernet-segment
      identifier 0000:0000:0001:0012:000a
      route-target import 00:00:00:01:12:0a
   lacp system-id 0201.0012.000a
!
interface Ethernet5
   description to-CLIENT
   switchport mode trunk
   switchport trunk allowed vlan 10
   channel-group 10 mode active

```

### Проверка:

- Убедиться, что порт‑канал поднят и LACP активен:

#### show port-channel dense
#### <img width="705" height="291" alt="image" src="https://github.com/user-attachments/assets/b91a8fab-5bba-40d3-8bc2-359e046d5459" />
#### sh lacp interface ethernet 5
<img width="670" height="246" alt="image" src="https://github.com/user-attachments/assets/2a7f1c25-71be-425f-86f5-e5e546c2bb32" />

Проверить состояние Ethernet Segment:
#### show bgp evpn instance
Должно быть State: up и назначен Designated Forwarder.
#### <img width="404" height="242" alt="image" src="https://github.com/user-attachments/assets/de1cc42d-4c13-4358-889e-ca6ed2f68250" />

- Проверить наличие Type‑4 маршрутов:

#### show bgp evpn route-type ethernet-segment
### <img width="791" height="235" alt="image" src="https://github.com/user-attachments/assets/a269a467-52c3-4144-b305-342407883e8e" />
Проверить, что в Type‑2 маршрутах клиента указан ESI:
#### show bgp evpn route-type mac-ip | include 5000.0088.fe27
<img width="565" height="96" alt="image" src="https://github.com/user-attachments/assets/01cc7e9d-9fba-46c4-a029-8d1663e5e6fe" />

## 6. Настройка инкапсуляции маршрута от EBGP соседа в фабрику <a id="encapsulation"></a>
1. Настроить P2P  интерфейсы на обоих роутерах
2. На обоих роутерах поднять SVI интерфейсы
3. На роутере без EVPN - просто настраиваем eBGP сессию с соседом, lO поднят просто для примера, именно его мы будем инскапсулировать в фабрику
#### <img width="407" height="323" alt="image" src="https://github.com/user-attachments/assets/8f6e0671-cb79-4137-8d45-9abe07b8ba66" />
4. На BorderLeaf-1 поднятый SVI интерфейс необходимо добавить в VRF, объявить в BGP EVPN и доавить в BGP VRF адрес SVI соседа
#### <img width="549" height="976" alt="image" src="https://github.com/user-attachments/assets/3d8ec976-ad33-404c-a3be-fa1281ff68c8" />

## 7. Траблшутинг <a id="troubleshooting"></a>
После всех настроек нет связности между хостами в сетях 192.168.1.0/24 и 192.168.2.0/24
1. посмотреть что устройства попадают в нужный vlan и vxlan
### sh mac-address-table на лифе куда он подключен
### <img width="663" height="270" alt="image" src="https://github.com/user-attachments/assets/c5b3105a-6a0d-4274-955f-97e776ce3d19" />
### проверить что объявленный vlan попадает в нужный vni
### <img width="547" height="169" alt="изображение" src="https://github.com/user-attachments/assets/bcbab6b2-4980-4117-88e7-0ecd55cf319e" />
### посмотреть что vlan попадают в нужный vrf 
### <img width="599" height="97" alt="изображение" src="https://github.com/user-attachments/assets/8263b898-0abd-460d-9d9e-ca887e45f217" />
### убедиться что с устройств точно доступны их GW
2. проверка на всех задействованых узлах сессий eBGP и BGP EVPN (на лифах должны быть установлены сессии IPv4 с Spine-2шт, Border-2шт, EVPN с Border - 2 шт)
<img width="917" height="276" alt="изображение" src="https://github.com/user-attachments/assets/91d3006f-c4e4-4c27-9c66-2259e4526957" />
### Если сессия EVPN не установливается возможные ошибки:
- правильность id на всех узлах
- доступен ли нужный neighbor (если это p2p - проверить связность утилитой пинг, если это loopback - наличие маршрута в GRT)
3. Проверить наличие маршрутов, связность
### проверка связности между GW нужных сегментов
<img width="768" height="239" alt="image" src="https://github.com/user-attachments/assets/7afde69f-7ac3-401e-9dcf-7637fd2a9f2e" />
проверить что маршрут попадает в нужный vrf
### <img width="960" height="330" alt="image" src="https://github.com/user-attachments/assets/6fed2c5b-28e7-4c76-a36f-e1888c3ee555" />
### посмотреть какие видно маршруты
### <img width="1046" height="363" alt="изображение" src="https://github.com/user-attachments/assets/ab7f1c3a-47bf-4086-8a83-355b420073ce" />
### проверка маршрутов типа 2 и 3 на всех задейстованных Leaf
### <img width="801" height="706" alt="image" src="https://github.com/user-attachments/assets/53695eb3-e034-4e24-a83c-6505f39f0a69" />


