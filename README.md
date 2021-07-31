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
