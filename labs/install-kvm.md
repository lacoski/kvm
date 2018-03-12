# Cài đặt KVM Hypervisor trên CentOS 7
---
## Tổng quan KVM
(xem thêm docs)
## Chuẩn bị
Cài đặt trên 1 node CentOS 7

Tài nguyên
```
CPU 4 Core
RAM 4 GB
HDD 50GB
```
## Cài đặt
### Chuẩn bị
Trước khi cài kiểm tra CPU có hỗ trợ ảo hóa
```
grep -E '(vmx|svm)' /proc/cpuinfo
```
> Nếu hiện ra 1 vài dòng vmx hoặc svm tại đầu ra thì CPU hỗ trợ và ngược lại

### Phần 1: Cài đặt KVM và các package hỗ trợ
#### Bước 1: Cài đặt gói và các hỗ trợ
```
yum install qemu-kvm qemu-img virt-manager libvirt libvirt-python libvirt-client virt-install virt-viewer bridge-utils -y
```
#### Bước 2: Khởi động dịch vụ libvirtd service
```
systemctl start libvirtd
systemctl enable libvirtd
```
#### Kiểm tra service KVM
```
lsmod | grep kvm

# KQ
kvm_intel             162153  0
kvm                   525409  1 kvm_intel
```
#### Bước 3: Nếu chạy CentOS 7/RHEL 7 minimal
Nếu chạy server với phiên bản trên, thiết lập x-window để có thể truy cập từ xa

```
yum install "@X Window System" xorg-x11-xauth xorg-x11-fonts-* xorg-x11-utils -y
```

### Phần 2: Thiết lập Virt Manager
#### Bước 1: Khởi động Virt Manager
```
virt-manager
```
__Giao diện__
![](../images/install-kvm-1.png)
### Phần 3: Cấu hình Bridge KVM
#### Bước 1: Cấu hình card mạng KVM node
```
cd /etc/sysconfig/network-scripts
vi ifcfg-ens33
```
Nội dung
```
TYPE="Ethernet"
BOOTPROTO=static    # Cần phải gán IP tính
IPADDR=192.168.2.133
GATEWAY=192.168.2.2
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="908a3370-6a4b-4f4f-8bd1-3f03bae90528"
DEVICE="ens33"
ONBOOT="yes"
BRIGDE=br0      # Lưu ý brigde sang địa chỉ này

```
#### Bước 2: Cấu hình card mạng brigde
```
cd /etc/sysconfig/network-scripts
vi ifcfg-br0
```
Nội dung
```
TYPE=Bridge
BOOTPROTO=static
NAME="br0"
DEVICE="br0"
ONBOOT="yes"
IPADDR=192.168.2.150
NETMASK=255.255.255.0
GATEWAY=192.168.2.2
DNS1=192.168.2.2
```
#### Bước 3: Khởi động lại card mạng
```
service network restart
```
__KẾT QUẢ__
```
br0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.2.150  netmask 255.255.255.0  broadcast 192.168.2.255
        ether d2:b2:84:9e:df:cd  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.2.133  netmask 255.255.255.0  broadcast 192.168.2.255
        inet6 fe80::c13f:a265:4dad:982f  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:f8:1c:51  txqueuelen 1000  (Ethernet)
        RX packets 2920426  bytes 2239202293 (2.0 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4796572  bytes 2889719378 (2.6 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 2655210  bytes 137587525244 (128.1 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2655210  bytes 137587525244 (128.1 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:b1:44:4e  txqueuelen 1000  (Ethernet)
        RX packets 1229  bytes 98657 (96.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1922  bytes 1721426 (1.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```