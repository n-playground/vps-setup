- Access your server using VNC or SSH

#### Install nginx webserver

- `sudo dnf install nginx`
- `sudo systemctl enable nginx`
- `sudo systemctl start nginx`


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
