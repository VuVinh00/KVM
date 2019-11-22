## 1. Linux Namespace

### 1.1 Linux namespace là gì?

- **Network namespace** là khái niệm cho phép bạn cô lập môi trường mạng network trong một host. Namespace phân chia việc sử dụng các khái niệm liên quan tới network như devices, địa chỉ address, ports, định tuyến và các quy tắc tường lửa vào trong một hộp ( box ) riêng biệt, chủ yếu là ảo hóa mạng trong một máy chạy kernel duy nhất

- Mỗi network namespace có bản định tuyến riêng, các thiết lập iptables riêng cung cấp cơ chế NAT và lọc đối với các máy ảo thuộc namespace đó, Linux network namespace cũng cung cấp thêm khả năng để chạy các tiến trình riêng biệt trong mỗi nội bộ namespace

- Network namespace được sử dụng trong nhiều dự án như Openstack, Docker và Mininet

### 1.2 Một số thao tác quản lý và làm việc với linux network namespace

Ban đầu khi khởi động hệ thống Linux, bạn sẽ có một namespace mặc định đã chạy trên hệ thống và mọi tiến trình mới tạo sẽ kế thừa kế namespace này, gọi là root namespace.

#### 1.2.1 List namespace

Để liệt kê tất cả các network namespace trên hệ thống sử dụng câu lệnh :

```
ip netns
# or
ip netns list
```

<img src="https://github.com/vjnkvt/Images/blob/master/nm.png">

Nếu chưa thêm bất kì network namespace nào thì đầu ra màn hình sẽ để trống, root namespace sẽ không được liệt kê khi sử dụng câu lệnh ip netns list

#### 1.2.2 Add namespace

Để thêm một network namespace sử dụng lệnh ``ip netns add <namespace_name>`` :

Ví dụ : tạo thêm 2 namespace là ns1 và ns2 : 

```
 ip netns add ns1
 ip netns add ns2
```

Mỗi khi thêm vào một namespace , một file mới sẽ được tạo trong thư mục ``/var/run/netns`` với tên giống như tên namespace ( không bao gồm file của root namespace )

```
[root@localhost netns]# ls -a
total 0
-r--r--r--.  1 root root   0 Nov 22 15:56 ns1
-r--r--r--.  1 root root   0 Nov 22 15:56 ns2

```

1.2.3 Executing commands trong namespaces

Để xử lý các lệnh trong một namespace ( không phải root namespace ) sử dụng ``ip netns exec <namespace> <command>``

Ví dụ chạy lệnh ``ip a`` liệt kê địa chỉ các interface trong namespace ns1 

```
[root@localhost netns]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:8c:45:26 brd ff:ff:ff:ff:ff:ff
    inet 10.0.10.136/24 brd 10.0.10.255 scope global noprefixroute dynamic ens33
       valid_lft 1444sec preferred_lft 1444sec
    inet6 fe80::a8de:5472:6ecb:873a/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
[root@localhost netns]# ip netns exec ns1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

Kết quả đầu ra sẽ khác so với khi chạy câu lệnh ``ip a`` ở chế độ mặc định ( trong root namespace ). Mỗi namespace sẽ có một môi trờng mạng cô lập và có các interface và bảng định tuyến riêng

- Để sử dụng các câu lệnh với namespace ta sử dụng command bash để xử lý các câu lệnh trong riêng namespace đã chọn :

```
 ip netns exec <namespace_name> bash
 ip a #se chi hien thi thong tin trong namespace <namespace_name> 
```

Thoát khỏi vùng làm việc của namespace gõ exit :

```
[root@localhost netns]# ip netns exec ns1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
[root@localhost netns]# ^C
[root@localhost netns]# ip netns exec ns1 bash
[root@localhost netns]# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
[root@localhost netns]# exit
exit
[root@localhost netns]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default q                                                                       len 1000
    link/ether 00:0c:29:8c:45:26 brd ff:ff:ff:ff:ff:ff
    inet 10.0.10.136/24 brd 10.0.10.255 scope global noprefixroute dynamic ens33
       valid_lft 1262sec preferred_lft 1262sec
    inet6 fe80::a8de:5472:6ecb:873a/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

#### 1.2.4 Gán interface vào một network namespace

- Sử dụng câu lệnh sau để gán interface vào namespace : 

``ip link set <interface_name> netns <namespace_name>``

- Gán một interface if1 vào namespace ns1 sử dụng lệnh sau :

``ip link set if1 netns ns1``

#### 1.2.5 Xóa namespace 

Xóa namespace sử dụng câu lệnh :

