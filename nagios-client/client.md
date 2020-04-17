# Nagios client setup on centos7  
# On client server
install the EPEL repository:  
`sudo yum install epel-release`  
Now install Nagios Plugins and NRPE:  
`sudo yum install nrpe nagios-plugins-all`  
update the NRPE configuration file as below  
`sudo vi /etc/nagios/nrpe.cfg`

change the allowed host as below  
`allowed_hosts=127.0.0.1,192.168.3.28`  

add below commands to monitor cpu, memory, total process, root disk, mnt directory in /etc/nagios/nrpe.cfg

```
command[check_users]=/usr/lib64/nagios/plugins/check_users -w 5 -c 10  
command[check_load]=/usr/lib64/nagios/plugins/check_load -r -w .15,.10,.05 -c .30,.25,.20  
command[check_sda1]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/sda1  
command[check_zombie_procs]=/usr/lib64/nagios/plugins/check_procs -w 5 -c 10 -s Z  
command[check_total_procs]=/usr/lib64/nagios/plugins/check_procs -w 150 -c 200  
command[check_mem]=/usr/lib64/nagios/plugins/check_mem.pl -f -w 20 -c 10  
command[check_root_disk]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /  
command[check_mnt_disk]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /mnt  
```  
*Need to download check_mem.pl from below link as it wasnt available in default *
```
cd /usr/lib64/nagios/plugins/  
sudo wget https://raw.githubusercontent.com/justintime/nagios-plugins/master/check_mem/check_mem.pl  
sudo chmod +x check_mem.pl  
```


Now start the nrpe service and configure it to run on boot time  
`sudo systemctl start nrpe.service`  
`sudo systemctl enable nrpe.service`  

*Noted that whenever you change anything in nrpe.cfg restart the service.*  

So, our client setup is almost done. Now we need to add this host in our nagios server.

# Go to nagios server

First download the check_mem plugin  
```
cd /usr/local/nagios/libexec/  
sudo wget https://raw.githubusercontent.com/justintime/nagios-plugins/master/check_mem/check_mem.pl  
sudo chmod +x check_mem.pl  
```

create a new configuration file at below location with the name of your host  

`sudo /usr/local/nagios/etc/servers/nagios-2.cfg`  

[check this file for final config](nagios-2.cfg)

After that restart the nagios service

`sudo systemctl restart nagios.service`


