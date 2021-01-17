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

Enable HTTP by typing:
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
> that didn't work for me so I deleted the ```eth0``` from the command

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

> [Source](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04)

### Step 1 — Installing the Components from the Ubuntu Repositories
```bash
$ sudo apt update
$ sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
```

### Step 2 — Creating a Python Virtual Environment

```bash
mkdir ./myproject
cd ./myproject
```

Create a virtual environment to store your Flask project’s Python requirements by typing:
```bash
$ python3 -m venv myprojectenv
$ source myprojectenv/bin/activate
```

### Step 3 — Setting Up a Flask Application
First, let’s install wheel with the local instance of pip to ensure that our packages will install even if they are missing wheel archives:
```bash
pip install wheel
```

Next, let’s install Flask and uWSGI:
```bash
$ pip install uwsgi Flask
```

#### Creating a Sample App

While your application might be more complex, we’ll create our Flask app in a single file, called myproject.py:
```bash
$ vi myproject.py
```

If you followed the initial server setup guide, you should have a UFW firewall enabled. To test the application, you need to allow access to port 5000:

```bash
$ sudo ufw allow 5000
Rule added
Rule added (v6)
```

Now, you can test your Flask app by typing:
```bash
$ python myproject.py
 * Serving Flask app "myproject" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

#### Creating the WSGI Entry Point

Next, let’s create a file that will serve as the entry point for our application. This will tell our uWSGI server how to interact with it.

Let’s call the file ```wsgi.py```:

```bash
$ vi ./wsgi.py
```

In this file, let’s import the Flask instance from our application and then run it:

```bash
from myproject import app

if __name__ == "__main__":
    app.run()
```


### Step 4 — Configuring uWSGI

Your application is now written with an entry point established. We can now move on to configuring uWSGI.

#### Testing uWSGI Serving

Let’s test to make sure that uWSGI can serve our application.

We can do this by simply passing it the name of our entry point. This is constructed by the name of the module (minus the ```.py``` extension) plus the name of the callable within the application. In our case, this is ```wsgi:app```.

Let’s also specify the socket, so that it will be started on a publicly available interface, as well as the protocol, so that it will use HTTP instead of the ```uwsgi``` binary protocol. We’ll use the same port number, ```5000```, that we opened earlier:

```bash
$ uwsgi --socket 0.0.0.0:5000 --protocol=http -w wsgi:app

*** Starting uWSGI 2.0.19.1 (64bit) on [Sun Jan 17 15:40:48 2021] ***
compiled with version: 9.3.0 on 17 January 2021 19:57:08
os: Linux-5.4.0-7642-generic #46~1598628707~20.04~040157c-Ubuntu SMP Fri Aug 28 18:02:16 UTC 
nodename: pop-os
machine: x86_64
clock source: unix
pcre jit disabled
detected number of CPU cores: 8
current working directory: /home/bbearce/Documents/flask_w_nginx/myproject
detected binary path: /home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi
*** WARNING: you are running uWSGI without its master process manager ***
your processes number limit is 95353
your memory page size is 4096 bytes
detected max file descriptor number: 1024
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uwsgi socket 0 bound to TCP address 0.0.0.0:5000 fd 3
Python version: 3.8.5 (default, Jul 28 2020, 12:59:40)  [GCC 9.3.0]
*** Python threads support is disabled. You can enable it with --enable-threads ***
Python main interpreter initialized at 0x562e5fe38320
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 72920 bytes (71 KB) for 1 cores
*** Operational MODE: single process ***
ModuleNotFoundError: No module named 'wsgi'
unable to load app 0 (mountpoint='') (callable not found or import error)
*** no app loaded. going in full dynamic mode ***
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI worker 1 (and the only) (pid: 14095, cores: 1)
```

Visit your server’s IP address with :5000 appended to the end in your web browser again:
```bash
http://your_server_ip:5000
```

We’re now done with our virtual environment, so we can deactivate it:
```bash
deactivate
```

> Any Python commands will now use the system’s Python environment again.

#### Creating a uWSGI Configuration File
You have tested that uWSGI is able to serve your application, but ultimately you will want something more robust for long-term usage. You can create a uWSGI configuration file with the relevant options for this.

Let’s place that file in our project directory and call it ```myproject.ini```:
```bash
$ vi ./myproject.ini
```

Inside, we will start off with the ```[uwsgi]``` header so that uWSGI knows to apply the settings. We’ll specify two things: the module itself, by referring to the wsgi.py file minus the extension, and the callable within the file, ```app```:

```bash
[uwsgi]
module = wsgi:app
```

Next, we’ll tell uWSGI to start up in master mode and spawn five worker processes to serve actual requests:

```bash
[uwsgi]
module = wsgi:app

