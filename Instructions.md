

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
 
 New user "manan" public key:
 
 manan@devops-test-01:~/.ssh$ cat manan-alayacare-ssh.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGTPr0SREeyJUdCFlCFTo37AYADSTxYFY55xnETgYVH/12H3FDHwhNDokM8K+a9SrXk4eHk2+VGaRMNIQdM/3+u7Z2GyA15iITBgNZrDCXOnIpVMLTMX0yzHaQL+d3n14oJCRTNJJn/oLduHXOrDJSjFAsUsBg0BGaBAVcGkVDXPmrc46Xhf+Al/Hck8hmdNr84BdRolV2JWDr08Lw1WPF12Ld1CnrUuRQ4xfQmcYhVdQPU3vSv9xPj1GqaqQ6QbWXaOq54fGvTUR1Oyuc9cLX13dg4mBO4lsb1glyW2M5F9XGLBAlChIf1j+24mIoVf0ID2DF2QHCwla/HRrKlWQD manan@devops-test-01

 
 
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

        Adda new user using below command:
         $ adduser manan
        
        <password for this account is alayacareassignment>
        
        Below was the Output:
                   
            Adding user `manan' ...
            Adding new group `manan' (1000) ...
            Adding new user `manan' (1000) with group `manan' ...
            Creating home directory `/home/manan' ...
            Copying files from `/etc/skel' ...
            Enter new UNIX password:
            Retype new UNIX password:
            passwd: password updated successfully
            Changing the user information for manan
            Enter the new value, or press ENTER for the default
                    Full Name []: Mananpreet Singh
                    Room Number []: 1908
                    Work Phone []: 000-000-0000
                    Home Phone []:
                    Other []:
            Is the information correct? [Y/n] Y

        
        Add user into sudo group using below command:
        $ adduser manan sudo
        
        Another way to add into sudo group is <use sudo in front of command if it's not root account>:
        $ usermod -aG <groupname> <username>
          
- d) ensure that all the packages installed below are locked to the version installed, however, security updates need to be installed on every run

       sudo apt-mark hold <package_name>
       
       Following are the commands that I used to lock the version installed:
       
       sudo apt-mark hold mariadb-server
       sudo apt-mark hold postgresql-9.4
       sudo apt-mark hold mongodb-org
       sudo apt-mark hold monit

       OutPut:
       
        manan@devops-test-01:~$ sudo vi /etc/monit/monitrc
        [sudo] password for manan:
        manan@devops-test-01:~$ sudo apt-mark hold mariadb-server
        mariadb-server set on hold.
        manan@devops-test-01:~$ 
        manan@devops-test-01:~$ sudo apt-mark hold postgresql-9.4
        postgresql-9.4 set on hold.
        manan@devops-test-01:~$ sudo apt-mark hold mongodb-org
        mongodb-org set on hold.

  
  
  
#### Switching to user "manan" to perform the further operation since manan is the part of sudo group and can perform root operations. So most of the commands you will see ownward proceeded by "sudo".



- e) implement a firewall to restrict network access (ports only)

Firewall protection can be implemented either using Uncomplicated Firewall (UFW) or iptables. UFW is installed by default on Ubuntu to ease iptables firewall configuration, ufw provides a user-friendly way to create an IPv4 or IPv6 host-based firewall. The UFW is a frontend for iptables and is particularly well-suited for host-based firewalls.

       Check the status of UFW:
       $ sudo ufw status
       or
       $ sudo ufw status verbose
       
       Syntax to allow ordeny ports in ufw firewall are:
        $ sudo ufw allow <port>/<optional: protocol>
        $ sudo ufw deny <port>/<optional: protocol>

       Enable the firewall suing below command:
        $ sudo ufw enable
       
       Added traffic to below incoming ports for this assignment purposes:       
        $ sudo ufw allow 22
        $ sudo ufw allow 80
        $ sudo ufw allow 443
        $ sudo ufw allow 21/tcp
       
       If you have to allow port range you can do it as :
        $ sudo ufw allow 3000:3007/tcp
        $ sudo ufw allow 3000:3007/udp
       
       
       If service name need to be enable rather than port number it can be done as below:
        $ sudo ufw enable
        $ sudo ufw allow ssh
        $ sudo ufw allow http
        $ sudo ufw allow https
        $ sudo ufw allow ftp
       
       
