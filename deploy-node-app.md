### # Install pm2

If you close your SSH client window, your website will stop working. We need some tool that will keep our server “alive”.
```
$ npm install pm2 -g
$ pm2 start server.js
```

then check if your website is running

#### Make PM2 to start at boot

```
pm2 startup
# this will generate another command that you need to run

# We also need to save what processes 
# should get started with pm2
pm2 save

# reboot VPS and check if your website is up
sudo reboot
```
