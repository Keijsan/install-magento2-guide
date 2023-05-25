
<h1 align="center">How to install magento 2.4.6 on LEMP with Ubuntu OS</h1>

### 1. Install Php & composer
> Install php
Make sure that installed packages are updated to the latest version use command:

```
sudo apt update && sudo apt upgrade
```

Add repository with command:

```
sudo apt install software-properties-common && sudo add-apt-repository ppa:ondrej/php -y
```

```
sudo apt update
```

> Install php and its modules with command:

```
sudo apt install php8.2
```

```
sudo apt install php8.2-{bcmath,fpm,xml,mysql,zip,intl,ldap,gd,cli,bz2,curl,mbstring,pgsql,opcache,soap,cgi,xdebug}
```

Change some value to match magento settings

```
sudo nano /etc/php/8.1/fpm/php.ini
```

Edit: 
```
memory_limit = 2G
max_execution_time = 1800
zlib.output_compression = On
```

```
sudo nano /etc/php/8.1/cli/php.ini
```
and same edit as above

> Install composer
```
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
```
```
sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

And use this command to check composer version

```
composer --version
```

### 2. Install Mysql
Make sure that installed packages are updated to the latest version use command:

```
sudo apt update && sudo apt upgrade
```

> Install mysql

```
sudo apt install mysql-server
```

```
sudo systemctl start mysql.service
```

> Configure mysql

```
sudo mysql
```

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '12345678';
```

```
exit
```

```
mysql -u root -p
```
(Enter password is 12345678) => success

Create database for magento246

```
CREATE DATABASE magento246;
```

### 3. Install Elasticsearch 7
> Install and configure Elasticsearch

```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
```

```
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

```
sudo apt update
```

```
sudo apt install elasticsearch
```

```
sudo nano /etc/elasticsearch/elasticsearch.yml
``` 
and edit `network.host: localhost`

> Run elasticsearch

```
sudo systemctl start elasticsearch
```

```
sudo systemctl enable elasticsearch
```

You can test if the installation was successful with the command:

```
curl -X GET 'http://localhost:9200'
```

### 4. Install nginx

You run the command:

```
sudo apt -y install nginx
```

We will configure nginx in step 5 when installing magento

### 5. Install Magento

> Install magento via composer

```
composer create-project --repository=https://repo.magento.com/ magento/project-community-edition <install-directory-name>
```

Here is the latest magento ver installation command if you want to install lower vers, as follows:

```
composer create-project --repository=https://repo.magento.com/ magento/project-community-edition=2.4.4 <home/keij/www/magento246>
```

> Grant permission to magento folder

Go to the magento root folder just downloaded. For example, here we have downloaded magento at home/keij/www/magento246 so:

```
cd ~
```

```
cd www/magento246
```

```
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
```

```
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
```

```
chown -R :www-data .
```

```
chmod u+x bin/magento
```

> Configure nginx for magento site

Create a virtual host for a magento site with command `sudo nano /etc/nginx/sites-available/magento246` <br>
Add this code to the configuration file just created:
```
upstream fastcgi_backend {
  server  unix:/run/php/php8.1-fpm.sock;
}

server {

  listen 80;
  server_name magento246.local;
  set $MAGE_ROOT /home/keij/www/magento246;
  include /home/keij/www/magento246/nginx.conf.sample;
}
```
* Remember to correct php-fpm according to the version you are using
* Edit `server_name` to the site name you want
* Edit the `$MAGE_ROOT` set and `include` is the path to install my magento root

And save and exit the editor<br>
Activate the newly created virtual host by creating a symlink to it in the /etc/nginx/sites-enabled directory:

```
sudo ln -s /etc/nginx/sites-available/magento245 /etc/nginx/sites-enabled
```

```
sudo nginx -t
```
to check if it was successful

If successful, restart nginx with the command `sudo systemctl restart nginx` Add a new host just as `server_name` created above

```
sudo nano /etc/hosts
```

And edit this look like that:
```
127.0.0.1       localhost
127.0.0.1       magento246
```

> Install magento with servers, databases, elasticsearch configured above:

Go to the magento installation folder and user command:

```
cd ~
```

```
cd home/keij/www/magento246
```

```
php bin/magento setup:install \
--base-url=http://magento246.local \
--db-host=localhost \
--db-name=magento246 \
--db-user=root \
--db-password=12345678 \
--backend-frontname=admin \
--admin-firstname=admin \
--admin-lastname=admin \
--admin-email=admin@admin.com \
--admin-user=admin \
--admin-password=admin123 \
--language=en_US \
--currency=USD \
--timezone=America/Chicago \
--use-rewrites=1 \
--search-engine=elasticsearch7 \
--elasticsearch-host=localhost \
--elasticsearch-port=9200
```
Due to installation via composer, there is no sampledata. Run the following command:

```
bin/magento sampledata:deploy
```

```
bin/magento setup:upgrade && bin/magento cache:flush
```

You can go to `http://magento246.local` to check your site magento

### 6. If you have a problem with decentralization, you can fix it as follows:

`my-user` is your user. Mine is `keij`
Change the owner of the magento root installation directory to `my-user` with the command:

```
sudo chown -R /home/keij/www/m245 my-user
```

Change user run with nginx:

```
sudo nano /etc/nginx/nginx.conf
```
(Edit `user` to `keij`)

Add your user to www-data group

```
sudo adduser my-user www-data
```

And then restart some services like nginx, php

### 7. Conclusion

Above is the whole process of how to install magento 2.4.6 with LEMP on Ubuntu operating system. If you have any problems you can contact me via email: <span style = "color: green">*kakuzukyo2000@gmail.com*</span>

 
