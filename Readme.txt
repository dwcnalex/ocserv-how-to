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
mkdir /opt/ocserv/etc/
cp doc/sample.config /opt/ocserv/etc/config


4.生成证书

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

6.配置
     cat /usr/local/src/ocserv-0.10.9/doc/sample.config |grep -v '^#' | grep -v '^$' > /etc/ocserv/config
     /etc/ocserv/config
    ···
    ···

7.创建用户
/opt/ocserv/bin/ocpasswd -c /opt/ocserv/etc/passwd username
按提示输入两次密码。

8.NAT
iptables 规则
···
iptables -t nat -A POSTROUTING -j SNAT --to-source <server ip> -o <nic>
···

搞定
折腾完毕，AnyConnect 客户端可以成功使用了。
把 <server ip> 和 <nic> 改为服务器公网 IP 和对应网卡的名称。

