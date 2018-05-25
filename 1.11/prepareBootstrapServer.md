#################################################
########### DC/OS 1.11 BOOTSTRAP NODE ###########
################################################

# Create Directories
''
mkdir -p dcos-install/genconf
cd dcos-install
''

# Create IP-detect (based on cloud or on-prem)
'''
cat > genconf/ip-detect << 'EOF'
#!/usr/bin/env bash
set -o nounset -o errexit
export PATH=/usr/sbin:/usr/bin:$PATH
echo $(ip addr show eth0 | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1)
EOF
'''

# Create public-ip-detect (if on Public cloud)
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

# Create Cloud Fault Domain Detection Script (optional - this works for cloud... se example for file-based)
curl -O https://raw.githubusercontent.com/dcos/dcos/master/gen/fault-domain-detect/cloud.sh
mv cloud.sh genconf/fault-domain-detect

#Create config.yaml
cat > genconf/config.yaml << 'EOF'
bootstrap_url: http://<bootstrapIP>:80
cluster_name: '<CLUSTER NAME HERE>'
fault_domain_enabled: false
ip_detect_public_filename: genconf/public-ip-detect
exhibitor_storage_backend: static
master_discovery: static
master_list:
- <masterIP 1>
- <masterIP 2>
- <masterIP 3>
resolvers:
- 192.168.1.200
security: permissive
superuser_password_hash: <HashGoesHere>
superuser_username: <dcosAdminUserID>
ssh_user: <sudoUserName>
EOF

#Get the DC/OS 1.11 Bits
curl -O <Insert Download Link Here>

#Create Password Hash (put it in config.yaml)
#sudo bash dcos_generate_config.ee.sh --hash-password <dcosAdminPassword>

#Create the Docker Container
sudo bash dcos_generate_config.ee.sh

#Deploy the Docker Container
sudo docker run -d -p 80:80 -v ${PWD}/genconf/serve:/usr/share/nginx/html:ro nginx

#Move to your Master, Slave, and Public Slave Nodes
