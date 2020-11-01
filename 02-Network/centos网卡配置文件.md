文件路径：/etc/sysconfig/network-scripts/ifcfg-eth0  
配置IP、掩码、网关:ethX

DEVICE="eth0"

    此配置关联至的设备。设备名要与文件ifcfg-后ude内容保持一致

BOOTPROTO=none

    引导协议：{none|static|dhcp|bootp}

HWADDR="00:0C:29:26:62:92"

    MAC地址：要与真实MAC地址保持一致，可省略

NM_CONTROLLED="yes"

    是否接受NetworkManager脚本控制：{yes|no}

ONBOOT="yes"

    是否开机自动启动此网络设备{yes|no}

TYPE="Ethernet"

    设备类型Etheraget Bridge（桥接）

UUID="14351f7f-a726-4dfc-966e-dfb1f352f226"

    唯一标识，可省略

IPADDR=

    ip地址

NETMASK=

    掩码

GATEWAY=

    默认网关

DNS1=

    DNS1服务地址

IPV6INIT=no

    是否开启ipv6

USERCTL=no

    是否允许普通用户操作网卡

PEERDNS={yes|no}

    是否允许DHCP服务分配地址时直接更新/etc/resolv。conf中的DNS服务器地址