#### 2. Install required packages
 
Setup Mariadb 10+, PostgreSQL 9.4 and MongoDB. 

1. Install Mariadb on ubuntu:
          
        Before installting MAriadb we need to import the repository key and add the MariaDB repository. Here are the commands that I used :

       $ sudo apt-get install software-properties-common
       $ sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
       $ sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mariadb.mirror.rafal.ca/repo/10.1/ubuntu xenial main'


       Once the key is imported and the repository added, I used below commands to install Mariadb 10.1:

        $ sudo apt update
        $ sudo apt install mariadb-server
       
       Follow the prompt to create password for "root" user which I setup to "alayacare-mariadb" for this assignment purposes.
       
       Once installation completed check the mariadb servie if it is running:
       
        $ sudo systemctl status mariadb
        
       If needed we can stop and start using below commands:
        $ sudo systemctl stop mariadb
        $ sudo systemctl start mariadb
       
       In order to auto start it when next time system starts it needs to be enabled using:
       
        $ sudo systemctl enable mariadb


In Mariadb:
 
- a) create a secure root user login

       To secure the mariadb installtion use the below command:
        $ sudo mysql_secure_installation
       
       And follow the prompt, I have selected below options to secure the installation and root user login:
       
       Change the root password? [Y/n] n
       Remove anonymous users? [Y/n] Y
       Disallow root login remotely? [Y/n] Y
       Remove test database and access to it? [Y/n] Y
       Reload privilege tables now? [Y/n] Y
       
       
       Below command give you the information of which version it is installed:
              
        manan@devops-test-01:~$ mysql -V
        mysql  Ver 15.1 Distrib 10.1.22-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2
        
       In order to do the next steps we need to login into maridb as root and command that I used as below:
         manan@devops-test-01:~$ mysql -u root -p
         Enter password:
         Welcome to the MariaDB monitor. 

             
- b) create a user (do not create a super user), create a database owned by that user
      
       CREATE DATABASE maria_db_test
       CREATE USER 'maria-user-test'@'localhost' IDENTIFIED BY 'maria-user-password';
      
      In order to provide "maria_db_test" db access to user use the below command:
       GRANT ALL PRIVILEGES ON maria_db_test.* TO 'maria-user-test'@'localhost';
       FLUSH PRIVILEGES; 
 
- c) allow remote access to the database created using the user above

      You need to edit the default config file that can be foun at /etc/mysql/my.cnf
      $ vi /etc/mysql/my.cnf
      
      Comment below lines:   
       #bind-address           = 127.0.0.1
       #skip-networking   [ Instead of skip-networking the default is now to listen only on localhost, so  skip-networking line is not there]

      Saved the file and restart mysql using below command:      
       $ sudo service mysql restart
      
      Now provide the remore access to database created above: 
      mysql -u root -p
      Enter password:
      Welcome to the MariaDB monitor.
  
       GRANT ALL PRIVILEGES ON maria_db_test.* TO 'maria-user-test'@'%' IDENTIFIED BY 'maria-user-password';
       FLUSH PRIVILEGES;
      
      FYI:
      If you want to give remote access to specific IP then need to create user with specific ip and grant privileges to same ip
      
      CREATE USER 'maria-user-test'@'192.168.100.33' IDENTIFIED BY 'maria-user-password';
      GRANT ALL PRIVILEGES ON maria_db_test.* TO 'maria-user-test'@'192.168.100.33';
      
#### Since mysql is running on port 3306, So this port needs to be added in ufw list using command: sudo ufw allow 3306
      

- d) enforce ssl connection for the user (optional)

      WE can use below command to enforce the  ssl connection but SSL cert needs to be created and installed for that:
       GRANT USAGE ON maria-db-test.* TO 'maria-user-test'@'%' REQUIRE SSL;
      
