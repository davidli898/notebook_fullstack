http://luokr.com/p/12

Virtualbox虚拟机网络配置(NAT + Host-only - Bridged)

日常工作中，常常会用虚拟机，在里面安装Server，搭建服务端环境供开发调试，这种使用场景一般都需要虚拟机能够正常访问外部网络，同时宿主机必须可以访问虚拟机。在Virtualbox中，虚拟机访问外部网络一般是使用配置起来最简单的NAT模式，但纯NAT模式下，宿主机不能访问虚拟机，必须使用Bridged或者Host-only模式才可以。在这两个模式下，虚拟机都可以获得一个可用的IP地址，宿主机通过该IP地址即可访问虚拟机。

关于Virtualbox的网络接入模式，不了解的同学可以自行Google一下，这部分资料其实都挺齐全的，这篇文章主要是简单的介绍（记录）一下在Virtualbox虚拟机中使用NAT模式访问外部公共网络（互联网），再结合Host-only模式，令宿主机同时可以用虚拟机的静态IP地址访问虚拟机的配置实现。该配置相对于单纯使用Bridged模式的好处在于：即使没有外部公用网络，宿主机也可以无障碍的访问虚拟机，不会影响使用。

在Bridged模式下，虚拟机和宿主机处于同等地位，就像是一台真实主机一样存在于局域网中，可以分配到一个网络中独立的IP，所有网络功能都和在网络中的真实机器一样，网络中的其它机器（包括宿主机）也可以访问到这台虚拟机。同时，如果网络断开，即便虚拟机和宿主机其实是在一台物理机器上，宿主机也不能够访问虚拟机。而Host-only模式，可以理解为Virtualbox在宿主机中模拟出一张专供虚拟机使用的网卡，所有虚拟机都是连接到该网卡上的，虚拟机可以通过该网卡IP访问宿主机，同时Virtualbox提供一个DHCP服务，虚拟机可以获得一个内部网IP，宿主机可以通过该IP访问虚拟机。如果单纯使用Host-only模式，则虚拟机不能连接外部公共网络。

在有外部网络的情况下，假如需要将虚拟机开放给网络中的其它机器访问，比如让同事连上虚拟机做开发测试等工作，那么就可以直接使用Bridged模式，该模式也仅需要占用公共网络中的一个IP地址，但日常使用环境中，有时候不一定有公共网络可以用，假如使用Bridged模式，则虚拟机连不上，开发工作也做不了，此时Host-only模式就是一个不错的选择，若是再配合NAT模式，则外部公共网络可用时，虚拟机也可以访问外部公共网络。

下面简单介绍下使用这几个模式时需要做的相关配置。

首先在Virtualbox中的全局配置(呼出快捷键ctrl+g)界面的网络配置中，点击右侧添加按钮，增加一个Host-only网络。查看该网络的详情，可以看到：

001.png

可以看出，该网络是192.168.56.0，可供分配使用的IP地址是192.168.56.101 - 192.168.56.254。

打开虚拟机的网络配置，将网卡1的连接方式选为“网络地址转换(NAT)”，网卡2的连接方式选为“仅主机（Host-only）适配器”，如下图所示：

002.png

假如使用Bridged模式，则需要将连接方式选为“桥接网卡”。

保存后，启动虚拟机，虚拟机以Ubuntu server 12.04为例，打开配置文件 /etc/network/interfaces 加入如下配置：
``` bash
# The loopback network interface 
auto lo 
iface lo inet loopback 
 
# The primary network interface 
auto eth0 
iface eth0 inet dhcp 
 
# Virtualbox Host-only mode
auto eth1 
iface eth1 inet static 
address 192.168.56.101
netmask 255.255.255.0 
#network 192.168.56.0 
 
# Virtualbox Bridged mode
#auto eth1
#iface eth1 inet static 
#address 192.168.0.190 
#netmask 255.255.255.0 
#gateway 192.168.0.1 
```
该配置将虚拟机在内部网络中的IP地址设置为静态分配（192.168.56.101），方便宿主机在hosts中绑定该IP访问虚拟机。保存配置后，执行如下命令重启网络服务：
```bash
$ sudo /etc/init.d/networking restart
```
即可实现虚拟机使用NAT通过宿主机来正常访问外部网络，同时因为使用了Host-only模式，宿主机可以通过虚拟机在内部网络的IP地址访问虚拟机，即使外部网络不可用也不影响宿主机对虚拟机的访问。