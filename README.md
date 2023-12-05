# Indirect-Checks_NRPE

Monitoring HTTP port 80 with NagiosXI Indirect Service on Another Host.
##
#backstory

i am had server monitoring with Nagios XI (host A), with remote linux host with NRPE install (host B), now i want to monitoring another remote linux host (host C) and using the NRPE installed on Host B.

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
##

#Setup Host A
- Step 1: Install NagiosXI
  
    source : https://assets.nagios.com/downloads/nagiosxi/docs/Installing-Nagios-XI-Manually-on-Linux.pdf

- Step 2: Allow Firewall (in this case i used CentOS for Host A)
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
- Step 5: allow Firewall (in this case i used Ubuntu for Host B)
  
```bash
ufw allow 5666/tcp
ufw allow 5667/tcp
```

- Step 6: make sure NRPE Running and listen in Host B

```bash
netstat -tulpn | grep LISTEN
```

#Check NRPE Version on Host A to Host B
from Host A to directory
```bash
cd /usr/local/nagios/libexec/
```
then
```bash
./check_nrpe -H 10.10.10.100
```
#Setup Host C

