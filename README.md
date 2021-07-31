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

Confirme se o selinux está desativado

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

systemctl enable rtpproxy

# Configure os servidores mysql e apache para iniciar na inicialização.


systemctl enable mariadb

systemctl enable httpd

systemctl start mariadb

mysql_secure_installation

# Seguir instalação do MariaD

vim /etc/kamailio/kamctlrc (descomente a linha DBENGINE) "DBENGINE=MYSQL"


systemctl restart mariadb

# Criar banco de dados MySQL. Responda "y" para todas as opções

kamdbctl create    


# Configure Kamailio
Edite /etc/kamailio/kamailio.cfg , para que a parte superior do arquivo tenha a seguinte aparência:

#!KAMAILIO
#!define WITH_MYSQL
#!define WITH_AUTH
#!define WITH_USRLOCDB
#!define WITH_PRESENCE
##!define WITH_NAT
##!define WITH_TLS
#!define WITH_ACCDB

# Adicione esta parte em torno da linha #240 com as outras instruções loadmodule

###  ===== for siremis CDRs ==============
loadmodule "rtimer.so"
loadmodule "sqlops.so"


# Adicione o seguinte em torno da linha #460 no final da seção modparam antes da seção de lógica de roteamento.

#### ====== for siremis CDRs ===========
modparam("rtimer", "timer", "name=cdr;interval=300;mode=1;")
modparam("rtimer", "exec", "timer=cdr;route=CDRS")
modparam("sqlops", "sqlcon", "cb=>mysql://kamailio:kamailiorw@localhost/kamailio")

# Finalmente, adicione isso no final da seção de roteamento em torno da linha # 910

# ======================================================
# Populate CDRs Table of Siremis
# ======================================================
route[CDRS] {
    sql_query("cb","call kamailio_cdrs()","rb");
    sql_query("cb","call kamailio_rating('default')","rb");
    }


# Como alternativa, substitua o arquivo inteiro por este arquivo kamailio.cfg pré-configurado.


---------------------------------------------------------------------------------------------------



Se apontar usuários para um FQDN, você precisará especificá-lo como um alias local. O exemplo comentado está localizado em torno da linha # 170 em /etc/kamailio/kamailio.cfg. Isso é necessário se apontar clientes SIP para o FQDN em vez do endereço IP.

/* add local domain aliases */
alias="your_server_fqdn"


systemctl restart kamailio



# Criar o arquivo do Kamailio no systemd

vim /etc/systemd/system/kamailio.service

[Unit]
Description=Kamailio - the Open Source SIP Server
After=network-online.target
After=mariadb.service httpd.service

[Service]
Type=forking
Environment='CFGFILE=/etc/kamailio/kamailio.cfg'
EnvironmentFile=/etc/default/kamailio
ExecStartPre=/usr/bin/mkdir -m=2770 -p /var/run/kamailio
ExecStartPre=/usr/bin/chown kamailio:kamailio /var/run/kamailio
PIDFile=/var/run/kamailio.pid
ExecStart=/usr/sbin/kamailio -P /var/run/kamailio.pid -f $CFGFILE -m $SHM_MEMORY -M $PKG_MEMORY -u $USER -g $GROUP
ExecStopPost=/usr/bin/rm -f /var/run/kamailio.pid
Restart=on-abort

[Install]
WantedBy=multi-user.target


-----------------------------------------------------------------------------------------------------------------------------



systemctl daemon-reload
chkconfig kamailio off
systemctl enable kamailio
systemctl restart kamailio




------------------------------------------------------------------------------------------------------------------------------------

# Siremis

cd /usr/src
wget http://siremis.asipto.com/pub/downloads/siremis/siremis-4.4.0.tgz
tar zxvf siremis*
cp -a siremis*/. /var/www/html
cd /var/www/html
make prepare24
rm -rf /var/www/html/Makefile
rm -rf /var/www/html/Changelog
rm -rf /var/www/html/README
chown -R apache. /var/www/html

# Criar arquivo de configuração do Apache

vim /etc/httpd/conf.d/siremis.conf


<Directory "/var/www/html/siremis">
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        Allow from all
        <FilesMatch "\.xml$">
            Order deny,allow
            Deny from all
        </FilesMatch>
        <FilesMatch "\.inc$">
            Order deny,allow
            Deny from all
        </FilesMatch>
  </Directory>

<Directory "/var/www/html/openbiz">
    AllowOverride All
    Order deny,allow
    Deny from all
  </Directory>

<Directory "/var/www/html/misc">
    AllowOverride All
    Order deny,allow
    Deny from all
  </Directory>
  
  ------------------------------------------------------------------------------------------------
  
  systemctl restart httpd


--------------------------------------------------------------------------------------------------
# Criar usuário MySQL

mysql -e "GRANT ALL PRIVILEGES ON siremis.* TO siremis@localhost IDENTIFIED BY 'siremisrw';"

GRANT ALL PRIVILEGES ON * . * TO 'siremis'@'localhost';

FLUSH PRIVILEGES;


------------------------------------------------------------------------------------------------------

# Defina o fuso horário do PHP.

vim /etc/php.ini


# Exemplo
date.timezone = America/Sao_Paulo

systemctl restart httpd

-------------------------------------------------------------------------------------------------------


Execute o assistente de instalação na web a partir de um navegador: http://your_server_ip_or_fqdn/siremis

![image](https://user-images.githubusercontent.com/48540103/127747504-0a7c61c5-f9d3-4989-a040-1e64a9b958a7.png)




















