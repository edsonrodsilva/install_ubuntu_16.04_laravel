# install_ubuntu_16.04_laravel

Installing Laravel 5.x on Ubuntu 16.04 and Apache2

This post will document how I installed Laravel 5.2, on Apache2 on an Ubuntu 16.04 server. Will will also install MySQL, as we will need a database, and PHP which is required by Laravel. This will be the starting point of most of my posts, so if you’re following along from scratch…this is “scratch!”

First thing you need, of course, is the Ubuntu 16.04 server, with an SSH connection. Follow these excellent instructions and get yourself sorted out with one. Make sure you also give the server a static IP address (step 8., in the linked instructions). Come back when you’re done.

…

Welcome back! Lets get started.

Connect to your server now, using SSH, on your terminal.

1. Install Apache2 Web Server

The Apache server will run on Ubuntu and allow us to host websites. We will need to use “sudo” so that we have the correct permission to install it. Enter your user’s password when it asks for it. Run the following commands:

sudo apt-get update
sudo apt-get install apache2
If you enabled the UFW firewall, then you must add a rule to allow traffic to the Apache2 server by running the following command:

sudo ufw allow in "Apache Full"
Now use your web browser and test the Apache installation. Do this by going to the server’s IP address. In my case, this was 192.168.1.147. Insert your server IP address, of course, and not mine! 😛

url

If all is well, your browser will load the default “It Works!” page. Well done!

apache_hello

While we’re here, let’s enable mod_rewrite that Laravel will need later. Run the following commands in your terminal:

sudo a2enmod rewrite
sudo service apache2 restart
Upwards and onward!

2. Install MySQL server

Although Laravel 5.2 is supports many different types of database, I will be using MySQL for all my posts. If you are going to use a different type of database, then feel free the skip this section.

Go back to your terminal connection and run the following command:

sudo apt-get install mysql-server
When you are asked to set a MySQL root password, please do so. Don’t leave it blank.

3. Install PHP

Laravel 5.2 requires the following packages installed:

PHP >= 5.5.9
OpenSSL PHP Extension
PDO PHP Extension
Mbstring PHP Extension
Tokenizer PHP Extension
Run the following command to make it all happen:

$ sudo apt-get install php7.0 libapache2-mod-php7.0 php7.0-mbstring php7.0-zip php7.0-xml php7.0-mysql
We also included php7.0-zip  as this will be required by Composer, and
php7.0-xml which is required by PHPUnit (required to create a new Laravel project).

4. Install Composer

Laravel uses Composer to handle all its dependencies, so we need to install this next, by running the following command:

sudo curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
Note that this is for a global install. If you want to install it locally, just follow these instructions.

5. Install Laravel

We will use Composer to get Laravel, then add it to the PATH, so we can use Laravel commands anywhere. Run this command now:

composer global require "laravel/installer"
And add it to the PATH:

echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> ~/.bashrc
If for some reason your installation doesn’t have ~/.bashrc, you can use ~/.bash_profile  instead.

Restart apache now with:

$ sudo service apache2 restart
6. Create a Laravel Project and configure

Now in order to create a new Laravel project, just use the laravel new command, followed by the directory name. For example, laravel new monkey will create a new Laravel project, in the directory monkey . Let’s create a new project now at /var/www/html/ .

$ cd /var/www/html/
$ laravel new project
If you are hit with a permissions problem, make sure that you have correct permission to use the directory ( in this case html ).

We now have to make sure that a couple of directories, /storage  and /bootstrap/cache  are accessible by Apache2. We do this by running the following command:

sudo chgrp -R www-data /var/www/html/project/storage
sudo chgrp -R www-data /var/www/html/project/bootstrap/cache
Make sure you change “project” to whatever you named your own project.

7. Apache Virtual Host

To get Apache to serve up the correct page to your browser, we need to create a Virtual Host for our Laravel project. First thing to do is to go to the depth of Apache and copy the default configuration file, like so:

cd /etc/apache2/sites-available/
sudo cp 000-default.conf project.conf
Use your favorite text-editor, I will be using VIM, and open up the copy you just made:

sudo vim project.conf
Now edit your copy of project.conf as below. If you are following along with VIM, then you need to press “i” to start editing the file. Note that I have removed all the comments just to make it clearer:

<VirtualHost *:80>

        ServerName local.project.com
        ServerAdmin admin@project.com
        DocumentRoot /var/www/html/project/public

        <Directory "/var/www/html/project">
                AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
Again, if you are using VIM, click “esc” (the escape key) to come out of insert mode, then click :wq (colon w and q) to save the changes to you have made and quit VIM.

A quick note regarding some of the lines above. ServerName  is what you will type into the browser to reach your Laravel project’s first page. If you are not using a domain for this, then you can put whatever you want as the ServerName. If you do have a domain, then use that instead. DocumentRoot  and Directory are set with the structure that I have been using in this post, if yours differ, then please adjust as appropriate.

Now enable this new virtual host, disable the default one and reload Apache2:

sudo a2ensite project.conf
sudo a2dissite 000-default.conf
sudo service apache2 reload
For those who are not using a registered domain name, a final step is required. We need to change our computer’s host file so that entering http://local.project.com will tell our browser to go to our server, and not look for project.com on the internet.

On my Windows 10 machine, the hosts file is at C:\Windows\System32\drivers\etc\hosts . Once there, we’re going to add our server’s IP address (set when we were installing Ubuntu at the very beginning of this post) and the value we set for ServerName in the virtual host conf file (earlier in this step) to it:

192.168.1.147 local.project.com
The hosts file tends to have some crafty security added to it, so make sure you running your text-editor (notepad etc.) as administrator to edit it.

Now go back to your browser, enter the address you gave your project, and view the default index page in all its glory!

2016-08-03 12_59_22-Laravel
