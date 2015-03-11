ubuntu 14.04 
1.安装依赖包

apt-get install build-essential libwrap0-dev libpam0g-dev libdbus-1-dev \
  libreadline-dev libnl-route-3-dev libprotobuf-c0-dev libpcl1-dev libopts25-dev \
  autogen libgnutls28 libgnutls28-dev libseccomp-dev libhttp-parser-dev
  
2.下载ocserv 
wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.10.0.tar.xz


3.生成证书

certtool --generate-privkey --outfile ca-key.pem
cat <<_EOF_> ca.tmpl

cn = "sean vpn"
organization = "sean"
serial = 1
expiration_days = 9999
ca
signing_key
cert_signing_key
crl_signing_key
_EOF_
  
certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem

4.生成服务器证书
certtool --generate-privkey --outfile server-key.pem
cat <<_EOF_> server.tmpl

cn = "sean vpn"
organization = "sean"
serial = 2
expiration_days = 9999
signing_key
encryption_key
tls_www_server
_EOF_  

certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem

5.配置配置文件
     cat /usr/local/src/ocserv-0.10.0/doc/sample.config |grep -v '^#' | grep -v '^$' > /etc/ocserv/config
	 
	 
