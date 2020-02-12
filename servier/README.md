# 服务端

## ssr 服务端
### 关联文件
ssr.sh
install_bbr_centos.sh

### 命令
```bash
yum -y update
yum -y install wget
chmod +x ssr.sh && bash ssr.sh
// 到这会弹出安装ssr信息，修改密码和端口就好啦，其他保持默认

// BBR提速
bash install_bbr_centos.sh
sysctl net.ipv4.tcp_congestion_control // 测试是否开启bbr


// 防火墙规则
#将SSR服务端口放行，2333换成你SSR设置的端口
firewall-cmd --zone=public --add-port=2333/tcp --permanent
#查看端口是否开启成功，一般都是sucess提示
firewall-cmd --query-port=2333/tcp
#重启下防火墙
firewall-cmd --reload
#查看下服务端口是否在内
firewall-cmd --list-port
```

## l2tp 服务端
### 关联文件
l2tp.sh

### 命令
```bash
// 阿里云服务器要去掉云盾


// 脚本自动添加防火墙规则和转发设置,会生成l2tp.log文件
chmod +x l2tp.sh
bash l2tp.sh
// 到这按提示选择

// 服务名称
systemctl restart ipsec xl2tpd
ipsec verify // 检测ipsec配置


// 涉及的的配置文件
/etc/ipsec.conf
/etc/ipsec.secrets
/etc/pptpd.conf
/etc/xl2tpd/xl2tpd.conf
/etc/ppp/options.xl2tpd
/etc/ppp/options.pptpd
/etc/ppp/chap-secrets // 帐号文件
/etc/sysctl.conf // 转发设置
```

## pptp和l2tp 服务端
### 关联文件
vpn-script-for-centos7.sh

### 命令
```bash
chmod +x vpn-script-for-centos7.sh && bash vpn-script-for-centos7.sh
ipsec verify // 确保都是ok

// 设置防火墙规则
ens=$(ls /etc/sysconfig/network-scripts/ | grep 'ifcfg-e.*[0-9]' | cut -d- -f2)
systemctl restart firewalld.service
systemctl enable firewalld.service
firewall-cmd --set-default-zone=public
firewall-cmd --add-interface=m=$ens
firewall-cmd --add-port=1723/tcp --permanent #pptp
firewall-cmd --add-port=1701/udp--permanent #l2tp
firewall-cmd --add-masquerade --permanent
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -i $ens -p gre -j ACCEPT
firewall-cmd --reload

// 设置mtu值
cat > /etc/ppp/ip-up.local << END
/sbin/ifconfig $1 mtu 1400
END
chmod +x /etc/ppp/ip-up.local

// 重启服务，开机运行
systemctl enable pptpd ipsec xl2tpd
systemctl restart pptpd ipsec xl2tpd
```

## pptp 服务端
### 命令
```bash
modprobe ppp-compress-18 && echo yes // 如果不返回yes，则运行yum install kernel-devel安装支持
cat /dev/net/tun // 返回File descriptor in bad state或中文类似说明表示支持隧道。如果没有，需要服务商开通TUN/TAP功能
strings /usr/sbin/pppd |grep -i mppe | wc --lines // 如果以上命令输出为“0”则表示不支持；输出为“30”或更大的数字就表示支持，MPPE(Microsoft Point to Point Encryption)
yum update
yum -y install ppp pptpd

// 编辑配置文件，如果是只读，执行chmod a+w filepath添加写属性
vi /etc/pptpd.conf // 配置ip
localip 192.168.0.1 #找到这行去掉前面的#号
remoteip 192.168.0.234-238,192.168.0.245 #找到这行去掉前面的#号

vi /etc/ppp/options.pptpd // 配置dns
ms-dns  8.8.8.8 #这是谷歌的，你也可以改成阿里巴巴的或者其它
ms-dns  8.8.4.4

vi /etc/ppp/chap-secrets // 添加vpn账号
#client     server  secret      IP addresses
username        pptpd   password        *

vi /etc/sysctl.conf // 修改内核设置
net.ipv4.ip_forward = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.eth0.accept_redirects = 0
net.ipv4.conf.eth0.rp_filter = 0
net.ipv4.conf.eth0.send_redirects = 0
net.ipv4.conf.lo.accept_redirects = 0
net.ipv4.conf.lo.rp_filter = 0
net.ipv4.conf.lo.send_redirects = 0
net.ipv4.conf.ppp0.accept_redirects = 0
net.ipv4.conf.ppp0.rp_filter = 0
net.ipv4.conf.ppp0.send_redirects = 0 

sysctl -p // 使内核修改生效

systemctl restart pptpd
netstat -antup | grep pptpd // 查看端口
systemctl enable pptpd // 开机运行

// 防火墙
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE // 转发
// or
iptables -t nat -F
iptables -t nat -A POSTROUTING -s 192.168.0.234/24 -j SNAT --to 你的服务器的公网IP

// 查看开放的端口
firewall-cmd --list-services
firewall-cmd --list-port

// 添加pptp端口
firewall-cmd --permanent --zone=public --add-port=1723/tcp // pptp端口
firewall-cmd --permanent --zone=public --add-port=47/tcp // gre端口
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -i eth0 -p gre -j ACCEPT
firewall-cmd --reload // 重载规则
```