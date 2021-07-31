# ProjetoKamailio
Projeto Centos 7  Kamailio e Siremis 


Recursos:
* comunicação ponto a ponto segura
* chamadas de voz pessoa a pessoa ou em grupo
* videochamadas pessoa a pessoa
* Compartilhamento de tela
* Mensagem instantânea
* status de presença
* registros de detalhes de chamadas

# Pré instalação

yum -y update && yum -y groupinstall core && yum -y groupinstall base && yum -y install epel-release


yum install httpd mariadb-server php php-mysql php-gd php-curl rtpproxy

# Confirme se o selinux está desativado

# Kamailio
nano /etc/yum.repos.d/kamailio44.repo



[home_kamailio_v4.4.x-rpms]
name=RPM Packages for Kamailio v4.4.x (CentOS_7)
type=rpm-md
baseurl=http://download.opensuse.org/repositories/home:/kamailio:/v4.4.x-rpms/CentOS_7/
gpgcheck=1
gpgkey=http://download.opensuse.org/repositories/home:/kamailio:/v4.4.x-rpms/CentOS_7//repodata/repomd.xml.key
enabled=1







yum install kamailio kamailio-presence kamailio-mysql kamailio-tls 


# Rtproxy é opcional se não estiver usando NAT

# Configure os servidores mysql e apache para iniciar na inicialização.


systemctl enable mariadb

systemctl enable httpd






