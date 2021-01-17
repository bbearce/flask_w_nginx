# Flask with Nginx

## Part 1 - Nginx
### Step 1 - Install Nginx
> [source](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)
```bash
$ sudo apt update
$ sudo apt install nginx
```

### Step 2 - Adjusting the Firewall

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

### Step 3 - Checking Your Web server

> It should already be up and running

```bash
$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2021-01-17 11:30:46 EST; 13min ago
       Docs: man:nginx(8)
   Main PID: 334131 (nginx)
      Tasks: 9 (limit: 28605)
     Memory: 9.1M
     CGroup: /system.slice/nginx.service
             ├─334131 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─334132 nginx: worker process
             ├─334133 nginx: worker process
             ├─334134 nginx: worker process
             ├─334135 nginx: worker process
             ├─334136 nginx: worker process
             ├─334137 nginx: worker process
             ├─334138 nginx: worker process
             └─334139 nginx: worker process

Jan 17 11:30:46 pop-os systemd[1]: Starting A high performance web server and a reverse proxy>
Jan 17 11:30:46 pop-os systemd[1]: Started A high performance web server and a reverse proxy >
```

As you can see above, the service appears to have started successfully. However, the best way to test this is to actually request a page from Nginx.
![nginx_homepage](./nginx_homepage.jpg)

> I used ```0.0.0.0``` but we can use our server IP too

Try typing this at your server’s command prompt:

```bash
$ ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
> that didn't work for me so I delete the ```eth0``` from the command

```bash
$ ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
Device "eth0" does not exist.
$ ip addr show | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
127.0.0.1
::1
192.168.86.37
fe80::928c:d8ac:88cd:4a51
172.17.0.1
192.168.64.1
10.251.224.224
fe80::c757:385e:ab7b:53d1
fe80::fe71:628e:c254:97bb
```

> These worked for me: ```localhost```, ```127.0.0.0```, ```0.0.0.0```, ```192.168.86.37```, ```172.17.0.1```, ```192.168.64.1``` and ```10.251.224.224```.

An alternative is typing this, which should give you your public IP address as seen from another location on the internet:
```bash
$ curl -4 icanhazip.com
```
> This didn't work as I believe our internet router doesn't allow it. I think an azure VM would work.

### Step 4 – Managing the Nginx Process

To stop your web server, type:

```bash
$ sudo systemctl stop nginx
```
 
To start the web server when it is stopped, type:

```bash
$ sudo systemctl start nginx
```
 
To stop and then start the service again, type:

```bash
$ sudo systemctl restart nginx
```
 
If you are simply making configuration changes, Nginx can often reload without dropping connections. To do this, type:

```bash
$ sudo systemctl reload nginx
```
 
By default, Nginx is configured to start automatically when the server boots. If this is not what you want, you can disable this behavior by typing:

```bash
$ sudo systemctl disable nginx
```
 
To re-enable the service to start up at boot, you can type:

```bash
$ sudo systemctl enable nginx
```

### Step 5 – Setting Up Server Blocks (Recommended)

Nginx on Ubuntu 18.04 has one server block enabled by default that is configured to serve documents out of a directory at ```/var/www/html```. Instead of modifying ```/var/www/html```, let’s create a directory structure within ```/var/www``` for our **example.com** site, leaving ```/var/www/html``` in place as the default directory to be served if a client request doesn’t match any other sites.

Create the directory for example.com as follows, using the -p flag to create any necessary parent directories:

```bash
$ sudo mkdir -p /var/www/example.com/html
 ```
Next, assign ownership of the directory with the $USER environment variable:
```bash
$ sudo chown -R $USER:$USER /var/www/example.com/html
```

The permissions of your web roots should be correct if you haven’t modified your umask value, but you can make sure by typing:

```bash
$ sudo chmod -R 755 /var/www/example.com
```
 
Next, create a sample ```index.html``` page using nano or your favorite editor:
```bash
$ vi /var/www/example.com/html/index.html
```

In order for Nginx to serve this content, it’s necessary to create a server block with the correct directives. Instead of modifying the default configuration file directly, let’s make a new one at ```/etc/nginx/sites-available/example.com```:

```bash
$ sudo vi /etc/nginx/sites-available/example.com
```

```bash
server {
        listen 81;
        listen [::]:81;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Next, let’s enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:

```bash
$ sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```

Two server blocks are now enabled and configured to respond to requests based on their listen and server_name directives:

* example.com: Will respond to requests for example.com and www.example.com.  
* default: Will respond to any requests on port 80 that do not match the other two blocks.

To avoid a possible hash bucket memory problem that can arise from adding additional server names, it is necessary to adjust a single value in the ```/etc/nginx/nginx.conf``` file. Open the file:

```bash
$ sudo vi /etc/nginx/nginx.conf
```

Find the ```server_names_hash_bucket_size``` directive and remove the # symbol to uncomment the line:
```bash
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
```

Next, test to make sure that there are no syntax errors in any of your Nginx files:

```bash
$ sudo nginx -t
```
 
If there aren’t any problems, restart Nginx to enable your changes:

```bash
$ sudo systemctl restart nginx
```

Now if you navigate to ```http://example.com``` you should see this:
![example.com](./nginx_example.com.jpg)

> NOTE 1: this didn't work at first and brought me to someone elses ```example.com```. What you need to do is edit ```/etc/hosts``` and map ```example.com``` to ```localhost``` or ```0.0.0.0```. Once I did that it worked. 

> NOTE 2: even that didn't work first time because example.com and ngnix's default directory and site were both on port 80 and nginx's default was chosen over ```example.com```. This is why in the examples above I change ```example.com``` to serve on port ```81```.

> [Great Video Tutorial](https://www.youtube.com/watch?v=1ndlRiaYiWQ)


## Part 2 - Flask Integration