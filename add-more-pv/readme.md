## Cách add thêm ổ cứng vật lý vào server 
```bash 
lsblk -l # xem danh sách các ổ hiện tại (nếu system đã thêm thì nó sẽ hiẻn thị)
________________________________________________________________

    NAME  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    loop0   7:0    0 63.7M  1 loop /snap/core20/2434
    loop1   7:1    0 63.7M  1 loop /snap/core20/2496
    loop2   7:2    0   87M  1 loop /snap/lxd/27037
    loop3   7:3    0 89.4M  1 loop /snap/lxd/31333
    loop4   7:4    0 44.4M  1 loop /snap/snapd/23545
    loop5   7:5    0 44.4M  1 loop /snap/snapd/23771
    sda     8:0    0   20G  0 disk 
    sda1    8:1    0    1M  0 part 
    sda2    8:2    0   20G  0 part /
    sdb     8:16   0   20G  0 disk  # đây chính là ổ cứng được thêm vào 
    sr0    11:0    1    2G  0 rom  


fdisk /dev/sdb
________________________________________________________________


    Welcome to fdisk (util-linux 2.37.2).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.

    Device does not contain a recognized partition table.
    Created a new DOS disklabel with disk identifier 0xb52aa151.

    Command (m for help): n     # chọn n để tạo phân vùng mới 
    Partition type
    p   primary (0 primary, 0 extended, 4 free)
    e   extended (container for logical partitions)
    Select (default p): e       # chọn e (extened) để tạo phân vùng mở rộng 
    Partition number (1-4, default 1):    # nhấn enter để chọn số phân vùng mặc dịnh (1)
    First sector (2048-41943039, default 2048): # nhập sector bắt đầu của phân vùng. Mặc định là 2048.
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943039, default 41943039): # nhập sector cuối của phân vùng hoặc kích thước của phân vùng. Mặc định là 41943039, tức là sử dụng toàn bộ không gian trống trên ổ đĩa. > nhấn enter là xong

    Created a new partition 1 of type 'Extended' and of size 20 GiB.

    Command (m for help): w # chọn w để ghi phân vùng mới vào ổ đĩa
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.

lsblk -l 
________________________________________________________________

    NAME  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    loop0   7:0    0 63.7M  1 loop /snap/core20/2434
    loop1   7:1    0 63.7M  1 loop /snap/core20/2496
    loop2   7:2    0   87M  1 loop /snap/lxd/27037
    loop3   7:3    0 89.4M  1 loop /snap/lxd/31333
    loop4   7:4    0 44.4M  1 loop /snap/snapd/23545
    loop5   7:5    0 44.4M  1 loop /snap/snapd/23771
    sda     8:0    0   20G  0 disk 
    sda1    8:1    0    1M  0 part 
    sda2    8:2    0   20G  0 part /
    sdb     8:16   0   20G  0 disk 
    sdb1    8:17   0    1K  0 part 
    sdb5    8:21   0   20G  0 part # đây chính là phân vùng vừa tạo thành công 
    sr0    11:0    1    2G  0 rom  


vgcreate ceph_data_vg /dev/sdb5 #tạo Volume Group (VG) để nhóm các physical volume lại với nhau. Đặt tên VG cho dễ quản lý, ví dụ ceph_data_vg.

lvcreate -l 100%FREE -n ceph_osd_data ceph_data_vg # Tạo Logical Volume (LV) trên Volume Group vừa tạo. Bạn có thể tạo một LV cho Ceph OSD. Giả sử tên LV là ceph_osd_data.

mkfs.ext4 /dev/ceph_data_vg/ceph_osd_data # format định dạng file


sudo mkdir -p /ceph-storage  # Tạo thư mục mount
sudo mount /dev/ceph_data_vg/ceph_osd_data /ceph-storage

sudo vi /etc/fstab

/dev/ceph_data_vg/ceph_osd_data  /ceph-storage ext4  defaults  0  2


```
