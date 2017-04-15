
### Instructions
 
1. A virtual machine will be provided to you for completing these tasks
2. Please write Salt code for all the steps required for configuring the machine
3. The Salt code needs to be shared using git. Please create an appropriate repository and share
it with us
4. Within the repository please add a text document detailing your process, the problems faced and
how they were resolved
5. Please share your public ssh key with us and contact us if there are any issues with accessing the machine
 
 
        New user "manan" public key: (I have added this key on server though)

          ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGTPr0SREeyJUdCFlCFTo37AYADSTxYFY55xnETgYVH/12H3FDHwhNDokM8K+a9SrXk4eHk2+VGaRMNIQdM/3+u7Z2GyA15iITBgNZrDCXOnIpVMLTMX0yzHaQL+d3n14oJCRTNJJn/oLduHXOrDJSjFAsUsBg0BGaBAVcGkVDXPmrc46Xhf+Al/Hck8hmdNr84BdRolV2JWDr08Lw1WPF12Ld1CnrUuRQ4xfQmcYhVdQPU3vSv9xPj1GqaqQ6QbWXaOq54fGvTUR1Oyuc9cLX13dg4mBO4lsb1glyW2M5F9XGLBAlChIf1j+24mIoVf0ID2DF2QHCwla/HRrKlWQD manan@devops-test-01




 Login to the Ubuntu server as a "root" user using private key authentication.

## Task

### 1. Secure System
<br /><br />
 #### - a) disable ssh login for root

 Edit the ssh config file using vi editor as below:

         $ vi /etc/ssh/sshd_config

 Find the below line and change it to PermitRootLogin as no:

         PermitRootLogin no

 After making the changes SSH need to be restarted but we will restart after next step.

<br /><br />
#### - b) disable ssh password based login

Edit the ssh config file using vi editor as below:

        $ vi /etc/ssh/sshd_config
        
 Find the below line if value is not set to "no" change it to "no", which is already "no" in our case:
 
        PasswordAuthentication no
       
Save the config file and restart the SSH service using below command:

        $ service ssh restart
          
<br /><br />
#### - c) create a user for yourself and include the user in the **sudo** group

Add a new user using below command (Password for this account is **alayacareassignment**):

         $ adduser manan
        

        
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
        
