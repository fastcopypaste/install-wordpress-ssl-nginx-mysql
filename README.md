## Install Wordpress SSL (certbot) Nginx Mysql in Ubuntu 18.04

### Prerequisite

```
sudo apt-get update
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
        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name yoursite.com;

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
