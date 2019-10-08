#setting for text editor
# _*_ mode: ruby _*_

Vagrant.configure("2") do |config|

  ################################
  #web app server
  ################################
  config.vm.define "app" do |app|

    #OS being pulled from Vagrant could
    app.vm.box = "ubuntu/bionic64"


    # specification of the virtualization software :
    app.vm.provider "virtualbox" do |v|
      # Virtual machine name to virtualization software
      v.name = "LAPside"
      # RAM-memory
      v.memory = 1024
      # CPU core/thread count for virtual machine
      v.cpus = 1
    end

    #networking configurations
    app.vm.network "forwarded_port", guest: 8080, host:80
    app.vm.network "private_network", ip: "192.168.26.45"

    bootstrap = <<SCRIPT
#! /usr/bin/env bash

#setting up color for highlight in provision BUG it does not work...
#blue=`tput setaf 4`
#reset=`tput sgr0`
# trying with boxed text instead by substituting "${blue}"->"\033[7m" and "${reset}"->"\033[0m"


export DEBIAN_FRONTEND=noninteractive
echo "\033[7mProvisioning Apache-PHP-WordPress virtual machine...\033[0m"

echo "\033[7mUpdating and upgrading Ubuntu\033[0m"
sudo apt-get update && sudo apt-get upgrade

echo "\033[7mInstalling apache\033[0m"
sudo apt-get install -y apache2
echo "\033[7mSetting up /var/www synced folder\033[0m"
if ! [ -L /var/www ]; then
    rm -rf /var/www
    ln -fs /vagrant /var/www
fi

echo "\033[7mInstalling PHP and dependencies for WordPress & MySQL\033[0m"
sudo apt-get install -y php php-curl php-gd php-mbstring mcrypt php-xml php-xmlrpc php-mysql #it cannot find "php-mcrypt", substituted with mcrypt

echo "\033[7mSetting the 'ServerName' directive globally\033[0m"
sudo echo "ServerName localhost" >> /etc/apache2/apache2.conf

echo "\033[7mRestarting apache2 web server\033[0m"
sudo service apache2 restart

echo "\033[7mSetting up simple php file\033[0m"
sudo mv /vagrant/index.php /var/www/html/index.php

echo "\033[7mDownloading WordPress\033[0m"
cd /var/www/
sudo wget -c http://wordpress.org/latest.tar.gz

echo "\033[7mInstalling WordPress\033[0m"
tar -xzvf latest.tar.gz -C /var/www/
rm  latest.tar.gz

echo "\033[7mCreate wordpress configuration file from sample\033[0m"
sudo cp /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php

echo "\033[7mWordpress database variables insertion\033[0m"
sed -i "s/database_name_here/wordpress/" /var/www/wordpress/wp-config.php
sed -i "s/username_here/dbuser/" /var/www/wordpress/wp-config.php
sed -i "s/password_here/dbpass/" /var/www/wordpress/wp-config.php
sed -i "s/localhost/192.168.26.44/" wordpress/wp-config.php

#echo "\033[7mEnabling WordPress site\033[0m"
#sed -i "s/\/var\/www\/html/\/var\/www\/wordpress/" /etc/apache2/sites-enabled/000-default.conf
SCRIPT

    #config.vm.provision :shell, path: "bootstrap.sh"
    app.vm.provision :shell, :inline => bootstrap
  end

  #######################################
  #mysql database server
  #######################################
  config.vm.define "db" do |db|

    #OS being pulled from Vagrant could
    db.vm.box = "ubuntu/bionic64"

    # specification of the virtualization software :
    db.vm.provider "virtualbox" do |vb|
      # Virtual machine name to virtualization software
      vb.name = "DBside"
      # RAM-memory
      vb.memory = 1024
      # CPU core/thread count for virtual machine
      vb.cpus = 1
    end

    dbbootstrap = <<SCRIPT
#! /usr/bin/env bash

#setting up color for highlight in provision
blue=`tput setaf 4`
reset=`tput sgr0`

export DEBIAN_FRONTEND=noninteractive
echo "\033[7mProvisioning database virtual machine...\033[0m"

echo "\033[7mUpdating and upgradin Ubuntu...\033[0m"
sudo apt-get update && sudo apt-get upgrade

echo "\033[7mInstalling debconf tool...\033[0m"
sudo apt-get install -y debconf-utils

echo "\033[7mPreparing mysql-server root password\033[0m"
sudo debconf-set-selections <<< "mysql-server mysql-server/root_password password root"
sudo debconf-set-selections <<< "mysql-server mysql-server/root_password_again password root"

echo "\033[7mInstalling mysql-server\033[0m"
sudo apt-get install -y mysql-server

#echo "\033[7mSecure installation of mysql\033[0m"
#sudo mysql_secure_installation #BUG: cause fatal failure, unnecessary!

echo "\033[7mOpen up mysql to listen from every external connection\033[0m"
sed -i "s/127.0.0.1/0.0.0.0/" /etc/mysql/mysql.conf.d/mysqld.cnf

echo "\033[7mCreating wordpress database and dbuser\033[0m"
sudo mysql -u root -proot <<EOF
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO 'dbuser'@'%' IDENTIFIED BY 'dbpass';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root';
FLUSH PRIVILEGES;
EOF

echo "\033[7mReinitialize mysql configuration\033[0m"
sudo systemctl restart mysql

SCRIPT

    #networking configurations
    db.vm.network "private_network", ip: "192.168.26.44"
    db.vm.network "forwarded_port", guest: 3306, host: 3333

    # config.vm.provision :shell, path: "db-bootstrap.sh"
    db.vm.provision :shell, :inline => dbbootstrap

  end





end
