# 服务端

## ssr 服务端

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


