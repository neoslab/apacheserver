## Onion Server

Setup for server running Apache2 as hosting services on Ubuntu 24.04 server freshly installed.

* * *

#### Change the user password

Change user password

```shell
passwd ${USER}
```

* * *

#### Prepare the environment

Configure APT sources

```shell
sudo add-apt-repository -y main && sudo add-apt-repository -y restricted && sudo add-apt-repository -y universe && sudo add-apt-repository -y multiverse
```

Keep system safe

```shell
sudo apt -y update && sudo apt -y upgrade && sudo apt -y dist-upgrade
sudo apt -y remove && sudo apt -y autoremove
sudo apt -y clean && sudo apt -y autoclean
```

Disable error reporting

```shell
sudo sed -i "s/enabled=1/enabled=0/" /etc/default/apport
```

Edit SSH settings

```shell
sudo sed -i "s/#Port 22/Port 49622/" /etc/ssh/sshd_config
sudo sed -i "s/#LoginGraceTime 2m/LoginGraceTime 2m/" /etc/ssh/sshd_config
sudo sed -i "s/#PermitRootLogin prohibit-password/PermitRootLogin no/" /etc/ssh/sshd_config
sudo sed -i "s/#StrictModes yes/StrictModes yes/" /etc/ssh/sshd_config
sudo systemctl restart sshd.service
```

Install prerequisite packages

```shell
sudo apt -y install libsodium-dev
```

Install necessary softwares

```shell
sudo apt -y install apache2 apt-transport-https autoconf curl build-essential fail2ban gcc git gpg make nano software-properties-common unattended-upgrades wget
sudo systemctl status apache2.service
sudo systemctl enable tor.service
```

Install PHP 8.3

```shell
sudo add-apt-repository -y ppa:ondrej/php
sudo apt -y update
sudo apt -y install php8.3 php8.3-cli php8.3-{bz2,curl,mbstring,intl} libapache2-mod-php8.3 
sudo a2enmod php8.3
sudo systemctl reload apache2.service
sudo truncate -s 0 /var/www/html/index.html
```

Edit Fail2Ban settings

```shell
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo systemctl restart fail2ban.service 
sudo systemctl status fail2ban.service 
```

Setting the firewall

```shell
sudo ufw app list
sudo ufw allow OpenSSH
sudo ufw allow 49622/tcp
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status
```

Reboot server

```shell
sudo reboot now
```

* * *

#### Automated Setup

If you prefer and in order to save time, you can use our deployment script which reproduces all the commands above.

```shell
cd /tmp/ && wget -O - https://raw.githubusercontent.com/neoslab/apacheserver/main/install.sh | bash
```

* * *

#### Create Virtual Host

We will now create a virtual host which will host the content of our site.

```shell
sudo mkdir -p /var/www/<website-domain>/public_html
sudo chown -R $USER:$USER /var/www/<website-domain>/public_html
sudo chmod -R 755 /var/www
echo "Hello Domain" | sudo tee /var/www/<website-domain>/public_html/index.html >/dev/null
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/<website-domain>.conf
sudo nano /etc/apache2/sites-available/<website-domain>.conf
```

Copy and edit the below content according to your hidden service URL.

```
<VirtualHost *:80>
    ServerAdmin webmaster@<website-domain>
    ServerName <website-domain>
    ServerAlias www.<website-domain>
    DocumentRoot /var/www/<website-domain>/public_html 
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost> 
```

Add the website to Apache configuration

```shell
sudo a2ensite <website-domain>.conf
```

Check the configuration

```shell
sudo apache2ctl configtest
```

Restart Apache server

```shell
sudo systemctl restart apache2.service
```

* * *

#### Conclusion


We can now visit our website using **Tor Browser** and pointing to the onion address we just configured above.