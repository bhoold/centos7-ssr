# 客户端
## ssr客户端
### windows
ShadowsocksR-win-4.9.2.zip  
https://github.com/shadowsocksrr/shadowsocksr-csharp/releases  
### macos
ShadowsocksX-NG-R8.dmg  
https://github.com/qinyuhang/ShadowsocksX-NG-R/releases  
### linux
https://blog.lucien.ink/archives/208/
https://github.com/LucienShui/shadowsocksr  



## vpn客户端
### pptp
windows自带  
ios不支持  
android不支持  
linux命令方式  
### l2tp
windows自带  
ios自带  
android自带  
#### linux命令方式  
```bash
// 安装拨号
yum install epel-release
yum -y install xl2tpd ppp

// 编辑配置文件
vi /etc/xl2tpd/xl2tpd.conf
#添加以下字段
[lac myvpn]
name = username
lns = server_ip
pppoptfile = /var/run/xl2tpd/myvpn.l2tpd
ppp debug = yes
length bit = yes
redial = yes
redial timeout = 15
autodial = yes

vi /var/run/xl2tpd/myvpn.l2tpd
#添加以下字段
remotename myvpn
user "usename"
password "password"
unit 0
nodeflate
nobsdcomp
noauth
persist
nopcomp
noaccomp
maxfail 5
debug

// 重启服务
systemctl restart xl2tpd
// 开始连接
echo 'c myvpn' > /var/run/xl2tpd/l2tp-control

// 这时拨号成功的话，通过ifconfig可以看见有个ppp0的接口

// 断开连接
echo 'd myvpn' > /var/run/xl2tpd/l2tp-control

// 添加路由设置哪些ip用ppp0连接，具体参考route命令
route  [add|del] [-net|-host] target [netmask Nm] [gw Gw] [[dev] If]
route add -host target_ip dev ppp0
    //添加到主机的路由    
    
　　# route add –host 192.168.168.110 dev eth0    
    
　　# route add –host 192.168.168.119 gw 192.168.168.1    
    
　　//添加到网络的路由    
    
　　# route add –net IP netmask MASK eth0    
    
　　# route add –net IP netmask MASK gw IP    
    
　　# route add –net IP/24 eth1    
    
　　//添加默认网关    
    
　　# route add default gw IP    
    
　　//删除路由    
    
　　# route del –host 192.168.168.110 dev eth0   
```

#### 参考链接
https://www.cnblogs.com/baiduboy/p/7278715.html
https://segmentfault.com/a/1190000014160574  
https://www.iteye.com/blog/sunbin-2438364  
https://www.cnblogs.com/th-lyc/p/11226464.html  
https://sakitami.github.io/posts/cf71037e/  

