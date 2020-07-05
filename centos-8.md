Access your server using VNC or SSH

### Change root password

- `$ sudo passwd root`


### Install nginx webserver

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

if there's an error, you can check the log via: `tail -20 /var/log/nginx/error.log`


### Install nano editor

- `$ sudo yum install nano`


### Connect a domain to VPS

- Add new DNS record
```
| Type  | Host          | Value                     |
| A     | kamui.cyou    | -your server ip address-  |
| A     | www           | -your server ip adress-   |
```

- run `$ dig A +short www.kamui.cyou`

if dig command is not found, install it with `$ yum install bind-utils`


### Setup FTP Server

- Install vsftpd with `$ sudo dnf install vsftpd`
- `$ sudo systemctl enable vsftpd`
- check the status with `$ sudo systemctl statuc vsftpd`

Add new user with 
```
$ sudo adduser dev
```
- then change the password `sudo passwd dev`

Limit user's folder access:
- add this line of code `chroot_local_user=YES` in the end of `/etc/vsftpd/vsftpd.conf`
- then restart the service `$ sudo systemctl restart vsftpd`


### Network monitoring

```
$ sudo yum install iptraf
$ sudo iptraf-ng
```


### Install MariaDB

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
- Try connect with created user `mysql -u admin -p` and enter the password


### Install phpMyAdmin

```
$ sudo yum install epel-release
$ rpm -iUvh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ sudo yum update
$ sudo yum install phpmyadmin
```

### Install PHP

```
$ dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
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

then, restart the Nginx service `$ sudo systemctl restart nginx`
