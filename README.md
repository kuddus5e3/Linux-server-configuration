# Linux-server-configuration
This project is about create a linux server and connecting it to our web application.

# IP Address
IP Address: 13.233.31.36

# Host Name
Host Name: ec2-13-233-31-36.ap-south-1.compute.amazonaws.com

# Amazon Lightsail server setup
1. Open **Amazon Lightsail** website and create a account in it.
2. After creating an account go to log in page.
3. After log in, click on **Create Instance**.
4. Select Platform (**Linux/Unix**).
5. Select Blueprint (Select **OS Only** and then select **Ubuntu 16.04 LTS**).
6. Scroll down and click on **Create**, it will take some time to setup.
7. After it is set up, you will see **running** in the left corner of the status card. Write down the **public IP address** on a paper as you will use it a lot in the following steps.
8. Click on **Manage**, in next page it shows **connect**, now scroll down and you'll see an link like **Account Page**. 
9. Click on **Account Page** and click the download button to download your **Private Key**. It is a .pem file, not the rsa file in other step-by-step guides, but we can still use it to log into our server.
10. Click the **Networking** tab and find the **Add another** at the bottom. Add port 123 and 2200. Amazon Lightsail allows only port 22 and 80 by default, no matter how you set it up in ubuntu's ufw.
        
        Application      Protocol      Port range
        SSH              TCP           22
        HTTP             TCP           80
        Custom           UDP           123
        Custom           TCP           2200
# Server Configuration
1. Save the **.pem** file where is your vagrant file is located.
2. Now we have to move/copy our **.pem** file into **.ssh** folder. I've saved my file as **k.pem**. To move/copy our **.pem** file follow below command.
        
        1. sudo cp -R /vagrant/YOURFILENAME.pem ~/.ssh/YOURFILENAME.pem
3. We need to make our public key usable and secure. Going back to your terminal and write the command
        
        2. sudo chmod 600 ~/.ssh/YOURFILENAME.pem
4. Now we use this key to log into our Amazon Lightsail server
        
        3. sudo ssh -i ~/.ssh/YOURFILENAME.pem ubuntu@13.233.31.36
5. Amazon Lightsail does not allow you to log in as a root user, but we can switch to become a root user. By executing the following command we can become root user
        
        4. sudo su -
After we became root user, we've have to create a user called **grader**.
        
        5. sudo adduser grader
6. Now create a new file under the sudoers directory
        
        6. sudo nano /etc/sudoers.d/grader
and fill that with
        
        grader ALL = (ALL:ALL) ALL
then save it.(*Control X, then type Y, then hit enter key on your keyboard*)
7. Run the following commands to update all packages and install package:
        
        7. sudo apt-get update
        8. sudo apt-get upgrade
        9. sudo apt-get install finger
8. Open a new terminal (**Windows + r**) and give following command
        
        10. ssh-keygen -f ~/.ssh/udacity_key.rsa
9. Stay on the same terminal window and give the following command to read the public key. Copy that public key
        
        11. cat ~/.ssh/udacity_key.rsa.pub
10. Going back to the first terminal window where you are logged into **Amazon Lightsail** as the **root user**, move to grader's folder by
        
        12. cd /home/grader
11. Create a **.ssh** directory
        
        13. mkdir .ssh
12. Create a file to store the public key
        
        14. touch .ssh/authorized_keys
13. Edit authorized_keys file
        
        15. sudo nano .ssh/authorized_keys
14. Change the permissions
        
        16. sudo chmod 700 /home/grader/.ssh
        17. sudo chmod 644 /home/grader/.ssh/authorized_keys
15. Change the owner from root to grader
        
        18. sudo chown -R grader:grader /home/grader/.ssh
16. Restart the ssh service
        
        19. sudo service ssh restart
17. Disconnect from **Amazon Lightsail Server**
        
        20. ~.
18. Log into the server as grader
        
        21. ssh -i ~/.ssh/udacity_key.rsa grader@13.233.31.36
19. We now need to enforce the key-based authentication:
        
        22. sudo nano /etc/ssh/sshd_config
Find the **PasswordAuthentication** line and change text after to **no**. After this, restart ssh again:
        
        23. sudo service ssh restart
20. We now need to change the ssh port from 22 to 2200, as required by Udacity:
        
        24. sudo nano /etc/ssh/sshd_config
Find the Port line and change 22 to 2200. Restart ssh:
        
        25. sudo service ssh restart
21. Disconnect the server
        
        26. ~.
and then log back through port 2200:
        
        27. ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.233.31.36
22. Disable ssh login for root user, as required by Udacity:
        
        28. sudo nano /etc/ssh/sshd_config
23. Find the **PermitRootLogin** line and edit to **no**. Restart ssh:
        
        29. sudo service ssh restart
24. Now we need to configure UFW to fulfill the requirement:
        
        30. sudo ufw allow 2200/tcp
        31. sudo ufw allow 80/tcp
        32. sudo ufw allow 123/udp
        33. sudo ufw enable
 And check the status of ufw by giving following command:
        
        34. sudo ufw status
 you'll get an output like this
        
        To                         Action      From
        --                         ------      ----
        2200/tcp                   ALLOW       Anywhere
        80/tcp                     ALLOW       Anywhere
        123/udp                    ALLOW       Anywhere
        2200/tcp (v6)              ALLOW       Anywhere (v6)
        80/tcp (v6)                ALLOW       Anywhere (v6)
        123/udp (v6)               ALLOW       Anywhere (v6)
# Deploy Catalog Application
We will use a virtual machine, apache2, and postgre to host our application. Before we use any thing, we need to log into the Amazon Terminal through your Terminal with grader.
1. Install required packages
        
        35. sudo apt-get install apache2
        36. sudo apt-get install libapache2-mod-wsgi python-dev
        37. sudo apt-get install git
2. Enable mod_wsgi by
        
        38. sudo a2enmod wsgi
and start the web server by 
        
        39. sudo service apache2 start
                      or
        40. sudo service apache2 restart
You should input the public IP address and you should see a page which shown as **Apache2 Ubuntu Default Page**. If you do not see the page, you have to check the error message and google a solution.
3. Set up the folder structure
        
        41. cd /var/www
        42. sudo mkdir catalog
        43. sudo chown -R grader:grader catalog
        44. cd catalog
4. Now we clone the project from **Github**
        
        45. git clone [your project link] catalog
Copy your link from your **Github Profile**, it'll be easy.
5. Create a **.wsgi** file
        
        46. sudo nano catalog.wsgi
and add the following thing into this file
        
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0, "/var/www/catalog/")

        from catalog import app as application
        application.secret_key = 'supersecretkey'
6. Rename the **Project.py** to **__init.py__**
7. Now we need to install and start the **virtual machine**
        
        47. sudo pip install virtualenv
        48. sudo virtualenv venv
        49. source venv/bin/activate
        50. sudo chmod -R 777 venv
You should see a (venv) appears before your username in the command line.
8. Now we need to install the Flask and other packages needed for this application
        
        51. sudo apt-get install python-pip (If needed)
        52. sudo pip install flask
        53. sudo pip install httplib2
        54. sudo pip install oauth2client
        55. sudo pip install sqlalchemy
        56. sudo pip install psycopg2 (sometimes it asks to install psycopg2-binary. You'll find this while executing your program)
        57. sudo pip install requests
        58. sudo pip install redirect
        59. sudo pip install psslib
9. Use the below command to change the **client_secrets.json** line to **/var/www/catalog/catalog/client_secrets.json**
        
        60. sudo nano __init__.py
and change the host to your Amazon Lightsail public IP address and port to 80 and change the last line of your program to this
        
        before app.run(host='0.0.0.0', port=5000)
        after  app.run()
10. Now we need to configure and enable the virtual host
        
        61. sudo nano /etc/apache2/sites-available/catalog.conf
paste the following code and save
         
        <VirtualHost *:80>
            ServerName [YOUR PUBLIC IP ADDRESS]
            ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME]
            ServerAdmin admin@13.233.31.36
            WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
            WSGIProcessGroup catalog
            WSGIScriptAlias / /var/www/catalog/catalog.wsgi
            <Directory /var/www/catalog/catalog/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/catalog/catalog/static
            <Directory /var/www/catalog/catalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
You can find the host name in this link: http://www.hcidata.info/host2ip.cgi
11. Now we need to setup the database
        
        62. sudo apt-get install libpq-dev python-dev
        63. sudo apt-get install postgresql postgresql-contrib
        64. sudo su - postgres
You should see the username changed again in command line, and type
        
        65. psql
to get into postgres command line
12. Now we create a user to create and set up the database. I name my database **catalog** with user **catalog**
        
        66. CREATE USER catalog WITH PASSWORD [your password];
        67. ALTER USER catalog CREATEDB;
        68. CREATE DATABASE catalog WITH OWNER catalog;
        69. (Connect to database) \c catalog
        70. REVOKE ALL ON SCHEMA public FROM public;
        71. GRANT ALL ON SCHEMA public TO catalog;
        72. Quit the postgrel command line: (\q) and then (exit)
13. use 
        
        73. sudo nano __init__.py
command to change all engine to **engine = create_engine('postgresql://catalog:[your password]@localhost/catalog**
change you engine in **database_setup.py** also.
14. Run the **database_setup.py** and **__init__.py** by using
        
        74. python database_setup.py
        75. python __init__.py
15. Restart the **Apache Server** by using below command
        
        76. sudo service apache2 restart
and enter your public IP address or host name into the browser. Your application should be online now!

# Reference
My sincere thanks to the people who poster their step-by-step on their Github:
https://github.com/KSaiAkhil17h75a0502/linux_server_configuration/blob/master/README.md
