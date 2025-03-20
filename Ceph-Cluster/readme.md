# Cài đặt Ceph cluster

### Prerequiste 

- 3 node for ceph-OSD and 3 node cho ceph-MON

```golang
ceph-mon-1 192.168.10.112   
ceph-mon-2 192.168.10.113
ceph-mon-3 192.168.10.114

ceph-osd-1 192.168.10.115
ceph-osd-2 192.168.10.116
ceph-osd-3 192.168.10.117
```

> Có thể tạo 1 node riêng làm admin nhưng ở đây lấy luôn node ceph-mon-1 làm admin. Đảm bảo rằng node admin có thể ssh qua các node còn lại = user root ở port 22 

- Các node ceph-osd phải có 1 ổ cứng raw

```css
root@datnd-ceph-osd-3:~# lsblk -l
NAME                                             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                                              7:0    0 63.7M  1 loop /snap/core20/2434
loop1                                              7:1    0 63.7M  1 loop /snap/core20/2496
loop2                                              7:2    0   87M  1 loop /snap/lxd/27037
loop3                                              7:3    0 89.4M  1 loop /snap/lxd/31333
loop4                                              7:4    0 44.4M  1 loop /snap/snapd/23545
loop5                                              7:5    0 44.4M  1 loop /snap/snapd/23771
sda                                                8:0    0   20G  0 disk 
sda1                                               8:1    0    1M  0 part 
sda2                                               8:2    0   20G  0 part /
sdb                                                8:16   0   20G  0 disk 
sr0                                               11:0    1    2G  0 rom  
ceph--b688eee3--4a73--41c3--8216--e4a35a683f22-osd--block--03f20a06--049a--4b6a--9ee3--8869b749ec12
                                                 253:0    0   20G  0 lvm  
```

### Cài đặt ceph

2. Follow các bước sau:


- Cài đặt python3 và docker trên tất cả các node

- Cài đặt ceph admin
```bash   
apt install -y cephadm

```
- Khởi tạo ceph admin

```bash

cephadm bootstrap --mon-ip 192.168.10.112   


```
- Cài đặt ceph-cli

```bash 

cephadm add-repo --release reef
cephadm install ceph-common

```

- Add các hosts còn lại vào cluster

```bash 
ceph orch host add datnd-ceph-mon-2 192.168.10.113
ceph orch host add datnd-ceph-mon-3 192.168.10.114
ceph orch host add datnd-ceph-osd-1 192.168.10.115
ceph orch host add datnd-ceph-osd-2 192.168.10.116
ceph orch host add datnd-ceph-osd-3 192.168.10.117
```


> Trường hợp add hosts mà báo lỗi sau:

![alt text](image-1.png)

**Chạy lệnh sau:**

- Copy ssh key gen bởi ceph admin vào các node còn lại để có thể join vào cluster

```bash
sudo ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.10.113 

...

sudo ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.10.117
```



