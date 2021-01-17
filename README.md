# flask_w_nginx

## Step 1 - Install Nginx
> [source](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)
```bash
$ sudo apt update
$ sudo apt install nginx
```

## Step 2 - Adjusting the Firewall

```bash
$ sudo ufw apt list
Available applications:
  CUPS
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
```

Enable HTTP this by typing:
```bash
$ sudo ufw allow 'Nginx HTTP'
Rules updated
Rules updated (v6)
```

> Sometimes nginx ix still inactive

```bash
$ sudo ufw status
Status: inactive
```

> Use this to enable it:

```
$ sudo ufw enable
Firewall is active and enabled on system startup
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
Nginx HTTP                 ALLOW       Anywhere                  
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```



