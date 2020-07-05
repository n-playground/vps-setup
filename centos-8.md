- Access your server using VNC or SSH

#### Change root password

- `sudo passwd root`


#### Install nginx webserver

- `sudo dnf install nginx`
- `sudo systemctl enable nginx`
- `sudo systemctl start nginx`

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
- run `ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/` <- this will copy `default` file to `/etc/nginx/sites-enabled`
- run `systemctl restart nginx`

if there's an error, you can check the log via: `tail -20 /var/log/nginx/error.log`


#### Install nano editor

- `yum install nano`


#### Connect a domain to VPS

- Add new DNS record
```
| Type  | Host          | Value                     |
| A     | kamui.cyou    | -your server ip address-  |
| A     | www           | -your server ip adress-   |
```

- run `dig A +short www.kamui.cyou`

if dig command is not found, install it with `yum install bind-utils`


#### Setup FTP Server

- Install vsftpd with `sudo dnf install vsftpd`
- `sudo systemctl enable vsftpd`
- check the status with `sudo systemctl statuc vsftpd`

- Add new user with `sudo adduser dev`
- `sudo passwd dev`

- Limit user's folder access:
- add this line of code `chroot_local_user=YES` in the end of `/etc/vsftpd/vsftpd.conf`
- then restart the service `systemctl restart vsftpd`
