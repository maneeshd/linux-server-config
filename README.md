# Linux Server Configuration

Udacity Full Stack Developer Nanodegree :: Deploying Item Catalog web application in a linux server

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- [Linux Server Configuration](#linux-server-configuration)
  - [Deployment Details](#deployment-details)
  - [Deployment Steps](#deployment-steps)
    - [1. Initial login to the server as user `ubuntu`](#1-initial-login-to-the-server-as-user-ubuntu)
    - [2. Update the operating system pakages and reboot if required](#2-update-the-operating-system-pakages-and-reboot-if-required)
    - [3. Configure automatic security and critical updates](#3-configure-automatic-security-and-critical-updates)
    - [4. Set timezone to UTC](#4-set-timezone-to-utc)
    - [5. Create user `grader`](#5-create-user-grader)
    - [6. Create passwords for users `ububtu` and `grader` to do ssh-copy-id](#6-create-passwords-for-users-ububtu-and-grader-to-do-ssh-copy-id)
    - [7. Add user `grader` to sudoers list](#7-add-user-grader-to-sudoers-list)
    - [8. Create SSH Keypairs for users `ubuntu` and `grader`](#8-create-ssh-keypairs-for-users-ubuntu-and-grader)
    - [9. Create a keypair in your local machine](#9-create-a-keypair-in-your-local-machine)
    - [10. Enable SSH logins through passwords in server temporarily](#10-enable-ssh-logins-through-passwords-in-server-temporarily)
    - [11. `ssh-copy-id` the local machines' public key to `grader`](#11-ssh-copy-id-the-local-machines-public-key-to-grader)
    - [12. Disable SSH logins through passwords in server permanently](#12-disable-ssh-logins-through-passwords-in-server-permanently)
    - [13. Install Apache2 and enable required proxy modules](#13-install-apache2-and-enable-required-proxy-modules)
    - [14. Configure firewall to allow OpenSSH, 'Apache Full', port 80, 123 and 2200](#14-configure-firewall-to-allow-openssh-apache-full-port-80-123-and-2200)
    - [15. Enable port 2200 and HTTPS in the Lightsail VM Networking settings](#15-enable-port-2200-and-https-in-the-lightsail-vm-networking-settings)
    - [16. Disable root login through SSH, change SSH port and add aloowed users to SSH config](#16-disable-root-login-through-ssh-change-ssh-port-and-add-aloowed-users-to-ssh-config)
    - [17. Install Pip and Virtualenv, create a virtual environment for webapp](#17-install-pip-and-virtualenv-create-a-virtual-environment-for-webapp)
    - [18. Install PostgreSQL and setup item-catalog database](#18-install-postgresql-and-setup-item-catalog-database)
    - [19. Clone item-catalog git repository, put oauth2 data and install python packages](#19-clone-item-catalog-git-repository-put-oauth2-data-and-install-python-packages)
    - [20. Configure gunicorn server and systemd service to manage the backend server](#20-configure-gunicorn-server-and-systemd-service-to-manage-the-backend-server)
    - [21. Configue Apache2 server to be a reverse proxy, add domain name](#21-configue-apache2-server-to-be-a-reverse-proxy-add-domain-name)
    - [22. Configure HTTPS and SSL](#22-configure-https-and-ssl)
  - [References](#references)

## Deployment Details

- Application URL: https://md-item-catalog.ml

- Virtual Server: Amazon Lightsail Instance
  
- Operating System: Ubuntu 18.04 LTS

- IP Address: `13.233.215.53`

- SSH Port: `2200` (Only Key based logins supported)
  
- Web Server: Apache2 Web Server acting as reverse proxy
  
- Backend Server: Gunicorn with 4 workers running the web app listening to port 5000 in localhost. Managed by systemd.
  
## Deployment Steps

### 1. Initial login to the server as user `ubuntu`

```bash
$ ssh -i ~/.ssh/Lightsail_Key.pem ubuntu@13.233.215.53
```

### 2. Update the operating system pakages and reboot if required

```bash
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
$ sudo reboot
```

### 3. Configure automatic security and critical updates

Follow the official documentation: [Ubuntu Automatic Update Configuration](https://help.ubuntu.com/lts/serverguide/automatic-updates.html.en)

### 4. Set timezone to UTC

Check if the current timezone is set to UTC using:

```bash
$ date
Tue Jan 29 15:42:28 UTC 2019
```

If not UTC set timezone to UTC using the command below:

(select 'None of the above' from the menu and then select 'UTC'.)

```bash
$ sudo dpkg-reconfigure tzdata

Current default time zone: 'Etc/UTC'
Local time is now:      Tue Jan 29 15:45:18 UTC 2019.
Universal Time is now:  Tue Jan 29 15:45:18 UTC 2019.
```

### 5. Create user `grader`

```bash
$ sudo adduser grader
Adding user 'grader' ...
Adding new group 'grader' (1002) ...
Adding new user 'grader' (1002) with group 'grader' ...
Creating home directory '/home/grader' ...
Copying files from '/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for grader
Enter the new value, or press ENTER for the default
        Full Name []: Udacity Grader
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
```

### 6. Create passwords for users `ububtu` and `grader` to do ssh-copy-id

```bash
$ sudo passwd ubuntu
Enter new UNIX password: ********
Retype new UNIX password: ********
passwd: password updated successfully

$ sudo passwd grader
Enter new UNIX password: ********
Retype new UNIX password: ********
passwd: password updated successfully
```

### 7. Add user `grader` to sudoers list

```bash
$ usermod -aG sudo grader
```

### 8. Create SSH Keypairs for users `ubuntu` and `grader`

```bash
# As user ubuntu
$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Created directory '/home/ubuntu/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:6xESKDYeewIYOE2upVA71LwRzayaA0DttwPHAWwVmzs ubuntu@ip-172-26-6-133
The keys randomart image is:
+---[RSA 4096]----+
|++*++B.          |
|==.=+o*          |
|=.% o*o          |
|oB Oo+..         |
|o.+o=Eo S        |
|  +o o.. o       |
|   .  . o        |
|       . .       |
|        .        |
+----[SHA256]-----+

# As user grader
$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/grader/.ssh/id_rsa):
Created directory '/home/grader/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/grader/.ssh/id_rsa.
Your public key has been saved in /home/grader/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:6xESKDYeewIYOE2upVA71LwRzayaA0DttwPHAWwVmzs grader@ip-172-26-6-133
The keys randomart image is:
+---[RSA 4096]----+
|++*++B.          |
|==.=+o*          |
|=.% o*o          |
|oB Oo+..         |
|o.+o=Eo S        |
|  +o o.. o       |
|   .  . o        |
|       . .       |
|        .        |
+----[SHA256]-----+
```

### 9. Create a keypair in your local machine

```bashr
$ ssh-keygen -t rsa -b 4096 -C maneeshd77@gmail.com
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/mdivana/.ssh/id_rsa): fsnd_key
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in fsnd_key.
Your public key has been saved in fsnd_key.pub.
The key fingerprint is:
SHA256:GKW7yzA1J1qkr1Cr9MhUwAbHbF2NrIPEgZXeOUOz3Us maneeshd77@gmail.com
The keys randomart image is:
+---[RSA 2048]----+
|.*++ o.o.        |
|.+B + oo.        |
| +++ *+.         |
| .o.Oo.+E        |
|    ++B.S.       |
|   o * =.        |
|  + = o          |
| + = = .         |
|  + o o          |
+----[SHA256]-----+
```

### 10. Enable SSH logins through passwords in server temporarily

```bash
$ sudo vi /etc/ssh/sshd_config
Locate 'PasswordAuthentication no' and change it to 'PasswordAuthentication yes'. Save.
$ sudo serivce sshd restart
```

### 11. `ssh-copy-id` the local machines' public key to `grader`

```bash
$ ssh-copy-id -i ~/.ssh/fsnd_key grader@13.233.215.53
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/mdivana/.ssh/fsnd_key.pub"
The authenticity of host '13.233.215.53 (13.233.215.53)' cannot be established.
ECDSA key fingerprint is SHA256:/9TWCe3NH67EQErMsagjifJ8hFa7uIyu0Nq6r1Pu1Iw.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
grader@13.233.215.53 password:

Number of key(s) added: 1

Now try logging into the machine, with:   'ssh -i ~/.ssh/fsnd_key grader@13.233.215.53'
and check to make sure that only the key(s) you wanted were added.

Now we can login without using password from our local machine.
```

### 12. Disable SSH logins through passwords in server permanently

```bash
$ sudo vi /etc/ssh/sshd_config
Locate 'PasswordAuthentication yes' and change it to 'PasswordAuthentication no'. Save.
$ sudo serivce sshd restart
```

### 13. Install Apache2 and enable required proxy modules

```bash
$ sudo apt-get install apache2
$ sudo a2enmod
give the below list of modules to enable

proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html

$ sudo systemctl restart apache2
```

### 14. Configure firewall to allow OpenSSH, 'Apache Full', port 80, 123 and 2200

```bash
$ sudo ufw app list
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH

$ sudo ufw allow OpenSSH
Rules updated
Rules updated (v6)

$ sudo ufw allow 'Apache Full'
Rules updated
Rules updated (v6)

$ sudo ufw allow 80
Rules updated
Rules updated (v6)

$ sudo ufw allow 2200
Rules updated
Rules updated (v6)

$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

$ sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] OpenSSH                    ALLOW IN    Anywhere
[ 2] Apache Full                ALLOW IN    Anywhere
[ 3] 80                         ALLOW IN    Anywhere
[ 4] 2200                       ALLOW IN    Anywhere
[ 5] OpenSSH (v6)               ALLOW IN    Anywhere (v6)
[ 6] Apache Full (v6)           ALLOW IN    Anywhere (v6)
[ 7] 80 (v6)                    ALLOW IN    Anywhere (v6)
[ 8] 2200 (v6)                  ALLOW IN    Anywhere (v6)

$ sudo reboot
```

### 15. Enable port 2200 and HTTPS in the Lightsail VM Networking settings

Open the Lightsail Instance console. Go to Netorking tab. Then in 'Firewall' settings

1. Add a Custom TCP protocol with Port as 2200 to be enabled.
2. Add HTTPS protocol to be enabled.

### 16. Disable root login through SSH, change SSH port and add aloowed users to SSH config

```bash
$ sudo vi /etc/ssh/sshd_config
Locate 'Post 22'. Uncomment line and change to 'Post 2200' (without quotes)
Locate 'PasswordAuthentication'. Change to 'PasswordAuthentication no' (without quotes)
Add line 'AllowUsers ubuntu grader' (without quotes)
Add line 'PermitRootLogin no' (without quotes)
Save.

$ sudo service sshd restart

Now the we can only ssh to server using: 'ssh -i ~/.ssh/fsnd_key grader@13.233.215.53'
```

### 17. Install Pip and Virtualenv, create a virtual environment for webapp

```bash
# As grader
$ sudo apt-get install build-essential python3-pip

$ sudo pip3 install virtualenv -U

$ virtualenv fenv
Using base prefix '/usr'
New python executable in /home/grader/fenv/bin/python3
Also creating executable in /home/grader/fenv/bin/python
Installing setuptools, pip, wheel...
done.

$ source ~/fenv/bin/activate
(fenv) $ pip list
Package    Version
---------- -------
pip        19.0.1
setuptools 40.7.1
wheel      0.32.3
```

### 18. Install PostgreSQL and setup item-catalog database

```sql
-- Install postgresql
$ sudo apt-get install postgresql

-- Switch into postgresql superuser 'postgres'
$ sudo su - postgres

-- Enter psql shell
$ psql

-- Create user 'catalog'
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
CREATE ROLE

-- Create database 'catalog'
postgres=# CREATE DATABASE catalog;
CREATE DATABASE

-- Grant all previleges on database 'catalog' to user 'catalog'
postgres=# REVOKE ALL ON SCHEMA public FROM public;
REVOKE
postgres=# GRANT ALL ON SCHEMA public TO catalog;
GRANT
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
GRANT

-- Exit psql shell
postgres=# \q

-- Exit postgres user
$ exit

Use 'postgresql://catalog:catalog@localhost/catalog' as database url in 'db_models.py' 'db_util.py' and 'server.py' in item-catalog
```

### 19. Clone item-catalog git repository, put oauth2 data and install python packages

```bash
(fenv) $ git clone https://github.com/maneeshd/restaurants-menu.git item-catalog
Cloning into 'item-catalog'...
remote: Enumerating objects: 93, done.
remote: Counting objects: 100% (93/93), done.
remote: Compressing objects: 100% (66/66), done.
remote: Total 93 (delta 26), reused 83 (delta 22), pack-reused 0
Unpacking objects: 100% (93/93), done.

(fenv) $ cd item-catalog

(fenv) $ pip install -r requirements.txt -U

# Put the database url in the files stated and then -
(fenv) $ python db_models.py
Using db_uri: postgresql://catalog:catalog@localhost/catalog to create models...
Database models have been created successfully.

(fenv) $ python db_util.py
Database has been populated successfully.

(fenv) $ vi wsgi.py
# Put the following inside wsgi.py
from server import APP


if __name__ == "__main__":
    APP.run("localhost", port=5000, threaded=True)
```

**Put the Google and Facebook OAuth2 data in oauth_data directory. (Refer [README.txt](oauth_data/README.txt) in oauth_data for more info)**

### 20. Configure gunicorn server and systemd service to manage the backend server

```bash
$ cd ~/item-catalog

$ mkdir gunicorn_logs

$ vi gunicorn.conf
# Put the following data inside gunicorn.conf
workers = 3
errorlog = '/home/grader/item-catalog/gunicorn_logs/errors.log'
accesslog = '/home/grader/item-catalog/gunicorn_logs/access.log'

# Create a systemd service for gunicorn
$ sudo vi /etc/systemd/system/ItemCatalog.service
# Put the following data between --- inside ItemCatalog.service
--------------------------------
[Unit]
Description=Gunicorn instance to serve Item-Catalog
After=network.target

[Service]
User=grader
Group=grader
Restart=on-failure
WorkingDirectory=/home/grader/item-catalog
ExecStart=/home/grader/fenv/bin/gunicorn -c gunicorn.conf -b localhost:5000 wsgi:APP --preload --capture-output --log-level debug

[Install]
WantedBy=multi-user.target
--------------------------------

# Reload daemon
$ sudo systemctl daemon-reload

# Enable ItemCatalog service
$ sudo systemctl enable ItemCatalog

# Start ItemCatalog serivice
$ sudo systemctl start ItemCatalog

$ sudo systemctl status ItemCatalog
● ItemCatalog.service - Gunicorn instance to serve Item-Catalog
   Loaded: loaded (/etc/systemd/system/ItemCatalog.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-01-28 13:00:07 UTC; 1 day 8h ago
 Main PID: 17468 (gunicorn)
    Tasks: 4 (limit: 547)
   CGroup: /system.slice/ItemCatalog.service
           ├─17468 /home/grader/fenv/bin/python3 /home/grader/fenv/bin/gunicorn -c gunicorn.conf -b localhost:5000 wsgi:APP --preload          ├─17491 /home/grader/fenv/bin/python3 /home/grader/fenv/bin/gunicorn -c gunicorn.conf -b localhost:5000 wsgi:APP --preload         ├─17492 /home/grader/fenv/bin/python3 /home/grader/fenv/bin/gunicorn -c gunicorn.conf -b localhost:5000 wsgi:APP --preload        └─17493 /home/grader/fenv/bin/python3 /home/grader/fenv/bin/gunicorn -c gunicorn.conf -b localhost:5000 wsgi:APP --preload
Jan 28 13:00:07 ip-172-26-6-133 systemd[1]: Stopped Gunicorn instance to serve Item-Catalog.
Jan 28 13:00:07 ip-172-26-6-133 systemd[1]: Started Gunicorn instance to serve Item-Catalog.
lines 1-13/13 (END)
```

### 21. Configue Apache2 server to be a reverse proxy, add domain name

```bash
$ sudo vi /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
    ServerName md-item-catalog.ml  
    ServerAlias www.md-item-catalog.ml

    ServerAdmin maneeshd77@gmail.com
    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>
    ProxyPreserveHost On
    <Location "/">
        ProxyPass "http://localhost:5000/"
        ProxyPassReverse "http://localhost:5000/"
    </Location>
</VirtualHost>

# Check for syntax errors
$ sudo apache2ctl configtest
Syntax OK

$ sudo apache2ctl restart
```

### 22. Configure HTTPS and SSL

```bash
# Enable mod_ssl in apache2
$ sudo a2enmod ssl

# Install Certbot
$ sudo add-apt-repository ppa:certbot/certbot
Press ENTER to accept.

$ sudo apt-get install python-certbot-apache
Certbot is now ready to use, but in order for it to configure SSL for Apache we need to assign a ServerName in 
the apache config i.e. a domain has to be assigned to ServerName and the www.domain assigned to ServerAlias.

$ cat /etc/apache2/sites-available/000-default.conf
Make sure ServerName and ServerAlias is set with required values.

$ sudo apache2ctl configtest
Syntax OK

$ sudo systemctl reload apache2

# Obtain a SSL certificate from LetsEncrypt
$ sudo certbot --apache -d md-item-catalog.ml -d www.md-item-catalog.ml
...
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2

The configuration will be updated, and Apache will reload to pick up the new settings. 
certbot will wrap up with a message telling you the process was successful and where your certificates are stored.
```

The server is now ready and secured. The web application is also secured with HTTPS/SSL encryption.

## References

- https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04
  
- https://help.ubuntu.com/lts/serverguide/automatic-updates.html.en

- https://www.ssh.com/ssh/copy-id

- https://help.ubuntu.com/community/SSH/OpenSSH/Keys
  
- https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart

- https://www.vioan.eu/blog/2016/10/10/deploy-your-flask-python-app-on-ubuntu-with-apache-gunicorn-and-systemd/

- https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04
