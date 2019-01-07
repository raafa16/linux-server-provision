# Linux server provisioning script to deploy a rails 5 app.  

# 1) !/bin/bash

set -e

## 2) install locale

## 3) updating system

apt-get update

apt-get upgrade -y

apt-get dist-upgrade -y

apt-get autoremove -y

apt-get -y install build-essential zlib1g-dev libssl-dev libreadline-dev libcurl4-openssl-dev

apt-get install -y git-core

## 4) install rbenv

git clone git://github.com/rbenv/rbenv.git /usr/local/rbenv
echo 'export RBENV_ROOT=/usr/local/rbenv' >> /etc/profile.d/rbenv.sh
echo 'export PATH="$RBENV_ROOT/bin:$PATH"' >> /etc/profile.d/rbenv.sh
echo 'eval "$(rbenv init -)"' >> /etc/profile.d/rbenv.sh
chmod +x /etc/profile.d/rbenv.sh
mkdir -p $RBENV_ROOT/plugins
git clone https://github.com/rbenv/ruby-build.git $RBENV_ROOT/plugins/ruby-build
source /etc/profile.d/rbenv.sh
rbenv install 2.5.3
rbenv global 2.5.3
gem install bundler --no-ri --no-rdoc
rbenv rehash

## 5) install nginx

add-apt-repository ppa:nginx/stable --yes
apt-get -y update
apt-get -y install nginx

## 6) install nodejs

curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs

## 7) create deployer user
useradd deployer -m -s /bin/bash
usermod -a -G sudo deployer
mkdir /home/deployer/.ssh
vim /home/deployer/.ssh/authorized_keys # add your public key to authorized_keys
chown -R deployer:deployer /home/deployer/.ssh
chmod 400 /home/deployer/.ssh/authorized_keys
echo 'gem: --no-ri --no-rdoc' > /home/deployer/.gemrc

## 8) setup ufw
ufw --force enable
ufw allow ssh
ufw allow http
ufw allow https

## 9) sudonopassword
vim /etc/sudoers.d/deployer
deployer ALL=NOPASSWD:/bin/rm /etc/nginx/sites-enabled/default
deployer ALL=NOPASSWD:/bin/ln -nfs /home/deployer/* /etc/nginx/sites-enabled/*
deployer ALL=NOPASSWD:/bin/ln -nfs /home/deployer/* /etc/init.d/*
deployer ALL=NOPASSWD:/bin/ln -nfs /home/deployer/* /etc/logrotate.d/*
deployer ALL=NOPASSWD:/etc/init.d/nginx
deployer ALL=NOPASSWD:/etc/init.d/puma_{ app_name }_{ env } # change "app_name" to your app's name and "env" to environment such as staging/production

## 10) file write finished
chown root:root /etc/sudoers.d/deployer
chmod 0440 /etc/sudoers.d/deployer


## 11) installing postgres
sudo apt-get install postgresql postgresql-contrib libpq-dev

## 12) enter postgres console to create user and database
sudo su - postgres
createuser --pwprompt deployer
createdb -O deployer app_name_production  # change "app_name" to your app's name which we'll also use later on
exit