Another way to add into sudo group is (use sudo in front of command if it's not a root account):

        $ usermod -aG <groupname> <username>
          
<br /><br />
#### - d) ensure that all the packages installed below are locked to the version installed, however, security updates need to be installed on every run

Command syntax that is used to lock the installed packages:

       sudo apt-mark hold <package_name>
       
Following are the commands are used to lock the version installed (these are used after installting all the packages):
       
       sudo apt-mark hold mariadb-server
       sudo apt-mark hold postgresql-9.4
       sudo apt-mark hold mongodb-org
       sudo apt-mark hold monit

Output:
       
        manan@devops-test-01:~$ sudo vi /etc/monit/monitrc
        [sudo] password for manan:
        manan@devops-test-01:~$ sudo apt-mark hold mariadb-server
        mariadb-server set on hold.
        manan@devops-test-01:~$ 
        manan@devops-test-01:~$ sudo apt-mark hold postgresql-9.4
        postgresql-9.4 set on hold.
        manan@devops-test-01:~$ sudo apt-mark hold mongodb-org
        mongodb-org set on hold.

  
> Switching to user "manan" to perform the further operation since manan is the part of sudo group and can perform root operations. So most of the commands you will see ownward proceeded by "sudo".


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
        $ sudo ufw allow 3306    # This is added for mysql, you will find explanation in next section
       
       If you have to allow port range you can do it as well :
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
          
       Before installting Mariadb we need to import the repository key and add the MariaDB repository. Below are the commands:

         $ sudo apt-get install software-properties-common
         $ sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
         $ sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mariadb.mirror.rafal.ca/repo/10.1/ubuntu xenial main'


       Once the key is imported and the repository added, use below commands to install Mariadb 10.1:

          $ sudo apt update
          $ sudo apt install mariadb-server
       
       Follow the prompt to create password for "root" user which is setup to "alayacare-mariadb" for this assignment purposes.
       
       Once installation gets complete check the mariadb servie if it is running:
          $ sudo systemctl status mariadb
        
       If needed we can stop and start using below commands:
          $ sudo systemctl stop mariadb
          $ sudo systemctl start mariadb
       
       In order to auto start it when next time system starts it needs to be enabled, use the below command to enable Mariadb:
          $ sudo systemctl enable mariadb


In Mariadb:
 
- a) create a secure root user login

       To secure the mariadb installation use the below command:
        $ sudo mysql_secure_installation
       
       And follow the prompt, I have selected below options to secure the installation and root user login:
       
        Change the root password? [Y/n] n
        Remove anonymous users? [Y/n] Y
        Disallow root login remotely? [Y/n] Y
        Remove test database and access to it? [Y/n] Y
        Reload privilege tables now? [Y/n] Y
       
       
      To check the information about installtion like which version db is installed etc:
              
        manan@devops-test-01:~$ mysql -V
        mysql  Ver 15.1 Distrib 10.1.22-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2
        
      In order to do the next steps you need to login into maridb as root, Below is the command and output:
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

      You need to edit the default config file that can be found at /etc/mysql/my.cnf
       $ vi /etc/mysql/my.cnf
      
      Comment the below lines:   
       #bind-address           = 127.0.0.1
       #skip-networking   [Instead of skip-networking the default is now to listen only on localhost]

      Save the file and restart mysql using below command:      
       $ sudo service mysql restart
      
      Now provide the remote access to the database that is created above: 
       mysql -u root -p
       Enter password:
       Welcome to the MariaDB monitor.
  
       GRANT ALL PRIVILEGES ON maria_db_test.* TO 'maria-user-test'@'%' IDENTIFIED BY 'maria-user-password';
       FLUSH PRIVILEGES;
      
      FYI:
      If you want to give remote access to specific IP then need to create user with specific ip and grant privileges to same ip. For example:
      
      CREATE USER 'maria-user-test'@'192.168.100.33' IDENTIFIED BY 'maria-user-password';
      GRANT ALL PRIVILEGES ON maria_db_test.* TO 'maria-user-test'@'192.168.100.33';
      
#### Since mysql is running on port 3306, So this port needs to be added in ufw list using command: sudo ufw allow 3306
      

- d) enforce ssl connection for the user (optional)

      WE can use below command to enforce the  ssl connection but SSL cert needs to be created and installed for that:
       GRANT USAGE ON maria_db_test.* TO 'maria-user-test'@'%' REQUIRE SSL;
      
2. Install: PostgreSQL 9.4

       In order to install this specific version "$ apt-get install postgresql-9.4" command won't work, as It won't find the specific package by using default repository. In order to do the work around  please follow the below steps:

       1. Create the file /etc/apt/sources.list.d/pgdg.list, and add a line for the repository
          deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main
          
       2. Import the repository signing key, and update the package lists
           wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
           sudo apt-key add -
           sudo apt-get update
           
       3. Now ue the command to install:
           $ apt-get install postgresql-9.4
   

3. Install: MongoDB

       Following steps needed in order to install MongoDB:    
       
       1. Add Mongo DB repository using below command:
          $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
          
       2. Add MongoDB repository details so "apt" will know from where to download the packages:
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
      #
      eck process sshd with pidfile /var/run/sshd.pid
      start program "/etc/init.d/ssh start"
      stop program "/etc/init.d/ssh stop"
      #
      ###################################################################

Mongo db Init script is not available so it cannot be added into monit, As per mongodb website Init scripts needs to installed using automation agent.

- b) Monit service is started automatically on port 9812

      Uncomment the follow line in monit config file which is at "/etc/monit/monitrc" and change its port to "9812"
       set httpd port 9812 and


- c) secure Monit, to be visible only with credentials
      
      In the same config file "/etc/monit/monitrc" uncommented following lines to login using username and password:
      
        use address localhost  # only accept connection from localhost
        allow localhost        # allow localhost to connect to the server and
        allow admin:monit      # require user 'admin' with password 'monit', username and password can be changed as per requirement
        
        And monit needs to be relaoded after the above config change:
        $ sudo monit reload


#### 4. Setup secure static webservice (Optional)
 
