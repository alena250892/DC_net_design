# OSPF 
## Схема сети
<img width="1322" height="642" alt="image" src="https://github.com/user-attachments/assets/86ba29ff-6bfe-4502-950a-aae32e64eac0" />
## Backbone AREA 0.0.0.0
### Spine 1
router ospf 1010
   router-id 10.0.1.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   network 10.0.1.1/32 area 0.0.0.0
   network 10.2.1.0/31 area 0.0.0.0
   network 10.2.1.2/31 area 0.0.0.0
   network 10.2.1.4/31 area 0.0.0.0
   max-lsa 12000

SPINE1#sh ip ospf database

            OSPF Router with ID(10.0.1.1) (Instance ID 1010) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.1.1.1        10.1.1.1        1735        0x800008c0   0x97c    5
10.0.2.1        10.0.2.1        1784        0x80000a53   0x5332   7
10.0.1.1        10.0.1.1        1726        0x80000a97   0xf457   7
10.1.1.2        10.1.1.2        1738        0x80000763   0x6076   5
10.1.1.3        10.1.1.3        1769        0x800006f4   0x85b7   5

                 Summary Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum
192.168.4.0     10.1.1.3        1829        0x80000026   0xaed8
192.168.3.0     10.1.1.3        1829        0x80000023   0xbfcb
192.168.2.0     10.1.1.2        1618        0x80000022   0xd2bb
192.168.1.0     10.1.1.1        1135        0x8000000c   0x1096
10.3.1.4        10.1.1.3        1289        0x80000069   0xc1e8
10.3.1.0        10.1.1.1        1135        0x80000016   0x9c67
10.3.1.6        10.1.1.3        1289        0x80000069   0xadfa
10.3.1.2        10.1.1.2        1318        0x800000b2   0x491b

                 Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#         Checksum Tag
8.8.8.8         10.0.2.1        1664        0x80000011   0x570    0