2. Install: PostgreSQL 9.4

       In order to install this specific version "$ apt-get install postgresql-9.4" command won't work, It will not find the specific package. In order to do the work around  please follow the below steps:

       1. Create the file /etc/apt/sources.list.d/pgdg.list, and add a line for the repository
          deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main
          
       2. Import the repository signing key, and update the package lists
           wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
           sudo apt-key add -
           sudo apt-get update
           
       3. $ apt-get install postgresql-9.4
   


3. Install: MongoDB

       Following steps has been done to install MongoDB:    
       
       1. Adding Mongo DB repository using below command:
          $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
       2. Add MongoDB repository details so apt will know where to download the packages:
          $ echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
       3. Update the package list:
          $ sudo apt-get update
       4. Install MongodB
          $ sudo apt-get install -y mongodb-org
       5. Once installtion finish, Start the service:
          $ sudo systemctl start mongod
       6. Check the status of the service if it is running:
          $ sudo systemctl status mongod
       7. In order to start auto at boot it needs to be enabled as:
          $ sudo systemctl enable mongod  

    
    #### 3. System Monitoring
 
Install M/Monit on the server and ensure:

     To install Monit use the below command:
      $ sudo apt-get install monit
 
- a) all services created above are monitored
        
      To monitor the services it needs to be added in config file, Added below commands to monitor services:
      
      
      ######################################################
      #               Custon Config to monitor services
      #####################################################
      #
      #
      check process mysqld with pidfile /var/run/mysqld/mysqld.pid
      start program = "/etc/init.d/mysqld start"
      stop program = "/etc/init.d/mysqld stop"
      #
      #
      check process postgresql with pidfile /var/lib/postgresql/9.4/main/postmaster.pid
      start program = "/etc/init.d/postgresql start"
      stop program = "/etc/init.d/postgresql stop"
      #
      #
      #check process mongodb with pidfile "/var/lib/mongodb/mongod.lock"
      # start program = "/sbin/start mongodb"
      # stop program = "/sbin/stop mongodb"
      #
      #
      eck process sshd with pidfile /var/run/sshd.pid
      start program "/etc/init.d/ssh start"
      stop program "/etc/init.d/ssh stop"
      #
      ###################################################################

Mongo db Init script is not available so it cannot be added into monit, As per mongodb website it needs to installed using automation agent.

- b) Monit service is started automatically on port 9812

      Uncomment the follow line in monit config file which is at "/etc/monit/monitrc" and change its port to "9812"
       set httpd port 9812 and


- c) secure Monit, to be visible only with credentials
      
      In the same config file "/etc/monit/monitrc" uncommented following lines to login using username and pass:
      
        use address localhost  # only accept connection from localhost
        allow localhost        # allow localhost to connect to the server and
        allow admin:monit      # require user 'admin' with password 'monit'
        
        And monit needs to be relaoded after config change:
        $ sudo monit reload


#### 4. Setup secure static webservice (Optional)
 
Setup a secure welcome page:
 
- a) install and configure an nginx with a static sample page (take any basic html page), http redirected to https

      To install nginx use the follwoing commands:
       $ sudo apt-get update
       $ sudo apt-get install nginx
       
      Check the status of the webserver:
       $ systemctl status nginx
      
      We need to adjust the firewall since uwf has been enabled.
      
       $ sudo ufw app list
       
       Outpput:
        manan@devops-test-01:~$ sudo ufw app list
          Available applications:
            Nginx Full
            Nginx HTTP
            Nginx HTTPS
            OpenSSH
        
        Traffic on Prt 80 and 443 is already allowed. So it should be good.
        
       Default home page for nginx server is at : /var/www/html/index.nginx-debian.html
       
        $ cat /var/www/html/index.nginx-debian.html
        
        
        We can just browser the server ip to see the default page in browser, but other ways to verify via CLI if server is running:
        
        wget http://138.197.137.35
        
        It will fetch the file and save in the current directory which can be viewed using cat command.

       
  
- b) using the dynamic dns solution (enlightNS.com) and the enlightns-cli from github, setup a dynamically updated domain name for the server
- c) using letsencrypt, generate a valid ssl certificate for the subdomain above and setup the server to auto-renew the certificate
- d) ensure that the https welcome page requires authentication to be visible to anyone, except those requests coming from a 172.0.0.0/18 subnet
 