Setup a secure welcome page:
 
- a) install and configure an nginx with a static sample page (take any basic html page), http redirected to https

      To install nginx use the following commands:
       $ sudo apt-get update
       $ sudo apt-get install nginx
       
      Check the status of the webserver:
       $ systemctl status nginx
      
      We need to adjust the firewall to accomodate nginx traffic since uwf has been enabled.
      
       $ sudo ufw app list
       
       Output:
       
        manan@devops-test-01:~$ sudo ufw app list
          Available applications:
            Nginx Full
            Nginx HTTP
            Nginx HTTPS
            OpenSSH
        
        Traffic on Port 80 and 443 is already allowed. So ufw should be good, unless specific port needs to be enabled.
        
       Created default home page for nginx server at : /var/www/html/index.html
       
        $ cat /var/www/html/index.html
                
        We can just browser the server ip to see the default page in browser, but other ways to verify via CLI isu sing wget command if server is running, It will fetch the file and save in the current directory which can then be viewed  using cat command.

        $ wget http://138.197.137.35
        
In order to redirect http to https we need to edit the nginx default config file as :
           
           $ cd /etc/nginx/sites-available
           $ sudo vi default
           
         Add the below code in the config:
         
         server {
               server_name alayacare.enlightns.info www.alayacare.enlightns.info;
               return 301 https://$server_name$request_uri;
                }
But this will create an issue since all traffic is going to redirect to https but we dont have SSL cert installed and https port is not enabled yet in config. Which I have explained in step c.
  
- b) using the dynamic dns solution (enlightNS.com) and the enlightns-cli from github, setup a dynamically updated domain name for the server

      Domain Name created for this assignment purposes: http://alayacare.enlightNS.info
      Currently this is pointing to the custom page I set it up on nginx server.
      
      On enlightns account I have setup A record for DNS, So A record is pointing to IP of this ubuntu server (138.197.137.35)
      
          Name 	   Content 	  Type  TTL 	
          Alayacare 	138.197.137.35     A 	  30
     
- c) using letsencrypt, generate a valid ssl certificate for the subdomain above and setup the server to auto-renew the certificate

       Following command I have used in order to install lets encrypt certs:

         $ sudo add-apt-repository ppa:certbot/certbot
         $ sudo apt-get update
         $ sudo apt-get install certbot 
         
         $ sudo certbot certonly
         
         Output is as below:
         
          manan@devops-test-01:~$ sudo certbot certonly
          Saving debug log to /var/log/letsencrypt/letsencrypt.log

          How would you like to authenticate with the ACME CA?
          -------------------------------------------------------------------------------
          1: Place files in webroot directory (webroot)
          2: Spin up a temporary webserver (standalone)
          -------------------------------------------------------------------------------
          Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
          Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
          Please enter in your domain name(s) (comma and/or space separated)  (Enter 'c'
          to cancel):alayacare.enlightns.info
          Obtaining a new certificate
          Performing the following challenges:
          http-01 challenge for alayacare.enlightns.info

          Select the webroot for alayacare.enlightns.info:
          -------------------------------------------------------------------------------
          1: Enter a new webroot
          -------------------------------------------------------------------------------
          Press 1 [enter] to confirm the selection (press 'c' to cancel): 1
          Input the webroot for alayacare.enlightns.info: (Enter 'c' to cancel):/var/www/html
          Waiting for verification...
          Cleaning up challenges
          Generating key (2048 bits): /etc/letsencrypt/keys/0000_key-certbot.pem
          Creating CSR: /etc/letsencrypt/csr/0000_csr-certbot.pem

          IMPORTANT NOTES:
           - Congratulations! Your certificate and chain have been saved at
             /etc/letsencrypt/live/alayacare.enlightns.info/fullchain.pem. Your
             cert will expire on 2017-07-12. To obtain a new or tweaked version
             of this certificate in the future, simply run certbot again. To
             non-interactively renew *all* of your certificates, run "certbot
             renew"
           - If you like Certbot, please consider supporting our work by:

             Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
             Donating to EFF:                    https://eff.org/donate-le

