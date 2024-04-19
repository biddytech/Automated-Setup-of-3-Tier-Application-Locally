# Automated-Setup-of-3-Tier-Application-Locally
This project is to automate setup locally using NGINX, Apache Tomcat,Memcached,Rabbit and MySQL
We will be using Vagrant (a tool for buiding and managing virtual machine environment in a single workflow. With an easy-to-use workflow and focus on automation,Vagrant lowers development environment setup time,increases production parity, and makes the "works on my machine" excuse a relic of the past.We have heard the dreaded "works on my machine" excuse from developers when something breaks in staging or production. This happens becuase developers often use different setups and configurations locally than what is used in other environments. These inconsistencies waste time debugging issues that only appear outside development and result in defects reaching users.Standardized development environment (SDEs) can prevent these problems by ensuring everyone codes using identifical infrastructure and configurations. SDEs greatly boost productivity, code quality and velocity by bridging local development and deployment gaps.
We will also use Apache Tomcat which is an open-source web server and Servlet container for Java code. It's a production-ready Java development tool used to implement many types of Jakarta EE (formerly known as Java EE) specifications.
We will use Memchached which is a general-purpose distributed memory-caching system. It is often used to speed up dynamic database-driven websites by caching data and objects in RAM to reduce the number of times an external data source (such as database or API) must be read. 
We will use Rabbit MQ- an open-source message-broker software  (sometimes called message-oriented middleware) that originally implemented the Advanced Message Queuing Protocol (AMPQ) and has since been extended with a plug-in architecture to support Streaming Text Oriented Messaging Protocol (STOMP),MQ Telemetry Transport (MQTT). It is a reliable and mature messaging and streaming broker, which is easy to deploy on cloud environments, and on local machine.
NGINX (eninge X) is an open source web server software that also performs reverse proxy, load balancing, email proxy and HTTP cache services.It provides HTTPS server capabilities and is mainly designed for maximum performance and stability. It also functions as a pro-server for email communications protocols such IMAP,POP3 and SMTP.
// Challenges: During deployment, tomcat and Ngnix servers were constantly getting stuck with 'waiting for address' at every time I try to spin up the servers.
   Steps taken: I troubleshoot to make sure there was no mistake in the variables declared and also ensured the assgined IPs are correct by using 'show IP addr'
   After much steps, I backed up my vagrant file so I can thoroughly troubleshoot with the backup file. After much efforts, I destroyed all the servers using 'vagrant destroy' and the brought all servers using 'vagrant up'. This time around, I setup tomcat and Ngnix all over again and then ran 'vagrant status'. I observed that the earlier problem encountered on tomcat and Ngnix was resolved. I validated my project using 192.168.56.11:80 and it returned a login page for me to enter a username.//

   // I ran into problem with database as well. I destroyed database and brought it up again.
   This database login issue.
Perform db steps once again
1. vagrant destroy db01
2. vagrant up db01
3. vagrant ssh db01
4. sudo -i
5. wget https://raw.githubusercontent.com/devopshydclub/vprofile-project/local-setup/vagrant/Automated_provisioning/mysql.sh
6. /bin/bash mysql.sh      //
   

// Below is the Vprofile project setup 
# VPROFILE PROJECT SETUP
1. Oracle VM Virtualbox
2. Vagrant
3. Vagrant plugins
Execute below command in your computer to install hostmanager plugin
$ vagrant plugin install vagrant-hostmanager
$ vagrant plugin install vagrant-vbguest
4. Git bash or equivalent editor
VM SETUP
1. Clone source code.
2. Cd into the repository.
3. Switch to the main branch.
4. cd into vagrant/Manual_provisioning
//Bring up vm’s
$ vagrant up
//NOTE: Bringing up all the vm’s may take a long time based on various factors. If vm setup stops in the middle run “vagrant up” command again.
INFO: All the vm’s hostname and /etc/hosts file entries will be automatically updated.
$ vagrant plugin install vagrant-hostmanager
$ vagrant plugin install vagrant-vbguest

//PROVISIONING
Services
 1. Nginx
2. Tomcat
3. RabbitMQ
4. Memcache
5. ElasticSearch => Indexing/Search service
6. MySQL         => SQL Database
=> Web Service
=> Application Server
=> Broker/Queuing Agent
=> DB Caching
Setup should be done in below mentioned order
 MySQL (Database SVC) Memcache (DB Caching SVC) RabbitMQ (Broker/Queue SVC) Tomcat (Application SVC) Nginx (Web SVC)

 Login to the db vm
 $ vagrant ssh db01
$ sudo -i
Verify Hosts entry, if entries missing update the it with IP and hostnames
 # cat /etc/hosts
Install Maria DB Package
 # yum install git mariadb-server -y
Starting & enabling mariadb-server
1. MYSQL Setup
   # systemctl start mariadb
# systemctl enable mariadb

 RUN mysql secure installation script.
# mysql_secure_installation
NOTE: Set db root password, I will be using admin123 as password
  Set root password? [Y/n] Y
New password:
Re-enter new password: Password updated successfully! Reloading privilege tables..
... Success!
By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.
Remove anonymous users? [Y/n] Y ... Success!
Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.
Disallow root login remotely? [Y/n] n ... skipping.
By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.
Remove test database and access to it? [Y/n] Y - Dropping test database...
... Success!
- Removing privileges on test database...
... Success!
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
Reload privilege tables now? [Y/n] Y ... Success!

 Set DB name and users.
 # mysql -u root -padmin123
Download Source code & Initialize Database.
  mysql> create database accounts;
mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123' ; mysql> FLUSH PRIVILEGES;
mysql> exit;
 # git clone -b main https://github.com/hkhcoder/vprofile-project.git
# cd vprofile-project
# mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
# mysql -u root -padmin123 accounts
 mysql> show tables; mysql> exit;
Restart mariadb-server
# systemctl restart mariadb
Starting the firewall and allowing the mariadb to access from port no. 3306
  # systemctl start firewalld
# systemctl enable firewalld
# firewall-cmd --get-active-zones
# firewall-cmd --zone=public --add-port=3306/tcp --permanent
# firewall-cmd --reload
# systemctl restart mariadb

 2. MEMCACHE SETUP
Install, start & enable memcache on port 11211
 # sudo yum install memcached -y
# sudo systemctl start memcached
# sudo systemctl enable memcached
# sudo systemctl status memcached
# sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
# sudo systemctl restart memcached
Starting the firewall and allowing the port 11211 to access memcache
 # firewall-cmd --add-port=11211/tcp
# firewall-cmd --runtime-to-permanent
# firewall-cmd --add-port=11111/udp
# firewall-cmd --runtime-to-permanent
# sudo memcached -p 11211 -U 11111 -u memcached -d

 Login to the RabbitMQ vm
3.RABBITMQ SETUP
 $ vagrant ssh rmq01
$ sudo -i
Verify Hosts entry, if entries missing update the it with IP and hostnames
 # cat /etc/hosts
Disable SELINUX on fedora
Install Dependencies
Install Rabbitmq Server
Start & Enable RabbitMQ
Config Change
  # sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
# setenforce 0
 # curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash # sudo yum clean all
# sudo yum makecache
# sudo yum install erlang -y
 # curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash # sudo yum install rabbitmq-server -y
 # sudo systemctl start rabbitmq-server
# sudo systemctl enable rabbitmq-server
# sudo systemctl status rabbitmq-server
 # sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config' # sudo rabbitmqctl add_user test test

  # sudo rabbitmqctl set_user_tags test administrator FEDORA Changes
 # firewall-cmd --add-port=5671/tcp --permanent
# firewall-cmd --add-port=5672/tcp --permanent
# firewall-cmd --reload
# sudo systemctl restart rabbitmq-server
# reboot
Restart RabbitMQ service
 # systemctl restart rabbitmq-server
 
 Login to the tomcat vm
4.TOMCAT SETUP
 $ vagrant ssh app01
Verify Hosts entry, if entries missing update the it with IP and hostnames
# cat /etc/hosts
Update OS with latest patches
# yum update -y Set Repository
# yum install epel-release -y Install Dependencies
  # dnf -y install java-11-openjdk java-11-openjdk-devel
  # dnf install git maven wget -y
Change dir to /tmp
# cd /tmp/
Download & Tomcat Package
# wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
  # tar xzvf apache-tomcat-9.0.75.tar.gz
Add tomcat user
  # useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
         
 Copy data to tomcat home dir
# cp -r /tmp/apache-tomcat-9.0.75/* /usr/local/tomcat/ Make tomcat user owner of tomcat home dir
# chown -R tomcat.tomcat /usr/local/tomcat Setup systemctl command for tomcat
Create tomcat service file
# vi /etc/systemd/system/tomcat.service Update the file with below content
     [Unit]
Description=Tomcat
After=network.target
[Service]
User=tomcat
WorkingDirectory=/usr/local/tomcat
Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINE_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
SyslogIdentifier=tomcat-%i
[Install]
WantedBy=multi-user.target
Reload systemd files
# systemctl daemon-reload Start & Enable service
Enabling the firewall and allowing port 8080 to access the tomcat
 # systemctl start firewalld
  # systemctl start tomcat
# systemctl enable tomcat
 
  # systemctl enable firewalld
# firewall-cmd --get-active-zones
# firewall-cmd --zone=public --add-port=8080/tcp --permanent
# firewall-cmd --reload
Download Source code
CODE BUILD & DEPLOY (app01)
 # git clone -b main https://github.com/hkhcoder/vprofile-project.git Update configuration
Build code
Run below command inside the repository (vprofile-project)
# mvn install
Deploy artifact
 # systemctl stop tomcat
 # cd vprofile-project
# vim src/main/resources/application.properties
# Update file with backend server details
   # rm -rf /usr/local/tomcat/webapps/ROOT*
# cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
# systemctl start tomcat
# chown tomcat.tomcat usr/local/tomcat/webapps -R
# systemctl restart tomcat

Login to the Nginx vm
5.NGINX SETUP
 $ vagrant ssh web01
$ sudo -i
Verify Hosts entry, if entries missing update the it with IP and hostnames
# cat /etc/hosts
Update OS with latest patches
# apt update Install nginx
# apt install nginx -y Create Nginx conf file
# vi /etc/nginx/sites-available/vproapp Update with below content
upstream vproapp {
 server app01:8080;
}
server {
  listen 80;
location / {
  proxy_pass http://vproapp;
}
}
Remove default nginx conf
# rm -rf /etc/nginx/sites-enabled/default Create link to activate website


==========

After an eloborate and tedious approach in setting up a 3 -Tier application locally, I went further to automate the process using bash script as seen below:
Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true 
  config.hostmanager.manage_host = true
  
### DB vm  ####
  config.vm.define "db01" do |db01|
    db01.vm.box = "jacobw/fedora35-arm64"
    db01.vm.hostname = "db01"
    db01.vm.network "private_network", ip: "192.168.56.25"
    db01.vm.provision "shell", path: "mysql.sh"  

  end
    db01.vm.provision "shell" , path; "mysql.sh"

end

  
### Memcache vm  #### 
  config.vm.define "mc01" do |mc01|
    mc01.vm.box = "jacobw/fedora35-arm64"
    mc01.vm.hostname = "mc01"
    mc01.vm.network "private_network", ip: "192.168.56.24"
    mc01.vm.provision "shell", path: "memcache.sh"  
  end
  mc01.vm.provision "shell" , path; "memcache.sh"

end
  
### RabbitMQ vm  ####
  config.vm.define "rmq01" do |rmq01|
    rmq01.vm.box = "jacobw/fedora35-arm64"
  rmq01.vm.hostname = "rmq01"
    rmq01.vm.network "private_network", ip: "192.168.56.23"
    rmq01.vm.provision "shell", path: "rabbitmq.sh"  
  end
  rmq01.vm.provision "shell" , path; "rabbitmq.sh"

end
  
### tomcat vm ###
   config.vm.define "app01" do |app01|
    app01.vm.box = "jacobw/fedora35-arm64"
    app01.vm.hostname = "app01"
    app01.vm.network "private_network", ip: "192.168.56.22"
    app01.vm.provision "shell", path: "tomcat.sh"  
    app01.vm.provider "vmware_desktop" do |vb|
     vb.memory = "1024"
   end
    app01.vm.provision "shell" , path; "tomcat.sh"

end

  
### Nginx VM ###
  config.vm.define "web01" do |web01|
    web01.vm.box = "spox/ubuntu-arm"
    web01.vm.hostname = "web01"
  web01.vm.network "private_network", ip: "192.168.56.21"
  web01.vm.provision "shell", path: "nginx.sh"  
  end
  web01.vm.provision "shell" , path; "nginx.sh"

end


// This automation solution is repeatable and its an Infrastructure as a Code (IAAC)
 # ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
Restart Nginx
 # systemctl restart nginx


 // Deploy with 192.168.56.11:80

 
