# stop containers
rm mycert.* myca.crt
sudo rm /etc/pki/ca-trust/source/anchors/myca.crt
sudo update-ca-trust
sudo vi /etc/hosts
Remove trust from Firefox
