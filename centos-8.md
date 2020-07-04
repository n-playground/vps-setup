- Access your server using VNC or SSH

#### Install nginx webserver

- `sudo dnf install nginx`
- `sudo systemctl enable nginx`
- `sudo systemctl start nginx`

- create folder named `sites-enabled` and `sites-available` in `/etc/nginx/`
- then edit `/etc/nginx/nginx.conf` file and add `include /etc/nginx/sites-enabled/*;` inside `http` block

- create a file named `default` in `/etc/nginx/`
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
- run `ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/`
- run `systemctl restart nginx`

- if there's an error, you can check the log via:
  `tail -20 /var/log/nginx/error.log`


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