master = true
processes = 5

```

When you were testing, you exposed uWSGI on a network port. However, you’re going to be using Nginx to handle actual client connections, which will then pass requests to uWSGI. Since these components are operating on the same computer, a Unix socket is preferable because it is faster and more secure. Let’s call the socket ```myproject.sock``` and place it in this directory.

Let’s also change the permissions on the socket. We’ll be giving the Nginx group ownership of the uWSGI process later on, so we need to make sure the group owner of the socket can read information from it and write to it. We will also clean up the socket when the process stops by adding the ```vacuum``` option:

```bash
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true
```

The last thing we’ll do is set the die-on-term option. This can help ensure that the init system and uWSGI have the same assumptions about what each process signal means. Setting this aligns the two system components, implementing the expected behavior:

```bash
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true

die-on-term = true
```

> You may have noticed that we did not specify a protocol like we did from the command line. That is because by default, uWSGI speaks using the ```uwsgi``` protocol, a fast binary protocol designed to communicate with other servers. Nginx can speak this protocol natively, so it’s better to use this than to force communication by HTTP.

### Step 5 — Creating a systemd Unit File

Next, let’s create the systemd service unit file. Creating a systemd unit file will allow Ubuntu’s init system to automatically start uWSGI and serve the Flask application whenever the server boots.

Create a unit file ending in .service within the /etc/systemd/system directory to begin:
```bash
$ sudo vi /etc/systemd/system/myproject.service
```

Inside, we’ll start with the ```[Unit]``` section, which is used to specify metadata and dependencies. Let’s put a description of our service here and tell the init system to only start this after the networking target has been reached:

```bash
[Unit]
Description=uWSGI instance to serve myproject
After=network.target
```

Next, let’s open up the ```[Service]``` section. This will specify the user and group that we want the process to run under. Let’s give our regular user account ownership of the process since it owns all of the relevant files. Let’s also give group ownership to the ```www-data``` group so that Nginx can communicate easily with the uWSGI processes. Remember to replace the username here with your username:

```bash
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=bbearce
Group=www-data
```

Next, let’s map out the working directory and set the ```PATH``` environmental variable so that the init system knows that the executables for the process are located within our virtual environment. Let’s also specify the command to start the service. Systemd requires that we give the full path to the uWSGI executable, which is installed within our virtual environment. We will pass the name of the ```.ini``` configuration file we created in our project directory.

Remember to replace the username and project paths with your own information:

```bash
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=bbearce
Group=www-data
WorkingDirectory=/home/bbearce/Documents/flask_w_nginx/myproject
Environment="PATH=/home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin"
ExecStart=/home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi --ini myproject.ini
```

Finally, let’s add an ```[Install]``` section. This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:

```bash
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=bbearce
Group=www-data
WorkingDirectory=/home/bbearce/Documents/flask_w_nginx/myproject
Environment="PATH=/home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin"
ExecStart=/home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi --ini myproject.ini

[Install]
WantedBy=multi-user.target
```

With that, our systemd service file is complete. Save and close it now.

We can now start the uWSGI service we created and enable it so that it starts at boot:

```bash
$ sudo systemctl start myproject
$ sudo systemctl enable myproject

Created symlink /etc/systemd/system/multi-user.target.wants/myproject.service → /etc/systemd/system/myproject.service.
```

Let’s check the status:

```bash
$ sudo systemctl status myproject

