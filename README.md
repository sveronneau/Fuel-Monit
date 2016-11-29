# Fuel-Monit
#Monitoring of the Fuel node
This is a quick way of enabling monitoring of the Fuel Master with zero impact on the node functionality in a few minutes using Monit.

FUEL : http://docs.openstack.org/developer/fuel-docs/

MONIT : https://mmonit.com/monit/

##Step 1 - Install Monit
On the Fuel Master

*monit status

*monit: Status not available -- the monit daemon is not running

*This means Monit is installed.  If you get a command not found, just do > yum install monit

##Step 2 - Enable incoming connections 
On The Fuel Master
iptables -A INPUT -p tcp -m multiport --dports 2812 -m comment --comment "200 monit_port" -m state --state NEW -j ACCEPT
iptables-save
##Step 3 - Configuration
On The Fuel Master
mv /etc/monitrc /etc/monitrc-BAK
vi /etc/monitrc 
Paste and modify SET ALERT, SET MAILSERVER and save the following:
#
set daemon 300 # polling interval in seconds
set alert joe@doe.com # who's the lucky person who will get the emails
set httpd port 2812 and
allow admin:monit # require user 'admin' with password 'monit'
allow @monit # allow users of group 'monit' to connect (rw)
allow @users readonly # allow users of group 'users' to connect readonly
#
set mailserver mail.bar.baz, # primary mailserver
backup.bar.baz port 10025, # backup mailserver on port 10025
localhost # fallback relay
#
###############################################################################
## Includes
###############################################################################
##
## It is possible to include additional configuration parts from other files or
## directories.
#
include /etc/monit.d/*
#
mv /etc/monit.d/monit-free-space.conf /etc/monit.d/monit-free-space.conf-BAK
vi /etc/monit.d/monit-free-space.conf
Paste and save the following: 
#
check filesystem os-root with path /
if space usage > 75% then alert
#
check filesystem var with path /var
if space usage > 75% then alert
#
check filesystem var-log with path /var/log
if space usage > 75% then alert
#
check filesystem boot with path /boot
if space usage > 85% then alert
#
vi /etc/monit.d/monit-webserver.conf
Paste and modify WITH_ADDRESS and save to following:
#
check host Fuel-WebServer WITH ADDRESS 10.20.0.2
if failed
port 80 protocol http
then alert
#
##Step 4 - Start Monit
On The Fuel Master
chmod 700 /etc/monitrc
monit -t
Should return: Control file syntax OK
systemctl enable monit && systemctl start monit
Open your browser at http://fuel-ui-ip:2812
User : admin
Password : monit
