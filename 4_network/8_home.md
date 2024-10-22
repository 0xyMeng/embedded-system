
# 家庭网络

在装宽带的时候，运营商会给个光猫，给个路由器。

光猫可以是桥接模式，然后由路由器完成 PPPoE 拨号。也可以光猫完成拨号，路由器的 wan 接光猫的 lan。

路由器除了 wan 口，还有几个 lan 口，这几个 lan 口可以看做一台交换机，通过一个内部的 lan 接到路由器上，这个内部 lan 口的 ip 是 192.168.1.1 ，这通常是路由器的管理页面，也是网关地址。

一台电脑想上网，通过网线接入 lan 口，一般设置为自动获取 ip 的方式。然后 DHCP 方式获得 IP。

DHCP 是一个应用层的协议；在传输层使用 UDP 协议发送，源端口号为 68，目标端口号为 67；在网络层包装上 IP 信息，源 ip 填上 0.0.0.0 目标 IP 为 255.255.255.255；在网络接口层，包装上 MAC 信息，源 MAC 就是主机的，目标 MAC 为 FF:FF:FF:FF:FF:FF 。然后一个包就可以发送出去了，交换机收到以后只能解读 MAC 这部分的东西，其他东西对他来说都是乱码。然后包广播了，路由器上开了这个服务，并监听着 67 端口，然后包装好数据，原路返回。

如果 DHCP 服务器没了，那么到期会自动释放，然后给自己一个 169.254.x.x 的地址。如果有两个 DHCP 服务器，那么也会有问题。

也可以手动设置 IP 地址。




