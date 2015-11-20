# Setup ocserv  on Ubuntu 14.04

---

AnyConnect 的好处是基于 HTTPS，证书可以申请 StartSSL 的，而且配置也不很复杂。
另外配置文件里发现了很多专门为商业化/企业服务定制的选项，例如最大同时在线客户端数量，同账户最大在线设备数量等等。
OpenConnect 是 AnyConnect 的开源实现。目前 0.8.0 以上版本需要 GnuTLS 3.1 以上版本，所以我们就可以直接在 Ubuntu 14.04 中搭建。另外 就算是基于 HTTPS，这货也是要用 TUN 设备的，所以 OpenVZ 用户们请注意。

1.安装依赖包
```bash
$ apt-get install build-essential libwrap0-dev libpam0g-dev libdbus-1-dev \
  libreadline-dev libnl-route-3-dev libprotobuf-c0-dev libpcl1-dev libopts25-dev \
  autogen libgnutls28 libgnutls28-dev libseccomp-dev libhttp-parser-dev
```
 
2.下载ocserv 源码
```bash
$ wget https://fossies.org/linux/privat/ocserv-0.10.9.tar.gz
  or  wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.10.9.tar.xz
$ tar zxvf ocserv-0.10.9.tar.gz
$ cd ocserv-0.10.9
```

3.编译安装
```bash
$ ./configure --prefix=/opt/ocserv
$ make
$ make install
$ mkdir -p /opt/ocserv/etc/
$ cat doc/sample.config | grep -v '^#' | grep -v '^$' > /opt/ocserv/etc/config
```

4.生成证书
```bash
$ apt-get install gnutls-bin
$ certtool --generate-privkey --outfile ca-key.pem
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
$ certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem
```

5.生成服务器证书
```bash
$ certtool --generate-privkey --outfile server-key.pem
$ cat <<_EOF_> server.tmpl
cn = "alex vpn"
organization = "alex"
serial = 2
expiration_days = 9999
signing_key
encryption_key
tls_www_server
_EOF_  

$ certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem

$ mkdir -p /opt/ocserv/etc/ssl
$ cp *.pem /opt/ocserv/etc/ssl/
```

6.配置
> File: `/opt/ocserv/etc/config`
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
```bash
$ /opt/ocserv/bin/ocpasswd -c /opt/ocserv/etc/passwd username
```
按提示输入两次密码。

8.修改系统允许转发
```bash
$ vim /etc/sysctl.conf
#修改这行
net.ipv4.ip_forward=1
#保存退出
$ sysctl -p
```

9.开启NAT
```bash
$ iptables -t nat -A POSTROUTING  -o  venet0  -s 192.168.111.0/24 -j MASQUERADE
```

10.启动ocserv
```bash
$ /opt/ocserv/sbin/ocserv  -c /opt/ocserv/etc/config
```
搞定，折腾完毕，世界安静了 :)







