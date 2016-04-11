# Cookbook_testing_using_docker
Cookbook_testing_using_docker




Docker requires a 64-bit installation regardless of your CentOS version.
Also, your kernel must be 3.10 at minimum, which CentOS 7 runs

uname -a
Linux ip-10-45-213-178 3.10.0-229.20.1.el7.x86_64 #1 SMP Tue Nov 3 19:10:07 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
yum update

#Check if already installed
yum list installed | grep docker
docker-engine.x86_64                1.10.3-1.el7.centos        @docker-main-repo
docker-engine-selinux.noarch        1.10.3-1.el7.centos        @docker-main-repo


#To remove old docker version and everything related to docker
yum -y remove docker-engine.x86_64
rm -rf /var/lib/docker

#If not installed, install latest version using below commands

###Install either using repo or script provided by docker

sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF

yum install docker-engine


OR-----------------------

curl -fsSL https://get.docker.com/ | sh




#Start docker service

service docker start

#Start docker at boot
chkconfig docker on
chkconfig --list docker
systemctl enable docker.service
systemctl status docker.service
systemctl is-enabled docker.service

#Need to verify this settings
Restart after crash
 
 vi /etc/systemd/system/multi-user.target.wants/docker.service

[Service]

...

...

Restart=always


####Some Notes
Docker daemon. This daemon currently requires root privileges
So, only trusted users should be allowed to control your Docker daemon
Docker allows you to share a directory between the Docker host and a guest container. it can also be /root.
The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user root and other users can access it with sudo. For this reason, docker daemon always runs as the root user.

To avoid having to use sudo when you use the docker command, create a Unix group called docker and add users to it. When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the docker group.

usermod -aG docker centos

#######################



#Common issue1  

docker run hello-world

Error below

docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.

See 'docker run --help'.

.....

.....

# Solution to common issue1: Start service to fix the issue
service docker start

Redirecting to /bin/systemctl start  docker


#Verify docker daemon and its functionality
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
....
.....
......



#Some common commands
#list all running containers
docker ps
#list all running + stopped containers
docker ps -a
# Remove container by providing the id
docker rm containerid
#Remove docker image by its image name
docker rmi images


#Install Chef DK

###This should work for Centos7 Only
wget https://packages.chef.io/stable/el/7/chefdk-0.12.0-1.el7.x86_64.rpm
rpm -Uvh chefdk-0.12.0-1.el7.x86_64.rpm 



# Now create a project folder and run below command

kitchen init --driver=kitchen-docker
create  .kitchen.yml
create  chefignore
create  test/integration/default
Fetching: kitchen-docker-2.3.0.gem (100%)
WARNING:  You don't have /root/.chefdk/gem/ruby/2.1.0/bin in your PATH,
gem executables will not run.
Successfully installed kitchen-docker-2.3.0
1 gem installed



#Common issues2

#Error below

kitchen list
------Exception-------
Class: Kitchen::UserError
Message: You must first install the Docker CLI tool 
----------------------
Please see .kitchen/logs/kitchen.log for more details
Also try running `kitchen diagnose --all` for configuration

# Solution to common issue1: Start service to fix the issue

What is happening is that the driver is trying to run the CLI check as sudo but can't because it is hiding the prompt for the password.
Updating the driver config to be something like this fixed it for me..


driver:
name: docker
use_sudo: false




# Now this is how yaml(yet another xml file) file looks like

---
driver:
  name: docker
  use_sudo: false

provisioner:
  name: chef_solo

platforms:
  - name: centos-7.1

suites:
  - name: default
    run_list:
    attributes:


#install some more drivers - if you are planning to work on ec2, azure, CFN templates, asnible etc

kitchen init --driver=kitchen-cloudformation  kitchen-ec2 kitchen-inspector kitchen-inspec kitchen-ansible kitchen-azure kitchen-fog kitchen-chef-extended-attributes kitchen-all

