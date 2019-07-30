## Install Wordpress SSL (certbot) Nginx Mysql in Ubuntu 18.04

### Prerequisite

```
sudo apt-get update
sudo apt-get install -y software-properties-common
```

### Install Nginx, MySQL and PHP

```
sudo apt-get install nginx
sudo apt install mysql-server
sudo apt install php-fpm php-mysql php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
sudo systemctl restart php7.2-fpm
```

### Config Mysql

```
sudo mysql_secure_installation
```

You should type `yes` for all options.

Now we make table for wordpress and its user

```
sudo mysql
```
The Mysql should be displayed, run these commands line by line:

```
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
EXIT;
```

### Config Nginx

```
sudo nano /etc/nginx/sites-available/yoursite.com
```

Replace `yoursite.com` with your domain name.

You can use any text editor instead of `nano`.

Add this code below and then save it:

```
server {
        listen 80;
        root /var/www/wordpress;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name yoursite.com www.yoursite.com;

	location = /favicon.ico { log_not_found off; access_log off; }
    	location = /robots.txt { log_not_found off; access_log off; allow all; }
    	location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    	}

        location / {
                #try_files $uri $uri/ =404;
		try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }
}
```

Again replace `yoursite.com` with your domain.

After that, run this command:

```
sudo ln -s /etc/nginx/sites-available/yoursite.com /etc/nginx/sites-enabled/
sudo unlink /etc/nginx/sites-enabled/default
```

If you want to restore the default config:

```
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
```

Test your config:

```
sudo nginx -t
```

Reload `nginx` to apply changes:

```
sudo systemctl reload nginx
```

### Install Certbot

Run following commands:

```
sudo add-apt-repository ppa:certbot/certbot
sudo apt install python-certbot-nginx
```

Everything should be ok, you need to make your own ssl for your domain name

```
sudo certbot --nginx -d yoursite.com -d www.yoursite.com
```

Please do not forget to replace `yoursite.com` to your domain name. You should see the congratulation page.

We need to make our certificate being renewed automatically by Certbot:

```
sudo certbot renew --dry-run
```

### Install Wordpress

Time to install Wordpress:

```
cd /tmp && curl -LO https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
sudo cp -a /tmp/wordpress/. /var/www/wordpress
sudo chown -R www-data:www-data /var/www/wordpress
```

Setup Wordpress config

```
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
Copy the return value to your clipboard.

After that, run it:

```
sudo nano /var/www/wordpress/wp-config.php
```

You should get the value of each key `AUTH_KEY, SECURE_AUTH_KEY, LOGGED_IN_KEY, NONCE_KEY, AUTH_SALT, SECURE_AUTH_SALT, LOGGED_IN_SALT, NONCE_SALT`, and now replace each value you get inside your `wp-config.php` file:

```
define('AUTH_KEY',         'VALUES COPIED FROM THE COMMAND LINE');
define('SECURE_AUTH_KEY',  'VALUES COPIED FROM THE COMMAND LINE');
define('LOGGED_IN_KEY',    'VALUES COPIED FROM THE COMMAND LINE');
define('NONCE_KEY',        'VALUES COPIED FROM THE COMMAND LINE');
define('AUTH_SALT',        'VALUES COPIED FROM THE COMMAND LINE');
define('SECURE_AUTH_SALT', 'VALUES COPIED FROM THE COMMAND LINE');
define('LOGGED_IN_SALT',   'VALUES COPIED FROM THE COMMAND LINE');
define('NONCE_SALT',       'VALUES COPIED FROM THE COMMAND LINE');
```

And we also need to change `DB_NAME, DB_USER, DB_PASSWORD` like this:

```

define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');
```

At the line below to the end:

```
define('FS_METHOD', 'direct');
```
