

### Tasks
 
#### 1. Secure System
 
SSH into the AWS server using the key provided for the user **ubuntu**. Initial setup includes:
 
- a) disable ssh login for root
- b) disable ssh password based login
- c) create a user for yourself and include the user in the **sudo** group
- d) ensure that all the packages installed below are locked to the version installed, however, security updates need to be installed on every run
- e) implement a firewall to restrict network access (ports only)
 
#### 2. Install required packages
 
Setup Mariadb 10+, PostgreSQL 9.4 and MongoDB. In Mariadb:
 
- a) create a secure root user login
- b) create a user (do not create a super user), create a database owned by that user
- c) allow remote access to the database created using the user above
- d) enforce ssl connection for the user (optional)
 
#### 3. System Monitoring
 
Install M/Monit on the server and ensure:
 
- a) all services created above are monitored
- b) Monit service is started automatically on port 9812
- c) secure Monit, to be visible only with credentials
 
#### 4. Setup secure static webservice (Optional)
 
Setup a secure welcome page:
 
- a) install and configure an nginx with a static sample page (take any basic html page), http
redirected to https
- b) using the dynamic dns solution (enlightNS.com) and the enlightns-cli from github, setup a
dynamically updated domain name for the server
- c) using letsencrypt, generate a valid ssl certificate for the subdomain above and setup the server
to auto-renew the certificate
- d) ensure that the https welcome page requires authentication to be visible to anyone, except those
requests coming from a 172.0.0.0/18 subnet
 
 ---------------------------------------------------------
 
 Login to the Ubuntu server using private key authentication.
 
 #### 1. Secure System
 - a) disable ssh login for root
      
       Edit the ssh config file using vi editor as below:
        $ vi /etc/ssh/sshd_config
       
       Find the below line and change it to PermitRootLogin as no:
        PermitRootLogin no
        
       After making the changes SSH need to be restarted but we will restart after next step.
       
- b) disable ssh password based login

       Edit the ssh config file using vi editor as below:
        $ vi /etc/ssh/sshd_config
        
       Find the below line if value is not set to no change it to no, which is already no in our case:    
        PasswordAuthentication no
       
       Clsoe the editor using keyboard keys <esc> : wq <enter>
       
       Now restart theSH service using below command:
        $ service ssh restart

- c) create a user for yourself and include the user in the **sudo** group

        Adding a new user:
        $ adduser <username>
        
        Adding user into sudo group:
        $ adduser <username> sudo
        
        another way to add into sudo group is:
        $ (sudo) usermod -aG <groupname> <username>
          
- d) ensure that all the packages installed below are locked to the version installed, however, security updates need to be installed on every run

       sudo apt-mark hold package_name
      

- e) implement a firewall to restrict network access (ports only)
Firewall protection can be implemented either using Uncomplicated Firewall (UFW) or iptables. UFW is installed by default on Ubuntu to ease iptables firewall configuration, ufw provides a user-friendly way to create an IPv4 or IPv6 host-based firewall. The UFW is a frontend for iptables and is particularly well-suited for host-based firewalls.

       Check the status of UFW:
       $ sudo ufw status
       $ sudo ufw status verbose
       
       Syntax:
       sudo ufw allow <port>/<optional: protocol>
       sudo ufw deny <port>/<optional: protocol>


       $ sudo ufw enable
       $ sudo ufw allow 22
       $ sudo ufw allow 80
       $ sudo ufw allow 443
       $ sudo ufw allow 21/tcp
       
       # For specific port ranges:
       $ sudo ufw allow 3000:3007/tcp
       $ sudo ufw allow 3000:3007/udp
       
       
       or 
       
       Can also use the service name rather than port number:
       $ sudo ufw enable
       $ sudo ufw allow ssh
       $ sudo ufw allow http
       $ sudo ufw allow https
       $ sudo ufw allow ftp
       
       
       https://wiki.ubuntu.com/UncomplicatedFirewall


#### 2. Install required packages
 
Setup Mariadb 10+, PostgreSQL 9.4 and MongoDB. In Mariadb:

1. Install Mariadb on ubuntu:
       
       http://www.tecmint.com/install-mariadb-in-ubuntu-and-debian/
   

 
- a) create a secure root user login

       Follow as above
       
- b) create a user (do not create a super user), create a database owned by that user
      
      CREATE DATABASE maria-db-test;
      
      CREATE USER 'maria-user-test'@'localhost' IDENTIFIED BY 'maria-user-password';
      CREATE USER 'maria-user-test'@'%' IDENTIFIED BY 'maria-user-password';
      
- c) allow remote access to the database created using the user above

      $ vi /etc/my.cnf 
      
      comment below lines:
      
      #skip-networking
      #bind-address = <some ip-address>
      
      save the file and restart mysql
      
      sudo service mysql restart
      
      
      GRANT ALL PRIVILEGES ON maria-db-test.* TO 'maria-user-test'@'localhost';
      GRANT ALL PRIVILEGES ON maria-db-test.* TO 'maria-user-test'@'%';
      FLUSH PRIVILEGES;
      
      FYI:
      If you want to give remote access to specific IP then need to create user with specific ip and grant privileges to same ip
      
      CREATE USER 'maria-user-test'@'192.168.100.33' IDENTIFIED BY 'maria-user-password';
      GRANT ALL PRIVILEGES ON maria-db-test.* TO 'maria-user-test'@'192.168.100.33';
      
      
      https://rbgeek.wordpress.com/2014/09/23/enable-remote-access-of-mysql-on-ubuntu/
      

- d) enforce ssl connection for the user (optional)


      GRANT USAGE ON maria-db-test.* TO 'maria-user-test'@'%' REQUIRE SSL;
      
      https://www.cyberciti.biz/faq/how-to-setup-mariadb-ssl-and-secure-connections-from-clients/


Install: PostgreSQL 9.4

    $ apt-get install postgresql-9.4
    


Install: MongoDB
    
    https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/
    
    
    #### 3. System Monitoring
 
Install M/Monit on the server and ensure:
 
- a) all services created above are monitored
- b) Monit service is started automatically on port 9812
- c) secure Monit, to be visible only with credentials
 
 https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-monit
 
http://www.tecmint.com/how-to-install-and-setup-monit-linux-process-and-services-monitoring-program/

