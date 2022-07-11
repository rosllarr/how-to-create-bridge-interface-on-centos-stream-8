---
description: อธิบายวิธีการสร้าง Bridge Interface บน Linux ด้วยคำสั่ง nmcli (NetworkManager)
---

# วิธีสร้าง Bridge Interface บน CentOS-Stream-8

### ขั้นตอนการสร้าง Bridge Interface

#### เช็ค Network Interface ทั้งหมดด้วยคำสั่ง

```
ip link
```

![](<.gitbook/assets/image (4).png>)

จะเห็นว่ามี NIC (Network Interface Card) ทั้งหมด 4 ตัว คือ enp1s0, enp7s0, enp8s0 และ enp9s0 โดยมีตัวที่ enable อยู่เพียงตัวเดียวคือ enp1s0 โดยรับ IP จาก DHCP Server ซึ่งเราจะใช้ NIC นี้เป็น Interface ที่แนบเข้ากับ Bridge ที่เราจะกำลังจะสร้างต่อไป

#### ทำการปิด IP บน enp1s0 เพราะเดี๋ยวจะไปกำหนดบน Bridge แทนหลังจากสร้างเสร็จ

* เช็ค NetworkManager Connection ด้วยคำสั่ง

```
ืnmcli con
```

![](<.gitbook/assets/image (5).png>)

จะเห็นว่ามี 4 connection โดยดูจากคอลัม NAME ซึ่งจะมีเพียง Connection เดียวที่แมพอยู่กับ Physical Interface ก็คือ enp1s0&#x20;

วิธีแมพดูจาก NAME{enp1s0} -> DEVICE{enp1s0} ซึ่ง NAME ก็คือชื่อ Connection ที่เราจะใช้อ้างอิง (สามารถตั้งชื่ออะไรก็ได้ในที่นี้กำหนดเหมือนกับ Physical Interface) ในการเซ็ต IP, Gateway, DNS หรืออื่นๆ และ DEVICE ก็คือชื่อ Physical Interface ที่ Connection นี้แมพอยู่ ซึ่งก็คือชื่อที่เราได้จากคำสั่ง `ip link` ที่รันไปก่อนหน้านี้

* สั่ง disable IPv4 (กรณีรับ IP จาก DHCP) และ down connection ด้วยคำสั่ง

```
nmcli con mod enp1s0 ipv4.method disable
nmcli con down enp1s0
```

* สั่ง disable IPv4 (กรณี Static IP) และ down connection ด้วยคำสั่ง

```
nmcli con mod enp1s0 remove ipv4
nmcli con mod enp1s0 ipv4.method disable
nmcli con down enp1s0
```

#### สร้าง Bridge Interface และแนบ Physical Interface

* สร้าง bridge interface ด้วยคำสั่ง

```
nmcli con add type bridge ifname br0 con-name br0 stp off autoconnect yes
```

![](<.gitbook/assets/image (3).png>)

ifname ก็คือชื่อของ DEVICE สามารถกำหนดอะไรก็ได้&#x20;

con-name ก็คือชื่อ Connection สามารถกำหนดอะไรก็ได้

stp off ก็คือจะไม่ใช้ feature: Spaning Tree&#x20;

หลังจากรันคำสั่งแล้วจะเห็นว่ามี br0 เกิดขึ้นมาและ active อยู่

* แนบ Physical Interface เข้ากับ Bridge ด้วยคำสั่ง

```
nmcli con add type bridge-slave \ 
    con-name enp1s0 \
    ifname enp1s0 \
    master br0 \
    autoconnect yes
```

![](<.gitbook/assets/image (1).png>)

![](<.gitbook/assets/image (6).png>)

หลังจากแนบ Physical interface สำเร็จจะเห็นว่ามี 2 connection ที่ active คือ br0 และ enp1s0

* เซ็ต Static IP ให้กับ br0 ด้วยคำสั่ง&#x20;

```
nmcli con mod br0 ipv4.method manual \
    ipv4.addresses 192.168.122.2/24 \
    ipv4.gateway 192.168.122.1 \
    ipv4.dns 192.168.122.1
```

![](<.gitbook/assets/image (2).png>)

ลองเช็ค NetworkConnection อีกครั้งจะเห็นว่ามี 2 connection ที่ active

ลองเช็ค IP ด้วยคำสั่ง `ip a` ก็จะเห็น Interface br0 เพิ่มมาพร้อมกับ IP ที่เราเซ็ตไว้

ลอง Ping ไปหา gateway และ google.com ได้สำเร็จ

### อ้างอิง

* [https://lukas.zapletalovi.com/2015/09/fedora-22-libvirt-with-bridge.html](https://lukas.zapletalovi.com/2015/09/fedora-22-libvirt-with-bridge.html)
* [https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking)
* [https://wiki.linuxfoundation.org/networking/bridge](https://wiki.linuxfoundation.org/networking/bridge)
