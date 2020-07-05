Access your server using VNC or SSH

### # Change root password
```
$ sudo passwd root
```

<hr>

### # Install net-tools
```
$ yum install net-tools
$ netstat -tulpen | grep nginx
```

<hr>

### # Install nginx webserver

```
$ sudo dnf install nginx
$ sudo systemctl enable nginx
$ sudo systemctl start nginx
```

- create folder named `sites-enabled` and `sites-available` in `/etc/nginx/`
- then edit `/etc/nginx/nginx.conf` file and add `include /etc/nginx/sites-enabled/*;` inside `http` block

- create a file named `default` in `/etc/nginx/sites-available`
- and add these lines of code:
```
server {
   listen 80 default_server;
   listen [::]:80 default_server;
   
   root /var/www/html; <- your public www path
   
   index index.php index.html index.htm;
   
   server_name localhost;
   
   location / {
    try_files $uri $uri/ =404;
   }
}
```
- command `server` block in `/etc/nginx/nginx.conf`
- run `$ sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/` <- this will copy `default` file to `/etc/nginx/sites-enabled`
- run `$ sudo systemctl restart nginx`

if there's an error, you can check the log via: `$ sudo tail -20 /var/log/nginx/error.log`

<hr>

### # Install nano editor

- `$ sudo yum install nano`

<hr>

### # Connect a domain to VPS

- Add new DNS record
```
| Type  | Host          | Value                     |
| A     | kamui.cyou    | -your server ip address-  |
| A     | www           | -your server ip address-  |
```

- run `$ dig A +short www.kamui.cyou`

if dig command is not found, install it with `$ yum install bind-utils`

<hr>

### # Setup FTP Server

- Install vsftpd with `$ sudo dnf install vsftpd`
- `$ sudo systemctl enable vsftpd`
- check the status with `$ sudo systemctl statuc vsftpd`

Add new user with 
```
$ sudo adduser dev
```
- then change the password `$ sudo passwd dev`

Limit user's folder access:
- add this line of code `chroot_local_user=YES` in the end of `/etc/vsftpd/vsftpd.conf`
- then restart the service `$ sudo systemctl restart vsftpd`

<hr>

### # Network monitoring

```
$ sudo yum install iptraf
$ sudo iptraf-ng
```

<hr>

### # Install MariaDB

```
$ sudo yum install mariadb-server mariadb
$ sudo systemctl start mariadb
```

Reconfigure MariaDB
```
$ sudo mysql_secure_installation
```
- follow the steps

Creating a new user 
```
$ mysql -u root -p` and enter the password
MariaDB [(none)]> CREATE USER 'username'@'localhsot' IDENTIFIED BY 'the_password';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' WITH GRANT OPTION;
```

to access outside localhost:
```
MariaDB [(none)]> CREATE USER 'username'@'%' IDENTIFIED BY 'the_password';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'username'@'%' WITH GRANT OPTION;

MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit
```
- Try connect with created user `$ mysql -u admin -p` and enter the password

<hr>

### # Change MariaDB port
- Edit file `/etc/my.cnf.d/mariadb-server.cnf` and add this line of code under `[mysqld]` block
```
port = 3307
```
then restart the service `$ sudo systemctl restart mariadb`
and you can check the port changed with `$ netstat -tlpn | grep mysql`

<hr>

### # Install phpMyAdmin

```
$ sudo yum install epel-release
$ rpm -iUvh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ sudo yum update
$ sudo yum install phpMyAdmin
```

if installation failed with this error message `unable to find a match: phpMyAdmin`, run this command:
```
$ yum --enablerepo=remi install phpMyAdmin
```

then
```
$ sudo ln -s /usr/share/phpMyAdmin /var/www/html <- your public www path
$ sudo systemctl restart php-fpm
```

and now you can access phpmyadmin in the browser via `http://YOUR_SERVER_IP/phpMyAdmin` (camel-case sensitive, if you folder name `phpmyadmin` then open `http://YOUR_SERVER_IP/phpmyadmin`)

<hr>

### # Set Up an Nginx Authentication Gateway/Securing your phpMyAdmin page

This will show an authentication prompt that a user would be required to pass before ever seeing the phpMyAdmin login screen.
First thing first, you need to create an encrypted password, the encryption is using `crypt()`, you can use an crypt generator.
Then type `$ openssl passwd`, and enter your un-encrypted password.

Then, create an authentication file `$ sudo nano /etc/nginx/pma_pass`, and type your credential inside that file, for example:
```
username:the-encrypted-password
dev:dev-password
```

Then add new location block inside `default` Nginx configuration.
```
$ sudo nano /etc/nginx/sites-available/default
```

and add these line of code
```
location /phpmyadmin {
   auth_basic "Promp Alert";
   auth_basic_user_file /etc/nginx/pma_pass;
}
```
```
$ sudo rm /etc/nginx/sites-enabled/default && ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
$ sudo systemctl restart nginx
```
and try opening your phpMyAdmin via browser, it will show an authentication prompt before entering the actual phpMyAdmin page.

<hr>

### # Install PHP

```
$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

Confirm the presence of the EPEL repository
```
$ rpm -qa | grep epel
```

```
$ dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

Verify the existence of the Remi repository
```
$ rpm -qa | grep remi
```
```
$ yum module list php
$ yum module enable php:remi-7.4
$ yum install php php-cli php-common
$ systemctl enable --now php-fpm
```

Configuring PHP to work with Nginx
```
$ sudo nano /etc/php-fpm.d/www.conf
```

Find and change this line
```
user = nginx
group = nginx
```

Make sure the `/var/lib/php` directory has the correct ownership
```
$ chown -R root:nginx /var/lib/php
```

then restart the PHP FPM service `$ sudo systemctl restart php-fpm`

Now edit the Nginx virtual host directive, and add the following location block so that Nginx can process PHP files
```
server {

    # . . . other code

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

then, restart the Nginx service `$ sudo systemctl restart nginx`'

<hr>

### # Install Composer PHP Package Manager

```
$ sudo yum install php-cli php-zip wget unzip
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