Now certs are installed, we need to config ngnix server to start using these certs, for that following config is needed:

           $ sudo vi /etc/nginx/sites-available/default
           
           and modified the code as below:
           
                      
           server {
         listen 443 ssl default_server;
         listen [::]:443 ssl default_server;

        ssl_certificate /etc/letsencrypt/live/alayacare.enlightns.info/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/alayacare.enlightns.info/privkey.pem;

        
        location ~ /.well-known {
                        allow all;
                }                      
        }
        
#### Auto Renewal:
For Auto renewal cert we need to create cron job, since cert will exprire after every 30 days: Use below commands to setup cron job on every monday, Its better to renew it on weekly basis since cert will expire after 30 days and if some issue comes up it will give us enough time to do the correction:


      $ sudo crontab -e
      Choose any text editor you are comfortable, I choose vim and then added below command.
      00 1 * * 1 /usr/bin/letsencrypt renew >> /var/log/letsencrypt-ssl-renew.log
      
      Save and exit. Cron job created and every monday cert will be renewed at 1:00 AM and log file will be saved at "/var/log/letsencrypt-ssl-renew.log" everytime when this crob job runs.

             
- d) ensure that the https welcome page requires authentication to be visible to anyone, except those requests coming from a 172.0.0.0/18 subnet

      Creating a basic http authencation when someone visit to: http://alayacare.enlightNS.info
      
      1. We will need htpassword to use it for passwords, For that apapche utilitiy needs to be installed as below:
           $ sudo apt-get install apache2-utils
           
      2. Once its install we need to create the username and password that we will use for authentication:
           $ sudo htpasswd -c /etc/nginx/.htpasswd alayacare-user
           
         In this case "alayacare-user" is the username and it will prompt for password and I am using "alayacare-nginx" as password.
         
         Output as below:
         
         manan@devops-test-01:~$ sudo htpasswd -c /etc/nginx/.htpasswd alayacare-user
           New password:
           Re-type new password:
           Adding password for user alayacare-user
           
         This is how password file look like:
           manan@devops-test-01:~$ cat /etc/nginx/.htpasswd
           alayacare-user:$apr1$9DqeCRtI$cevF25VbdW46eGJMAp70u.
           
       3. Last step is updating the nginx config file followed by service restart:
           $ cd /etc/nginx/sites-available
           $ sudo vi default
           
         Add the below code in the config:
         
        location / {
                # satisy any will verify either one of the condition
                # allow will bypass the authenication for mentioned subnet
                satisfy any;
                allow 172.0.0.0/18;
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                auth_basic "This is Protected content";
                auth_basic_user_file /etc/nginx/.htpasswd;
        }

        
        Save the config file and restart the service:
          $ sudo service nginx reload
          
          Complete config file looks like this:
          
              server {
               listen 80 default_server;
               listen [::]:80 default_server;
               server_name alayacare.enlightns.info www.alayacare.enlightns.info;
               return 301 https://$server_name$request_uri;
               
               }
               server {
               # SSL configuration 
                listen 443 ssl default_server;
                listen [::]:443 ssl default_server;
                server_name alayacare.enlightns.info www.alayacare.enlightns.info;
               ssl_certificate /etc/letsencrypt/live/alayacare.enlightns.info/fullchain.pem;
               ssl_certificate_key /etc/letsencrypt/live/alayacare.enlightns.info/privkey.pem;
               root /var/www/html;
               server_name _;
               
                 location ~ /.well-known {
                                 allow all;
                         }
                         location / {
                                 # satisy any will verify either one of the condition
                                 # allow will bypass the authenication for mentioned subnet
                                 satisfy any;
                                 allow 172.0.0.0/18;
                                 # First attempt to serve request as file, then
                                 # as directory, then fall back to displaying a 404.
                                 try_files $uri $uri/ =404;
                                 auth_basic "This is Protected content";
                                 auth_basic_user_file /etc/nginx/.htpasswd;
                         }
                 }
          
Any time you update  config file, service needs to be restarted. "nginx -t" command can be used to check if all the syntax are correct in the config file.

Now if you visit http://alayacare.enlightns.info/ it will promopt for username and password except if request is coming from 172.0.0.0/18 subnet and it will redirect you to https link which uses the lets encrypt cert.


 
