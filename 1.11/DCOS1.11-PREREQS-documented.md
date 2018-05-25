###############################################################
########### PREREQUISITES DOCUMENTED WITH BREAKS ###########
##############################################################

# Swith to SupeUser Role
sudo su -

# Stop & Disable Firewall
systemctl stop firewalld && systemctl disable firewalld

# Set SElinux to Permissive
sed -i s/SELINUX=enforcing/SELINUX=permissive/g /etc/selinux/config
set enforce 0

# For RHEL, it is necessary to add the RHEL Entries
#subscription-manager register --username <RHEL-SUBSCRIPTION-USERNAME> --password ******** --auto-attach
#subscription-manager repos --enable=rhel-7-server-rpms
#subscription-manager repos --enable=rhel-7-server-extras-rpms
#subscription-manager repos --enable=rhel-7-server-optional-rpms

# Create Overlay File System
echo 'overlay' >> /etc/modules-load.d/overlay.conf
modprobe overlay

# Perform OS Updates
yum update -y --exclude=docker-engine,docker-engine-selinux,centos-release* --assumeyes --tolerant

# Install Utility Applications
yum install -y wget curl zip unzip ipset ntp screen bind-utils

#Install JQ
wget http://stedolan.github.io/jq/download/linux64/jq
chmod +x ./jq
cp jq /usr/bin

# Add Required Groups
groupadd nogroup
groupadd docker


# Disable ipV6
sed -i -e 's/Defaults    requiretty/#Defaults    requiretty/g' /etc/sudoers
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1


# Stop and Disable DNS Masq
systemctl stop dnsmasq
systemctl disable dnsmasq.service

#Install Docker
echo ">>> Install Docker"
curl -fLsSv --retry 20 -Y 100000 -y 60 -o /tmp/docker-engine-17.05.0.ce-1.el7.centos.x86_64.rpm \
  https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-17.05.0.ce-1.el7.centos.x86_64.rpm
curl -fLsSv --retry 20 -Y 100000 -y 60 -o /tmp/docker-engine-selinux-17.05.0.ce-1.el7.centos.noarch.rpm \
  https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-selinux-17.05.0.ce-1.el7.centos.noarch.rpm
yum -y localinstall /tmp/docker*.rpm || true
systemctl start docker
systemctl enable docker
docker run hello-world
docker info | grep Storage
echo ">>> Update /etc/hosts on boot"


