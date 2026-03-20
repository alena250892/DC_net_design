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

---
## 7. Траблшутинг сетевой связности между хостами в сетях 192.168.1.0/24 и 192.168.2.0/24
⋅⋅* проверка на всех задействованых узлах сессий eBGP и BGP EVPN
<img width="917" height="276" alt="изображение" src="https://github.com/user-attachments/assets/91d3006f-c4e4-4c27-9c66-2259e4526957" />

