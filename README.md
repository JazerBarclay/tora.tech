# Tora Tech Website

![Screenshot](images/site-screenshot.png?raw=true "Website")

Table of Contents
=================
   * **[Server Build](#server-build)**
      * [Universe Repository](#add-universe-repository)
      * [Update & Upgrade](#update,-upgrade-and-remove-old)
      * [Install Packages](#install-packages)
      * [Set the Hostname](#set-the-hostname)
      * [Add Groups](#add-the-wheel-and-dev-groups)
      * [Create New User](#create-the-new-user)
      * [Set Password](#set-password-for-new-user)
      * [Set Home Permissions](#set-the-user-home-to-700)
      * [Test New User](#test-new-user-exists)
      * [Update Sudoers File](#update-the-sudoers-file)
      * [Update Firewall](#update-firewall-with-port-22-for-ssh-and-enable)
      * [Inspect Unattended-Upgrades Settings](#inspect-unattended-upgrades-settings)
      * [Add Unattended-Upgrades Period](#add-unattended-upgrades-period-for-auto-upgrades)
      * [Update SSH Config](#update-ssh-config-to-disable-root-login-over-ssh)
      * [Reboot System](#reboot-system-to-restart-all-services-with-the-updates)
   * **[Configure Website](#configure-website)**
      * [Setting Up Nginx Server Block](#setting-up-nginx-server-block)

## Server Build
### Make sure the root password is set
You want to run this if you have already got a profile and have yet to setup the root password
```
passwd
```

### Add Universe Repository
Some hosting providers do not include the universe packages. Best add them just incase.
```
sudo add-apt-repository universe
```

### Update, Upgrade and Remove old
Since we are on a clean installation , we can run a dist-upgrade which will install new files for the installed packages.
Check out this link for the difference between upgrade and dist-upgrade https://askubuntu.com/a/226213
```
apt update && apt -y dist-upgrade && apt -y autoremove
```

### Install Packages
Vim, curl and git are very useful when needing to download, edit and pull repositories to your server. Unattended upgrades, fail2ban and ubuntu firewall are vital to ensure the system is secure against attackers and keeping the system up to date when not being adminstered.
We will be installing nginx for the hosting platformm mysql for the database and php for a more flexible layout.
```
apt -y install vim curl git git-core unattended-upgrades fail2ban ufw nginx mysql-server php-fpm php-mysql
```

### Set the Hostname
Sets the hostname of the system.
```
hostnamectl set-hostname tora.tech
```

### Add the wheel and dev groups
I add the wheel group back for sanity reasons and a dev group for if I have another developer with ssh access to the system.
```
groupadd wheel
groupadd dev
```

### Create the new user
I create a new user account if there is only a root user with the wheel and dev groups
```
useradd -m -G wheel,dev -s /bin/bash username
```

### Set password for new user
When we disable root ssh login, we could lock ourselves out if we dont set a password for this user
```
passwd username
```

### Set the user home to 700
Set the user home to read, write and execute for only the user
```
chmod 700 -R /home/username
```

### Test new user exists
We can use su with the username to login to the new user we created and `exit` to return
```
su username
```

### Update the sudoers file
I add the new wheel group back to the sudoers file `%wheel ALL=(ALL) ALL`. `Defaults rootpw` will make all sudo commands require the root password rather than the users password. I set this on servers for a separate level of security.
```
vim /etc/sudoers
```

### Update firewall with port 22 for ssh and enable
Status check the state of the firewall. We will open port 22 for ssh and enable the service. We can then check the status again to ensure the service is working.
```
ufw status
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
ufw status
```

### Inspect unattended-upgrades settings
We can enable auto updates with restarts by uncommenting the line `//Unattended-Upgrade::Automatic-Reboot "false";` and setting it to true. We can also set a reboot time with the line `//Unattended-Upgrade::Automatic-Reboot-Time "02:00";`
```
vim /etc/apt/apt.conf.d/50unattended-upgrades
```

### Add unattended-upgrades period for auto upgrades 
We can add the line `APT::Periodic::Unattended-Upgrade "1";` to the bottom of the file for enabling unattended upgrades
```
vim /etc/apt/apt.conf.d/10periodic
```

### Update ssh config to disable root login over ssh
We can change the line `PermitRootLogin yes` to `PermitRootLogin no` to disable root login over ssh
```
vim /etc/ssh/sshd_config
```

### Reboot system to restart all services with the updates
```
reboot
```

## Configure Website
### Setting Up Nginx Server Block

```

server {

        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.php index.html index.htm index.nginx-debian.html;

        server_name www.tora.tech tora.tech;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        
                # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
                # With php-cgi (or other tcp sockets):
                #fastcgi_pass 127.0.0.1:9000;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        location ~ /\.ht {
                deny all;
        }
}


```
