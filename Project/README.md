## Проектирование сетевой фабрики на основе VxLAN EVPN.

### План работ:
#### 1. [Составление IP плана сети](#-ip-план-фабрики)
#### 2. [Схема сети](#-Схема-сети)
#### 3. Настройка Underlay EBGP
#### 4. Настройка Overlay VXLAN EVPN
##### ⋅⋅* Настройка оборудования ЦОД1
##### ⋅⋅* Настройка оборудования ЦОД2
##### ⋅⋅* Настройка Border Leaf для связности между l2 сегментами ЦОДов
#### 5. Настройка Multihoming ethernet сегмента в ЦОД 1
#### 6. Настройка инкапсуляции маршрута от EBGP соседа в фабрику
#### 7. Траблшутинг 

---
## 1. IP-план фабрики
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
## 2. Схема сети
### <img width="865" height="643" alt="image" src="https://github.com/user-attachments/assets/7b016234-fc1d-4b15-91da-3c30e38368ce" />
#### 3. Настройка Underlay EBGP
##### ⋅⋅* Настройка оборудования ЦОД1

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
```markdown
end
</details>




##### ⋅⋅*Настройка оборудования ЦОД2
##### ⋅⋅*Настройка Border Leaf для связности между l2 сегментами ЦОДов
1. настраиваем ebgp сессию между p2p линками. У меня линки собраны в Port-channel и интерфейсу дан адрес
## <img width="267" height="86" alt="изображение" src="https://github.com/user-attachments/assets/96b5a3c5-5534-4ef1-ad75-d102354baac3" />
2. настраиваем evpn сессию между loopback 2х бордеров 
..*В общей таблице bgp создаем соседей и активируем их в  address-family evpn
### <img width="484" height="756" alt="изображение" src="https://github.com/user-attachments/assets/b33d3082-f918-4753-a35d-da84e9ed6c71" />
3. Создаем общий vni для передачи префиксов из соседнего POD
..*объявить на обоих бордерах vlan из соседнего POD
..*создать vrf instance VRF1, включить маршрутизацию ip routing vrf VRF1, опустить VRF в нужный vlan (внимательно, т.к. собъются прежние настройки интерфейса), добавить VRF в BGP
### <img width="205" height="40" alt="изображение" src="https://github.com/user-attachments/assets/880b2b1a-5024-4eef-806e-07d072b01d45" />
### <img width="318" height="79" alt="изображение" src="https://github.com/user-attachments/assets/1381a73d-3e6b-42d9-afc2-7a2ae789687c" />
### <img width="301" height="154" alt="изображение" src="https://github.com/user-attachments/assets/c5d8a9fe-37b9-40e6-b0ee-d6f3bdec89ec" />
### <img width="617" height="936" alt="изображение" src="https://github.com/user-attachments/assets/a4cb9d9c-e9bf-4627-8429-6014eda479a6" />
*** если ранее на leaf не был добавлен общий vni - необходимо добавить
### CHECKING
..* проверка связности сегментов
#### первые пинги, ожидаемо, проходят не уверенно
### <img width="552" height="452" alt="изображение" src="https://github.com/user-attachments/assets/c7ab68fc-5371-45cb-9613-9d30406844d4" />
#### но затем маршруты 3 типа появляются в таблицах коммутаторов, а на PC появляется арп запись и все идет стабильно:
### <img width="623" height="236" alt="изображение" src="https://github.com/user-attachments/assets/c74cf671-bb4c-4f10-9d5f-9533219fc72e" />
..* посмотреть что видно vtep соседнего POD
### <img width="315" height="180" alt="изображение" src="https://github.com/user-attachments/assets/49e02564-3da3-4678-ad76-4f60b5a535a0" />
..* посмотреть маршруты типа 3
###
..* посмотреть маршруты типа 2
### <img width="849" height="486" alt="изображение" src="https://github.com/user-attachments/assets/a8b66728-8cdf-4cba-a70c-ccbf224cd187" />
..* посмотреть маршруты для vrf
###
..* проверить что l3vni есть


#### 5. Настройка Multihoming ethernet сегмента в ЦОД 1
#### 6. Настройка инкапсуляции маршрута от EBGP соседа в фабрику

---
## 7. Траблшутинг сетевой связности между хостами в сетях 192.168.1.0/24 и 192.168.2.0/24
⋅⋅* проверка на всех задействованых узлах сессий eBGP и BGP EVPN
<img width="917" height="276" alt="изображение" src="https://github.com/user-attachments/assets/91d3006f-c4e4-4c27-9c66-2259e4526957" />

Если сессия EVPN не установливается возможные ошибки:
⋅⋅* правильность id на всех узлах
⋅⋅* не доступен neighbor (если это p2p - проверить связность утилитой пинг, если это loopback - наличие маршрута в GRT)


..* проверить что маршрут попадает в нужный vrf
### посмотреть какие видно маршруты
### <img width="1046" height="363" alt="изображение" src="https://github.com/user-attachments/assets/ab7f1c3a-47bf-4086-8a83-355b420073ce" />
### посмотреть что vlan попадают в нужный vrf 
### <img width="599" height="97" alt="изображение" src="https://github.com/user-attachments/assets/8263b898-0abd-460d-9d9e-ca887e45f217" />

### посмотреть что устройства попадают в нужный vlan и vxlan
sh mac-address-table
### проверить что объявленный vlan попадает в нужный vni
### <img width="547" height="169" alt="изображение" src="https://github.com/user-attachments/assets/bcbab6b2-4980-4117-88e7-0ecd55cf319e" />




