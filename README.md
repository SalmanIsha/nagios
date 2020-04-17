# Nagios Setup on centos 7  
#### List of servers  
Nagios Server IP:  192.168.3.28  
Nagios Client IP:  192.168.3.29  

# Prerequisite  
A LAMP stack is also required in order to setup nagios  

### Install Build Dependencies  

We will install nagios from source file. So need to install some dependencies.  
`sudo yum install gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel xinetd unzip`  

### Create Nagios User and Group  
`sudo useradd nagios`  
`sudo groupadd nagcmd`  
`sudo usermod -a -G nagcmd nagios`  

### Install Nagios Core
`cd ~`  
`curl -L -O https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz`  
`tar xvf nagios-*.tar.gz`  
`cd nagios-*`
`./configure --with-command-group=nagcmd`  
`make all`  

###### Below make commands to install Nagios, init scripts, and sample configuration files  

`sudo make install`  
`sudo make install-commandmode`  
`sudo make install-init`  
`sudo make install-config`  
`sudo make install-webconf`  

###### In order to issue external commands via the web interface to Nagios, we must add the web server user, apache, to the nagcmd group:

`sudo usermod -G nagcmd apache`  

#### Install Nagios Plugins

`cd ~`  
`curl -L -O http://nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz`  
`tar xvf nagios-plugins-*.tar.gz`  
`cd nagios-plugins-*`  
`./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl`  
`make`  
`sudo make install`  

#### Install NRPE  

`cd ~`  
`curl -L -O http://downloads.sourceforge.net/project/nagios/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz`  
`tar xvf nrpe-*.tar.gz`  
`cd nrpe-*`  
`./configure --enable-command-args --with-nagios-user=nagios --with-nagios-group=nagios --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu`  
`make all`  
`sudo make install`  
`sudo make install-xinetd`  
`sudo make install-daemon-config`  

Open the xinetd startup script in an editor:  

`sudo vi /etc/xinetd.d/nrpe`  

add nagios server private ip address as below  

`only_from = 127.0.0.1 192.168.3.28`  

Restart the xinetd service to start NRPE:  

`sudo service xinetd restart`  

#### Configure Nagios  

Open nagios config file  

`sudo vi /usr/local/nagios/etc/nagios.cfg`  

and uncomment below line:  

`#cfg_dir=/usr/local/nagios/etc/servers`  

Now create the directory that will store the configuration file for each server that you will monitor:  

`sudo mkdir /usr/local/nagios/etc/servers`  


#### Configure check_nrpe Command  

`sudo vi /usr/local/nagios/etc/objects/commands.cfg`  

Add the following to the end of the file:  


>define command{  
        command_name check_nrpe  
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$  
>}  

#### Configure Apache  

`sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin`  
`sudo systemctl daemon-reload`  
`sudo systemctl start nagios.service`  
`sudo systemctl restart httpd.service`  
`sudo chkconfig nagios on`  

#### Accessing the Nagios Web Interface  

>http://nagios_server_ip/nagios  

[Nagios client setup on centos7](nagios-client/client.md)



