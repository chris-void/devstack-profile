# File name: local.conf

################ file start #############################

[[local|localrc]]
# Load the external LBaaS plugin.
enable_plugin neutron-lbaas https://git.openstack.org/openstack/neutron-lbaas

# ===== BEGIN localrc =====
DATABASE_PASSWORD=password
ADMIN_PASSWORD=password
SERVICE_PASSWORD=password
SERVICE_TOKEN=password
RABBIT_PASSWORD=password
# Enable Logging
LOGFILE=$DEST/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=True
SCREEN_LOGDIR=$DEST/logs
# Pre-requisite
ENABLED_SERVICES=rabbit,mysql,key
# Horizon
ENABLED_SERVICES+=,horizon
# Nova
ENABLED_SERVICES+=,n-api,n-crt,n-obj,n-cpu,n-cond,n-sch
IMAGE_URLS+=",https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img"
# Glance
ENABLED_SERVICES+=,g-api,g-reg
# Neutron
ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta
# Enable LBaaS V2
ENABLED_SERVICES+=,q-lbaasv2
# Cinder
ENABLED_SERVICES+=,c-api,c-vol,c-sch
# Tempest
ENABLED_SERVICES+=,tempest
# ===== END localrc =====

##################### file end ############################

Run stack.sh and do some sanity checks

./stack.sh
. ./openrc

neutron net-list  # should show public and private networks
Create two nova instances that we can use as test http servers:

#create nova instances on private network
nova boot --image $(nova image-list | awk '/ cirros-0.3.0-x86_64-disk / {print $2}') --flavor 1 --nic net-id=$(neutron net-list | awk '/ private / {print $2}') node1
nova boot --image $(nova image-list | awk '/ cirros-0.3.0-x86_64-disk / {print $2}') --flavor 1 --nic net-id=$(neutron net-list | awk '/ private / {print $2}') node2
nova list # should show the nova instances just created

#add secgroup rule to allow ssh etc..
neutron security-group-rule-create default --protocol icmp
neutron security-group-rule-create default --protocol tcp --port-range-min 22 --port-range-max 22
neutron security-group-rule-create default --protocol tcp --port-range-min 80 --port-range-max 80
Set up a simple web server on each of these instances. ssh into each instance (username ‘cirros’, password ‘cubswin:)’) and run

MYIP=$(ifconfig eth0|grep 'inet addr'|awk -F: '{print $2}'| awk '{print $1}')
while true; do echo -e "HTTP/1.0 200 OK\r\n\r\nWelcome to $MYIP" | sudo nc -l -p 80 ; done&
Phase 2: Create your load balancers
neutron lbaas-loadbalancer-create --name lb1 private-subnet
neutron lbaas-listener-create --loadbalancer lb1 --protocol HTTP --protocol-port 80 --name listener1
neutron lbaas-pool-create --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP --name pool1
neutron lbaas-member-create  --subnet private-subnet --address 10.0.0.3 --protocol-port 80 pool1
neutron lbaas-member-create  --subnet private-subnet --address 10.0.0.5 --protocol-port 80 pool1
Please note here that the “10.0.0.3” and “10.0.0.5” in the above commands are the IPs of the nodes (in my test run-thru, they were actually 10.2 and 10.4), and the address of the created LB will be reported as “vip_address” from the lbaas-loadbalancer-create, and a quick test of that LB is “curl that-lb-ip”, which should alternate between showing the IPs of the two nodes.