● myproject.service - uWSGI instance to serve myproject
     Loaded: loaded (/etc/systemd/system/myproject.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2021-01-17 16:23:03 EST; 1min 3s ago
   Main PID: 17331 (uwsgi)
      Tasks: 6 (limit: 28605)
     Memory: 19.6M
     CGroup: /system.slice/myproject.service
             ├─17331 /home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi ->
             ├─17345 /home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi ->
             ├─17346 /home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi ->
             ├─17347 /home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi ->
             ├─17348 /home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi ->
             └─17349 /home/bbearce/Documents/flask_w_nginx/myproject/myprojectenv/bin/uwsgi ->

Jan 17 16:23:03 pop-os uwsgi[17331]: mapped 437520 bytes (427 KB) for 5 cores
Jan 17 16:23:03 pop-os uwsgi[17331]: *** Operational MODE: preforking ***
Jan 17 16:23:03 pop-os uwsgi[17331]: WSGI app 0 (mountpoint='') ready in 0 seconds on interpr>
Jan 17 16:23:03 pop-os uwsgi[17331]: *** uWSGI is running in multiple interpreter mode ***
Jan 17 16:23:03 pop-os uwsgi[17331]: spawned uWSGI master process (pid: 17331)
Jan 17 16:23:03 pop-os uwsgi[17331]: spawned uWSGI worker 1 (pid: 17345, cores: 1)
Jan 17 16:23:03 pop-os uwsgi[17331]: spawned uWSGI worker 2 (pid: 17346, cores: 1)
Jan 17 16:23:03 pop-os uwsgi[17331]: spawned uWSGI worker 3 (pid: 17347, cores: 1)
lines 1-22

```

### Step 6 — Configuring Nginx to Proxy Requests

> This is where we start using the ```example.com``` site from earlier.

Edit the ```/etc/nginx/sites-available/example.com``` server block. Within this block, we’ll include the uwsgi_params file that specifies some general uWSGI parameters that need to be set. We’ll then pass the requests to the socket we defined using the uwsgi_pass directive:

```bash
server {
        listen 81;

        server_name example.com www.example.com;

        location / {
            include uwsgi_params;
            uwsgi_pass unix:/home/bbearce/Documents/flask_w_nginx/myproject/myproject.sock;
        }
}
```

With that, we can test for syntax errors by typing:
```bash
$ sudo nginx -t
```

If this returns without indicating any issues, restart the Nginx process to read the new configuration:
```bash
$ sudo systemctl restart nginx
```

You should see your application output:

![flask_app](./flask_app.jpg)


If you encounter any errors, trying checking the following:

* ```$ sudo less /var/log/nginx/error.log```: checks the Nginx error logs.  
* ```$ sudo less /var/log/nginx/access.log```: checks the Nginx access logs.  
* ```$ sudo journalctl -u nginx```: checks the Nginx process logs.  
* ```$ sudo journalctl -u myproject```: checks your Flask app’s uWSGI logs.  

### Step 7 — Securing the Application

#### Install Certbot

```bash
$ sudo snap install --classic certbot
```

#### Prepare the Certbot command

```bash
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

#### Choose how you'd like to run Certbot

Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.

```bash
$ sudo certbot --nginx
```

If you're feeling more conservative and would like to make the changes to your Nginx configuration by hand, run this command.

```bash
$ sudo certbot certonly --nginx
```

> I chose the first option

Here is the dialog:

```bash
$ sudo certbot --nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): bbearce@gmail.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: n
Account registered.

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: example.com
2: www.example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Requesting a certificate for example.com
Performing the following challenges:
http-01 challenge for example.com
Waiting for verification...
Challenge failed for domain example.com
http-01 challenge for example.com
Cleaning up challenges
Some challenges have failed.

IMPORTANT NOTES:
 - The following errors were reported by the server:

   Domain: example.com
   Type:   unauthorized
   Detail: Invalid response from
   http://example.com/.well-known/acme-challenge/lV3rb3swo9cvCOjBIL4RUaHBKGbcDjxFx3iVcbudgng
   [2606:2800:220:1:248:1893:25c8:1946]: "<!doctype
   html>\n<html>\n<head>\n    <title>Example Domain</title>\n\n
   <meta charset=\"utf-8\" />\n    <meta http-equiv=\"Content-type"

   To fix these errors, please make sure that your domain name was
   entered correctly and the DNS A/AAAA record(s) for that domain
   contain(s) the right IP address.

```
So it failed as I don't actually own example.com