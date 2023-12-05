# Indirect-Checks_NRPE

Monitoring HTTP port 80 with NagiosXI on Indirect Service on another Host.
##
#backstory

i had vm server with NagiosXI (host A), then remote linux host with NRPE install (host B), now i want to monitoring another vm (host C) and only used the NRPE installed on Host B.

note:
- only host A and host B can Communicate 
- host A and host C not will Allow to Comunicate

in my case :
- Host A -> Nagios Server    -> 10.10.10.222
- Host B -> Agent Server     -> 10.10.10.100
- Host C -> Prod-Web Server  -> 10.10.30.40

##
##
##

#Setup Host A
- Step 1: Install NagiosXI
  
    source : https://assets.nagios.com/downloads/nagiosxi/docs/Installing-Nagios-XI-Manually-on-Linux.pdf

- Step 2: Allow Firewall (i used CentOS for Host A)
 ```bash
firewall-cmd --permanent --add-port=5666/tcp
firewall-cmd --permanent --add-port=5667/tcp 
```

#Setup Host B
- Step 3: Install NRPE

    source : https://assets.nagios.com/downloads/nagiosxi/docs/Installing_The_XI_Linux_Agent.pdf

- Step 4: Allow IP Nagios Server in Host B
  
nano /etc/xinetd.d/nrpe

```bash
 # description: NRPE (Nagios Remote Plugin Executor)
service nrpe
{
    disable         = no
    per_source      = 25
    socket_type     = stream
    port            = 5666
    wait            = no
    user            = nagios
    group           = nagios
    server          = /usr/local/nagios/bin/nrpe
    server_args     = -c /usr/local/nagios/etc/nrpe.cfg --inetd
    only_from       = 127.0.0.1 10.10.10.222
    log_on_success  =
}
```
- Step 5: allow Firewall (i used Ubuntu for Host B)
  
```bash
ufw allow 5666/tcp
ufw allow 5667/tcp
```

- Step 6: make sure NRPE Running and listen in Host B

```bash
netstat -tulpn | grep LISTEN
```

##
#Check NRPE Version on Host A to Host B
from Host A to directory
```bash
cd /usr/local/nagios/libexec/
```

then

```bash
./check_nrpe -H 10.10.10.100
```
##
#Setup Host C
- Step 7: install Web-Server

  in this case, i used Nginx for example then make sure port 80 on listen and make ufw allow from 10.10.10.100 to only allow comunicate host B and host C

##
#Config CCM Nagios (Core Config Manager) in Host A
- Step 8: add custom command plugin

open
```bash
nano /usr/local/nagios/etc/commands.cfg
```

then

```bash
define command {
    command_name    indirect_http3
    command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_remote3
}
```

- Step 9: add custom service plugin

open
```bash
nano /usr/local/nagios/etc/services/10.10.10.100.cfg
```

then

```bash
define service {
    host_name                Jump Agent
    service_description      HTTP3 indirect Prod_NO NRPE Client
    use                      xiwizard_nrpe_service
    check_command            check_nrpe!indirect_http3! -c check_remote3
    max_check_attempts       5
    check_interval           5
    retry_interval           1
    check_period             xi_timeperiod_24x7
    notification_interval    60
    notification_period      xi_timeperiod_24x7
    contacts                 nagiosadmin
    _xiwizard                linux-server_legacy
    register                 1
}
```
note : adjust the IP and hostname above according to your environment,

and restart xinetd service
```bash
service xinetd restart
```
##
#Config NRPE in Host B
- Step 10: to directory NRPE.cfg
open
```bash
nano /usr/local/nagios/etc/nrpe.cfg
```
then

```bash
#The Following Indrecrt3 without nrpe client
command[check_remote3]=/usr/local/nagios/libexec/check_nrpe -H $ARG1$ -t 30 -c $ARG2$
command[check_remote3]=/usr/local/nagios/libexec/check_http -I 10.10.30.40
```

and restart xinetd service
```bash
service xinetd restart
```
##
#access GUI NagiosXI
- Step 11: add custom service in GUI
```bash
10.10.10.222/nagiosxi/includes/components/ccm/?cmd=view&type=service)
```

adjust CCM in GUI nagios like this

Config Name		: 10.10.10.100 #hostname host B

Description		: HTTP3 indirect host C

Check command	: indirect_http

Command view	: $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_remote



- Step 12: before save config in GUI, make Sure the command its run

choose 'Run Check Command'


the output will like this : 
[nagios@localdomain ~]$ /usr/local/nagios/libexec/check_nrpe -H 10.10.10.100 -c check_remote3
HTTP OK: HTTP/1.1 200 OK - 854 bytes in 0.003 second response time |time=0.002670s;;;0.000000 size=854B;;;0

- Step 13: Apply Configuration.

##