# Update Hosts Fileupdate_hosts_script=/usr/local/sbin/dcos-update-etc-hosts (needs improvement)
update_hosts_unit=/etc/systemd/system/dcos-update-etc-hosts.service
mkdir -p "$(dirname $update_hosts_script)"
cat << 'EOF' > "$update_hosts_script"
#!/bin/bash
export PATH=/opt/mesosphere/bin:/sbin:/bin:/usr/sbin:/usr/bin
curl="curl -s -f -m 30 --retry 3"
fqdn=$($curl http://169.254.169.254/latest/meta-data/local-hostname)
ip=$($curl http://169.254.169.254/latest/meta-data/local-ipv4)
echo "Adding $fqdn if $ip is not in /etc/hosts"
grep ^$ip /etc/hosts > /dev/null || echo -e "$ip\t$fqdn ${fqdn%%.*}" >> /etc/hosts
EOF
chmod +x "$update_hosts_script"
cat << EOF > "$update_hosts_unit"
[Unit]
Description=Update /etc/hosts with local FQDN if necessary
After=network.target
[Service]
Restart=no
Type=oneshot
ExecStart=$update_hosts_script
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable $(basename "$update_hosts_unit")
sync


# Wait and Reboot
sleep 4
reboot


###############################################################
########### PREREQUESITES EXECUTED AS 1-SHOT SCRIPT ##########
##############################################################

sudo su -
systemctl stop firewalld && systemctl disable firewalld
sed -i s/SELINUX=enforcing/SELINUX=permissive/g /etc/selinux/config
set enforce 0
#subscription-manager register --username <RHEL-SUBSCRIPTION-USERNAME> --password ******** --auto-attach
#subscription-manager repos --enable=rhel-7-server-rpms
#subscription-manager repos --enable=rhel-7-server-extras-rpms
#subscription-manager repos --enable=rhel-7-server-optional-rpms
echo 'overlay' >> /etc/modules-load.d/overlay.conf
modprobe overlay
yum update -y --exclude=docker-engine,docker-engine-selinux,centos-release* --assumeyes --tolerant
yum install -y wget curl zip unzip ipset ntp screen bind-utils
wget http://stedolan.github.io/jq/download/linux64/jq
chmod +x ./jq
cp jq /usr/bin
groupadd nogroup
groupadd docker
sed -i -e 's/Defaults    requiretty/#Defaults    requiretty/g' /etc/sudoers
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
systemctl stop dnsmasq
systemctl disable dnsmasq.service
echo ">>> Install Docker"
curl -fLsSv --retry 20 -Y 100000 -y 60 -o /tmp/docker-engine-17.05.0.ce-1.el7.centos.x86_64.rpm \
  https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-17.05.0.ce-1.el7.centos.x86_64.rpm
curl -fLsSv --retry 20 -Y 100000 -y 60 -o /tmp/docker-engine-selinux-17.05.0.ce-1.el7.centos.noarch.rpm \
  https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-selinux-17.05.0.ce-1.el7.centos.noarch.rpm
yum -y localinstall /tmp/docker*.rpm || true
systemctl start docker
systemctl enable docker
docker run hello-world
docker info | grep Storage
echo ">>> Update /etc/hosts on boot"
update_hosts_script=/usr/local/sbin/dcos-update-etc-hosts
update_hosts_unit=/etc/systemd/system/dcos-update-etc-hosts.service
mkdir -p "$(dirname $update_hosts_script)"
cat << 'EOF' > "$update_hosts_script"
#!/bin/bash
export PATH=/opt/mesosphere/bin:/sbin:/bin:/usr/sbin:/usr/bin
curl="curl -s -f -m 30 --retry 3"
fqdn=$($curl http://169.254.169.254/latest/meta-data/local-hostname)
ip=$($curl http://169.254.169.254/latest/meta-data/local-ipv4)
echo "Adding $fqdn if $ip is not in /etc/hosts"
grep ^$ip /etc/hosts > /dev/null || echo -e "$ip\t$fqdn ${fqdn%%.*}" >> /etc/hosts
EOF
chmod +x "$update_hosts_script"
cat << EOF > "$update_hosts_unit"
[Unit]
Description=Update /etc/hosts with local FQDN if necessary
After=network.target
[Service]
Restart=no
Type=oneshot
ExecStart=$update_hosts_script
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable $(basename "$update_hosts_unit")
sync
sleep 4
reboot

#################################################
########### DC/OS 1.11 BOOTSTRAP NODE ###########
################################################

# Create Directories
mkdir -p dcos-install/genconf
cd dcos-install

# Create IP-detect
cat > genconf/ip-detect << 'EOF'
#!/usr/bin/env bash
set -o nounset -o errexit
export PATH=/usr/sbin:/usr/bin:$PATH
echo $(ip addr show eth0 | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1)
EOF

# Create public-ip-detect
cat > genconf/public-ip-detect << 'EOF'
#!/usr/bin/env bash
set -o nounset -o errexit
export PATH=/usr/sbin:/usr/bin:$PATH
echo $(ip addr show eth0 | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1)
EOF

# Create license.txt file
cat > genconf/license.txt << 'EOF'
<LICENSE KEY TEXT GOES HERE>
EOF

# Create Cloud Fault Domain Detection Script (optional)
#curl -O https://raw.githubusercontent.com/dcos/dcos/master/gen/fault-domain-detect/cloud.sh
#mv cloud.sh genconf/fault-domain-detect

#Create config.yaml
cat > genconf/config.yaml << 'EOF'
bootstrap_url: http://192.168.1.210:80
cluster_name: '<CLUSTER NAME HERE>'
fault_domain_enabled: false
ip_detect_public_filename: genconf/public-ip-detect
exhibitor_storage_backend: static
master_discovery: static
master_list:
- <masterIP 1-N>
resolvers:
- 192.168.1.200
security: permissive
superuser_password_hash: <HashGoesHere>
superuser_username: dcmennell
ssh_user: dcmennell
EOF

#Get the DC/OS 1.11 Bits
curl -O https://downloads.mesosphere.com/dcos-enterprise/stable/1.11.2/dcos_generate_config.ee.sh

#Create Password Hash (put it in config.yaml)
#sudo bash dcos_generate_config.ee.sh --hash-password Dig4Fun!

#Create the Docker Container
sudo bash dcos_generate_config.ee.sh

#Deploy the Docker Container
sudo docker run -d -p 80:80 -v ${PWD}/genconf/serve:/usr/share/nginx/html:ro nginx

# Move to your Master, Slave, and Public Slave Nodes


######################################################################
########### DC/OS 1.11 MASTER, SLAVE, PUBLIC-SLAVE INSTALL ###########
######################################################################

#MASTER
mkdir /tmp/dcos && cd /tmp/dcos
curl -O http://<bootstrapIP>:80/dcos_install.sh && sudo bash dcos_install.sh master
----------------------------------------------------------------->

#PUBLIC
mkdir /tmp/dcos && cd /tmp/dcos
curl -O http://<bootstrapIP>:80/dcos_install.sh && sudo bash dcos_install.sh slave_public
----------------------------------------------------------------->>

#PRIVATE
mkdir /tmp/dcos && cd /tmp/dcos
curl -O http://<bootstrapIP:80/dcos_install.sh && sudo bash dcos_install.sh slave
----------------------------------------------------------------->>>