``ip netns delete <namespace_name>``

## 2. Một số bài lab thử nghiệm tính năng của linux network namespace

### 2.1 Kết nối 2 namespace sử dụng Open vSwitch

#### 2.1.1 Kết nối sử dụng veth

Tạo một Open vSwitch trong namespace mặc định của hệ thống có tên là ``my_switch``

``ovs-vsctl add-br my_switch``

Để kết nối namespace đến ``my_switch`` vừa được tạo, dùng cặp **veth** sẽ được giải thích dưới đây :

- **veth ( virtual ethernet )** : là một loại thiết bị mạng mà luôn luôn đi theo 1 cặp ( pairs ). Có thể tưởng tượng rằng từ **pair** nhắc đến ở đây như là một cái ống, mọi thứ gửi đến một đầu ống sẽ đi ra ngoài ở đầu bên kia 

```
$ sudo ip link add type veth

$ ip address

14: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 76:75:43:9f:a8:9e brd ff:ff:ff:ff:ff:ff
15: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether fa:92:ed:48:bd:b4 brd ff:ff:ff:ff:ff:ff
```

Có 2 device di theo một cặp : mọi thứ khi được gửi đến **veth0@veth1** sẽ đi ra ở **veth1@veth0**

Bây giờ khi xóa 1 trong 2 **veth** thì device còn lại cũng sẽ bị xóa theo

Quay trở lại việc kết nối namespace với **my_switch**, tạo **veth** kết nối namespace **ns1** với **my_switch** 

``ip link add veth-netns type veth peer name veth1-ovs``

Câu lệnh trên đã xác định thành phần của mỗi cặp với 2 interface là veth-netns và veth1-ovs. Vì vậy, **veth-netns** sẽ được thiết lập ở trong namespace **ns1** và **veth1-ovs** sẽ kết nối đến **my_switch** :

Đặt **veth-netns** vào trong namespace **ns1** :

``ip link set veth-netns netns ns1``

*namespace là một môi trường mạng độc lập và namespace mặc định của hệ thống là tách biệt với các namespace khác. Nếu điều này là đúng, veth-netns sẽ không được thấy trong namespace mặc định của hệ thống kể từ khi nó được đặt trong namespace ns1*

Kết nối veth còn lại trong cặp ( **veth1-ovs** ) với **my_switch** 

``ovs-vsctl add-port my_switch veth1-ovs``

veth1-ovs đã trở thành một port của **my_switch**

Làm các bước tương tự với namespace **ns2**

```
ip link add veth2-netns type veth peer name veth2-ovs
ip link set veth2-netns netns ns2
ovs-vsctl add-port my_switch veth2-ovs
```

##### Bringing up devices

Trong namespace mặc định của hệ thống : 

```
ip link set veth1-ovs up
ip link set veth2-ovs up
```

Trong namespace ns1 

```
ip netns exec ns1 ip link set dev lo up
ip netns exec ns1 ip link set dev veth-netns up
```

Trong namespace ns2 

```
ip netns exec ns2 ip link set dev lo up
ip netns exec ns2 ip link set dev veth2-netns up
```

##### Gán địa chỉ IP 

Bước tiếp theo sẽ gán địa chỉ IP cho các device veth-netns và veth2-netns trong các namespace ns1, ns2 tương ứng 

```
ip netns exec ns1 ip addr add 10.0.0.1/24 dev veth-netns
ip netns exec ns2 ip addr add 10.0.0.2/24 dev veth2-netns
```

```
[root@vuvinh netns]# ip netns exec ns1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
21: veth-netns@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group d                                                                       efault qlen 1000
    link/ether 22:b2:88:34:46:e2 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.0.0.1/24 scope global veth-netns
       valid_lft forever preferred_lft forever
    inet6 fe80::20b2:88ff:fe34:46e2/64 scope link
       valid_lft forever preferred_lft forever
[root@vuvinh netns]# ip netns exec ns2 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
26: veth2-netns@if25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group                                                                        default qlen 1000
    link/ether 5a:25:bd:2a:94:5e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.0.0.2/24 scope global veth2-netns
       valid_lft forever preferred_lft forever
    inet6 fe80::5825:bdff:fe2a:945e/64 scope link
       valid_lft forever preferred_lft forever
```

##### Ping test 

```
[root@vuvinh netns]# ip netns exec ns1 ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.732 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.296 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.077 ms
^C
--- 10.0.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.077/0.368/0.732/0.272 ms
```

## Tham khảo

https://techzones.me/linux/linux-network-namespace/
