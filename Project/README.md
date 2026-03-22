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
<img width="865" height="643" alt="image" src="https://github.com/user-attachments/assets/7b016234-fc1d-4b15-91da-3c30e38368ce" />
#### 3. Настройка Underlay EBGP
##### ⋅⋅* Настройка оборудования ЦОД1
##### ⋅⋅* Настройка оборудования ЦОД2
##### ⋅⋅* Настройка Border Leaf для связности между l2 сегментами ЦОДов
1. настраиваем ebgp сессию между p2p линками. У меня линки собраны в Port-channel и интерфейсу дан адрес
## <img width="267" height="86" alt="изображение" src="https://github.com/user-attachments/assets/96b5a3c5-5534-4ef1-ad75-d102354baac3" />
2. настраиваем evpn сессию между loopback 2х бордеров 
..* В общей таблице bgp создаем соседей и активируем их в  address-family evpn
### <img width="484" height="756" alt="изображение" src="https://github.com/user-attachments/assets/b33d3082-f918-4753-a35d-da84e9ed6c71" />
3. Создаем общий vni для передачи префиксов из соседнего POD
..* объявить на обоих бордерах vlan из соседнего POD
..* создать vrf instance VRF1, включить маршрутизацию ip routing vrf VRF1, опустить VRF в нужный vlan (внимательно, т.к. собъются прежние настройки интерфейса), добавить VRF в BGP
### <img width="205" height="40" alt="изображение" src="https://github.com/user-attachments/assets/880b2b1a-5024-4eef-806e-07d072b01d45" />
### <img width="318" height="79" alt="изображение" src="https://github.com/user-attachments/assets/1381a73d-3e6b-42d9-afc2-7a2ae789687c" />
### <img width="301" height="154" alt="изображение" src="https://github.com/user-attachments/assets/c5d8a9fe-37b9-40e6-b0ee-d6f3bdec89ec" />
### <img width="617" height="936" alt="изображение" src="https://github.com/user-attachments/assets/a4cb9d9c-e9bf-4627-8429-6014eda479a6" />
*** если ранее на leaf не был добавлен общий vni - необходимо добавить

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



