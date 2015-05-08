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





