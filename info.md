# Initial discussion:
asymmetric encryption (two keys)
encrypting vs signing
x.509 certificates

# Create self-signed certificate:
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out mycert.crt -keyout mycert.key

# Examine the generated files:
openssl x509 -in mycert.crt -noout -text
openssl rsa -in mycert.key -noout -text

# Set up a web server:
docker run --detach --rm --expose 80 --expose 443 --name nginx centos:7 init
docker exec -i -t nginx bash
yum -y install epel-release net-tools telnet vim lsof psmisc
yum -y install nginx
vim /usr/share/nginx/html/index.html
nginx
ifconfig
(on host) sudo vi /etc/hosts, add entry for test.ansible.com
(on host) browse to test.ansible.com, note that https://test.ansible.com doesn't work

# Set up self-signed TLS:
vim /etc/nginx/nginx.conf
mkdir -p /etc/pki/nginx/private
(on host) docker cp mycert.crt nginx:/etc/pki/nginx/server.crt
(on host) docker cp mycert.key nginx:/etc/pki/nginx/private/server.key
nginx -s reload
Note that https://test.ansible.com works but gives error that cert is self-signed

# Create CSR from existing key:
openssl req -key mycert.key -new -out mycert.req

# Set up a certificate authority:
docker run --detach --rm --name ca centos:7 init
docker exec -i -t ca bash
yum -y install epel-release vim
yum -y install easy-rsa
cp -av /usr/share/easy-rsa/3.0.3/ ~/myca/
cd ~/myca/
./easyrsa init-pki
./easyrsa build-ca

# Sign the CSR from the web server:
(on host) docker cp mycert.req ca:/root/myca/pki/reqs
./easyrsa sign-req server mycert
(on host) docker cp ca:/root/myca/pki/issued/mycert.crt mycert-signed.crt
(on host) docker cp mycert-signed.crt nginx:/etc/pki/nginx/server.crt
(on nginx) nginx -s reload

Note that https://test.ansible.com works but gives error that cert is from unknown authority

(on host) docker cp ca:/root/myca/pki/ca.crt myca.crt
Import CA into Firefox
https://test.ansible.com is secure!

# Add CA cert to Fedora/RHEL/CentOS system trust:
sudo cp myca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust

