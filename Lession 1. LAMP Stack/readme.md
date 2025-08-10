## WEB STACK IMPLEMENTATION (LAMP STACK) ON AWS

### Introduction:

__The LAMP stack is a very well-known web development platform, it is open source and it is made up of four main components which are: Linux, Apache, MySQL, and PHP (sometimes Perl or Python). This documentation shows how to setup, configure and use the LAMP stack for your consumption.__

## Step 0: Getting Ready

_Here, we ensure that the environment (Linux OS) we need to run/serve our website is made ready. Side note, I use the ubuntu os on my PC_

__1.__ Spin up an EC2 instance of t3.micro with the Ubuntu 24.04 LTS Operating System (I used eu-north-1)

![How to Lunch an Instance](./images/ec2-instance-creation.png)

__2.__ Create an SSH Key pair when launching/creating the instance.

__3.__ While creating this EC2 instance, it is important to create a security group configured with the below inbound rules:

- Allow traffic on port 80 (HTTP) from any source IP on the internet
- Allow traffic on port 443 (HTTPS) from any source IP on the internet
- Allow traffic on port 22 (SSH) from any source IP _(This rule is allowed by default)_
 
 __4.__ Select the right VPC when creating the EC2 instance (in my case, I used the default VPC).

 __5.__ Download private key, and modify private key file permissions, so that only your user can access the ssh key.
 ```
 chmod 600 "your-ec2-key.pem"
 ```  
 If you try to ssh wuthout doing this you would get `Permission denied` error, that looks like this  
 ![Permission Denied](./images/permission-denied.png)

 __6.__ Use private key to access the EC2 instance from your terminal with the help of running this commmand: 
 
 ```
 ssh -i "your-ec2-key.pem" <username>@IPAddress
 ```
 in my case, __username=ubuntu__ and __public ip address=52.228.16.110 (this address changes at every shutdown and startup of an instance (Elastic IP))__

 ![Connect to your created instance](./images/connect-to-ec2.png)


 ## Step 1 - Install Apache and Configure the Firewall
**1.** __First, you have to update and upgrade the package manager's repository__

To achieve this, after connecting to the instance, run this command on your terminal.
```
sudo apt update
sudo apt upgrade -y
```
![Update package manager's repository](./images/apt-update.png)

__2.__ __Install Apache2__
```
sudo apt install apache2 -y
```
![Install Apache2](./images/apache-download.png)

__3.__ __Enable Apache2 and Verify that it is running__
```
sudo systemctl enable apache2
sudo systemctl status apache2
```
after runnnig the `systemctl status` command, if the output has the green indication saying `active (running)` _(as shown in the image below)_ , then apache2 is up and running

![Apache status after installation](./images/apache-start.png)


__4.__ __Verify that the server is running and can be accessed locally via ubuntu shell by running this command__
```
curl http://localhost:80
OR
curl http://127.0.0.1:80
```

__5.__ __Test if you can access the default page apache serves on server by trying to access your EC2 public IP via a browser__
```
http://35.179.168.126:80
```
![Apache default page served on you EC2 machine](./images/apache-default.png)
This indicates that the web server has been correctly installed and is now accessible through the firewall.

## Step 2 - Install MySQL
__1.__ __Install a Realational Database (RDB) to handle webapp__

MySQL would be installed for this case. This is a widely used relational database managemet system that works well within PHP environments.
```
sudo apt install mysql-server
```
![Install MySQL](./images/install-mysql.png) type y and press Enter, when prompted.

__2.__ __Enable mysql and Ensure that it is running by running these commands:__
```
sudo systemctl enable --now mysql
sudo systemctl status mysql
```

__3.__ __Log in to mysql console__
```
sudo mysql
```
Running this command will conect your shell to the MySQL server as the admin database user __root__ assumed by the useof __'sudo'__ when you were executing the __`sudo mysql`__ command.

__4.__ __Assign a password to the root user using the mysql_native_password as the default authetication method.__
The password assigned to the user for the purpose of this documentation is "password"
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
![Assign mysql root user password](./images/mysql-pass.png)

Exit the MySQL shell 
```
exit
```

__5.__ __Run the script to secure MySQL__

This script is used to remove some insecure default settings and locks down access to the database system.
```
sudo mysql_secure_installation
```
![Run interactive script to remove insecure settings](./images/mysql-secure-install.png)

__6.__ __Login to the MySQL console as a root user after you have changed the password.__
```
sudo mysql -p
```
A prompt asking you to insert a password (this is because of the -p flag) as shown in the image below:

![Test login using mysql root new credential](./images/mysql-login.png)



### Step 3 - Install PHP

__1.__ __Install php.__

To set up php on the server, we would need to install:
* php package
* php-mysql _(this is a PHP module that allows PHP to communicate with MySQL databases)_
* libapache2-mod-php _(this helps Apache to handle and understand PHP files)
to get this all set up in the machine, run:
```
sudo apat install php libapache2-mod-php php-mysql
```
![Set up PHP on the machine](./images/php-install.png)

Confirm the PHP version by running:
```
php -v
```
![Confirm PHP version](./images/verify-php.png)

To test this setup, we need to setup an Apache Virtual host to hold the website's files and folders.
_(Virtual Hosts makes it possible to have many/different websites located on one machine while abstracting this from the users.)_

## Step 4 - Create a virtual host 
__1.__ __First, create a document directory for the new website you are about to create near the default web dir _(/var/www/html).__
```
cd /var/www/html
sudo mkdir lamp_project
```

__Assign the ownership of the directory to the current user in the session__
```
sudo chown -R $USER:$USER /var/www/html/lamp_project
```
![Assign ownership of the new directory to current User](./images/assign-ownership.png)

__2.__ __Create a configuration file for the new website__
```
sudo nano /etc/apche2/sites-available/lamp_project.conf
```
Write the configurations shown below into the config file:

![Virtual Host Bare Bones Config](./images/apache-config.png)

__3.__ __Show the new file in sites-available__
```
sudo ls /etc/apache2/sites-available
```

![lamp_project](./images/apache-config-ls.png)

With this configuration setup here (lamp_project.conf), the web server (Apache in this case) will serve our webproject (lamp_project) utilizing /var/www/html/lamp_project 

__4.__ __Enable the new virtual host__
```
sudo a2ensite lamp_project.conf
```
__5.__ __Disable Apache's default website.__
If we do not disable this default website, Apache's default site will overwrite that in the virtual host, and hence, we won't get to be served the site sitting in the virtual host.
_To disable Apache's default website, run this command_
```
sudo a2dissite 000-default.conf
```

__6.__ __Ensure that the configuration in the configuration file does not contain syntax error__

You can achieve the above by running the command below.
```
sudo apache2ctl configtest
```

![Apache Config Enable](./images/apache-config-enable.png)

__7.__ __Reload apache for the changes to be applied__
```
sudo systemctl reload apache2
```

![Reload Apache](./images/apache-reload.png)


## Step 5 - Enable PHP on the website.

Currently (i.e: by default), the DirectoryIndex setting on the Apache will always tell Apache to take html files more seriously (higher precedence) than php files. (This setting is good such that, when a php file is being served, and one wants to take the site down for maitainance for a while, he/she could easily setup a customized `maintenance` html file an place it in server's site directory and Apache would serve that `maintenance` page, instead of the main `php` file. and when the site maintenance is done, the file could be deleted or removed to resume normal site functions.)

__1.__ __To modify the precedence Apache gives to different file types, modify the dir.conf file to look like this.__
```
sudo nano /etc/apache2/mods-enabled/dir.conf
```
```

  # Change this:
  DirectoryIndex index.html index.cgi index.ppl index.php index.xhtml index.htm

  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm

```

![Rearrange the file priority of Apache](./images/fix-file-order.png)

__2.__ __Reload Apache__

You need to reload Apache so that the changes you have made would apply.
```
sudo systemctl reload apache2
```
![Reload Apache](./images/apache-reload.png)

__3.__ __Utilize the php test script to ensure that you have configured Apache to handle and process requests for PHP files.__

Create a new file inside the custom web root folder (/var/www/html/lamp_project/)

```
nano /var/www/html/lamp_project/index.php
```

__Type in the words below into the `index.php` file__
```
<?php
phpinfo();
```
![Php web file content](./images/php-code.png)

__4.__ __Refresh the page and note down any change you observe__

![Php default page](./images/php-page.png)
The page served here provides you with information about the server from the perspective of PHP. This is very useful in terms of debugging and to make sure that all necessary settings are applied correctly.

When you are done checking the information about the server through this page, 
It is very important that you remove the file you have created (i.e: index.php) so as to protect the sensitive information about the Php enviroment and the server. This can be repeated when necessary if the information is needed.
```
sudo rm /var/www/html/lamp_project/index.php
```

__Conclusion__

If you are considering/evaluating what stack to utilize in deploying a website and make it accessible, the LAMP stack would be an excellent choice.



