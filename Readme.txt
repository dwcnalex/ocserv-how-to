ubuntu 14.04 

1.安装依赖包
apt-get install build-essential libwrap0-dev libpam0g-dev libdbus-1-dev \
  libreadline-dev libnl-route-3-dev libprotobuf-c0-dev libpcl1-dev libopts25-dev \
  autogen libgnutls28 libgnutls28-dev libseccomp-dev libhttp-parser-dev
  
2.下载ocserv 
wget https://fossies.org/linux/privat/ocserv-0.10.9.tar.gz
tar zxvf ocserv-0.10.9.tar.gz
cd ocserv-0.10.9

3.编译安装
./configure --prefix=/opt/ocserv
make
make install
mkdir -p /opt/ocserv/etc/
cat doc/sample.config | grep -v '^#' | grep -v '^$' > /opt/ocserv/etc/config


4.生成证书
apt-get install gnutls-bin
certtool --generate-privkey --outfile ca-key.pem
cat <<_EOF_> ca.tmpl
cn = "alex vpn"
organization = "alex"
serial = 1
expiration_days = 9999
ca
signing_key
cert_signing_key
crl_signing_key
_EOF_
  
certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem

5.生成服务器证书
certtool --generate-privkey --outfile server-key.pem
cat <<_EOF_> server.tmpl
cn = "alex vpn"
organization = "alex"
serial = 2
expiration_days = 9999
signing_key
encryption_key
tls_www_server
_EOF_  

certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem

mkdir -p /opt/ocserv/etc/ssl
cp *.pem /opt/ocserv/etc/ssl/

6.配置
     file:/opt/ocserv/etc/config
```
auth = "plain[/opt/ocserv/etc/passwd]"
tcp-port = 443
udp-port = 443
run-as-user = nobody
run-as-group = daemon
socket-file = /var/run/ocserv-socket
server-cert = /opt/ocserv/etc/ssl/server-cert.pem
server-key = /opt/ocserv/etc/ssl/server-key.pem
ca-cert = /opt/ocserv/etc/ssl/ca-cert.pem
isolate-workers = true
max-clients = 16
max-same-clients = 10
keepalive = 32400
dpd = 90
mobile-dpd = 1800
try-mtu-discovery = false
cert-user-oid = 0.9.2342.19200300.100.1.1
tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-VERS-SSL3.0"
auth-timeout = 40
min-reauth-time = 300
max-ban-score = 50
ban-reset-time = 300
cookie-timeout = 300
deny-roaming = false
rekey-time = 172800
rekey-method = ssl
use-occtl = true
pid-file = /var/run/ocserv.pid
device = vpns
predictable-ips = true
default-domain = mixblue.com
ipv4-network = 192.168.111.0
ipv4-netmask = 255.255.255.0
dns = 8.8.8.8
dns = 8.8.4.4
ping-leases = false
#route = 10.10.10.0/255.255.255.0
#route = 192.168.0.0/255.255.0.0
#no-route = 192.168.5.0/255.255.255.0
cisco-client-compat = true
```

7.创建用户
/opt/ocserv/bin/ocpasswd -c /opt/ocserv/etc/passwd username
按提示输入两次密码。

8.修改系统配置，允许转发
vim /etc/sysctl.conf
#修改这行
net.ipv4.ip_forward=1
#保存退出
sysctl -p

9.NAT
iptables 规则
```
iptables -t nat -A POSTROUTING -j SNAT --to-source <server ip> -o <nic>
```
10.启动
/opt/ocserv/sbin/ocserv  -c /opt/ocserv/etc/config
把 <server ip> 和 <nic> 改为服务器公网 IP 和对应网卡的名称。

：）搞定，折腾完毕，AnyConnect 客户端可以成功使用了。


