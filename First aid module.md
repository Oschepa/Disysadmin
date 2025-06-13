а надо было учить !!!!  Внимание, осторожно с кодом !!!!
<hr>
<h2>nftables (isp, hq-rtr, br-rtr)</h2>

nano /etc/nftables/nat.nft <br>
копировать

![описание](/1_modul/1.jpg)

![](/1_modul/2.jpg)

nano /etc/sysconfig/nftables.nft <br>
вставить

![](/1_modul/3.jpg)

systemctl enable --now nftables

<h2>Ip forward (isp, hq-rtr, br-rtr)</h2>
sysctl net.ipv4.ip_forward=1 >> /etc/sysctl.conf <br>

<h2>openvswitch (hq-sw)</h2>
nmcli device set [имя интерфейса] managed no <br>
ip a a [ip-адрес/маска] dev ens3 <br>
ip r a default via [ip-адрес gateway] dev ens3 <br>
dnf install NetworkManager-ovs -y  <br>
systemctl restart NetworkManager <br>
systemctl enable –now openvswitch.service <br>
nmcli connection add con-name sw-con type ovs-bringe ifname switch0 ipv4.method disabled <br>
nmcli connection add con-name ens3-p type ovs-port ifname ens3-p master switch0  <br>
nmcli connection add con-name ens4-p type ovs-port ifname ens4-p master switch0 ovs-port.tag 100 <br>
nmcli connection add con-name ens5-p type ovs-port ifname ens5-p master switch0 ovs-port.tag 200 <br>
nmcli connection add con-name mgmt-p type ovs-port ifname mgmt-p master switch0 ovs-port.tag 999 <br>
nmcli connection add con-name ens3-i type ethernet ifname ens3 master ens3-p <br>
nmcli connection add con-name ens4-i type ethernet ifname ens4 master ens4-p <br>
nmcli connection add con-name ens5-i type ethernet ifname ens5 master ens5-p 

nmcli connection add con-name mgmt-i type ovs-interface ifname mgmt master mgmt-p ipv4.method manual ipv4.addresses [ip-адрес/маска] ipv4.gateway [ip-адрес gateway] <br>

nmcli connection up ens4-i <br>
nmcli connection up ens5-i <br>
nmcli connection up mgmt-i <br>
nmcli connection up ens3-i <br>

<h2>dhcp (hq-rtr)</h2>
dnf install dhcp-server -y <br>
nano /usr/share/doc/dhcp-server/dhcpd.conf.examle <br>
копировать от option-domain-name до конца subnet <br>
nano /etc/dhcp/dhcpd.conf <br>
вставить 

![](/1_modul/4.jpg)

nano /etc/sysconfig/dhcpd <br>
DHCPDARGS=ens4.200 <br>
systemctl enable –now dhcpd <br>
проверить на hq-cli и поменять hostname <br>
su – (пароль toor) <br>
hostnamectl hostname hq-cli.au-team.irpo <br>

<h2>dns (hq-srv)</h2>

dnf install bind -y <br>
nano /etc/named.conf

![](/1_modul/5.jpg)

![](/1_modul/6.jpg)

cd /var/named <br>
cp named.localhost au-team <br>
cp named.loopback 10.arpa <br>
chgrp named au-team 10.arpa <br>
nano au-team <br>

![](/1_modul/7.jpg)

nano 10.arpa

![](/1_modul/8.jpg)

nano /etc/system/resolved.conf <br>

![](/1_modul/9.jpg)

systemctl enable –now named

<h2>gre (hq-rtr, br-rtr)</h2>

nmtui

![](/1_modul/10.png)

![](/1_modul/11.jpg)

![](/1_modul/12.jpg)

nano /etc/NetworkManager/system-connections/gre1-nmconnection 

![](/1_modul/13.jpg)

<h2>Пользовать net_admin (hq-rtr, br-rtr)</h2>

adduser net_admin <br>
passwd net_admin (пароль P@$$word) <br>
nano /etc/sudoers <br>

![](/1_modul/14.jpg)

<h2>Пользователь sshuser (hq-srv, br-srv)</h2>

adduser sshuser -u 1010 <br>
passwd sshuser (пароль P@ssw0rd) <br>
nano /etc/sudoers <br>

![](/1_modul/15.jpg)

nano /etc/ssh/sshd_config

![](/1_modul/16.jpg)

nano /etc/issue.net

![](/1_modul/17.jpg)

systemctl restart sshd

<h2>frr (hq-rtr, br-rtr)</h2>

dnf install frr -y <br>
nano /etc/frr/daemons <br>
Прописать ospfd=yes <br>
systemctl enable –now frr <br>
vtysh <br>
conf t <br>
router ospf <br>
ospf router 1.1.1.1 (на втором 2.2.2.2) <br>
network [ip-сеть/маска] area 0 <br>
area 0 autentication message-digest <br>
interface [ens3, ens4, ens4.100-999] <br>
ip ospf passive <br>
interface gre1 <br>
ip ospf network point-to-point <br>
ip ospf autentication message-digest <br>
ip ospf message-digest key 1 md5 PASSWORD <br>
do wr mem или wr mem <br>
