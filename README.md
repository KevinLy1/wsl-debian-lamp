# Table of contents

- [Prerequisites](#prerequisites)
- [LAMP Setup](#lamp-setup)
- [Installation](#installation)
  - [Apache](#apache)
    - [Useful commands](#useful-commands)
  - [MariaDB / MySQL (pick one only)](#mariadb--mysql-pick-one-only)
    - [Useful commands](#useful-commands)
  - [PHP](#php)
  - [phpMyAdmin](#phpmyadmin)
  - [Claim ownership of /var/www/html](#claim-ownership-of-varwwwhtml)
  - [Create a user with all privileges in MariaDB / MySQL](#create-a-user-with-all-privileges-in-mariadb--mysql)
    - [Optional - Configure Auto-login in phpMyAdmin](#optional---configure-auto-login-in-phpmyadmin)
- [Commands for controlling the LAMP stack](#commands-for-controlling-the-lamp-stack)
  - [Start](#start)
  - [Check](#check)
  - [Stop](#stop)
  - [Optional - Set up aliases for shorter commands](#optional---set-up-aliases-for-shorter-commands)

<!-- ## Language

[Français](README.fr.md) | [English](README.md) -->

# Prerequisites

1. Setup WSL2
    - WSL2 (can be installed from Microsoft Store: https://apps.microsoft.com/store/detail/windows-subsystem-for-linux/9P9TQF7MRM4R)
    - Debian (can be installed from Microsoft Store: https://apps.microsoft.com/store/detail/debian/9MSVKQC78PK6)
    - Press `Win+R`, then run `optionalfeatures.exe`. Enable "Windows Subsystem for Linux"
    
2. Enable systemd in Debian WSL with
    ```sh
    sudo nano /etc/wsl.conf
    ```
    and add these lines
    ```sh
    [boot]
    systemd=true
    ```

    Then press `CTRL+O` to save and `CTRL+X` to exit.

    Restart WSL by running the following command in PowerShell
    ```powershell
    wsl --shutdown
    ```
3. Update the system
    ```sh
    sudo apt update && sudo apt upgrade
    ```

# LAMP Setup

LAMP stands for:
- **L**inux
- **A**pache
- **M**ySQL / **M**ariaDB
- **P**HP

The instructions below focus on setting up a local development environment (not a production one, with higher security).

# Installation

## Apache
Install Apache
```sh
sudo apt install apache2-utils apache2
```
### Useful commands
Start Apache
```sh
sudo systemctl start apache2
```
You can check http://localhost/ to see if Apache is running correctly.

Check Apache
```sh
sudo systemctl status apache2
```
Optional - Disable automatic start of Apache with WSL
```sh
sudo systemctl disable apache2
```

## MariaDB / MySQL (pick one only)
Install MariaDB or MySQL
```sh
sudo apt install mariadb-server
sudo apt install mysql-server
```
Start MariaDB or MySQL with
```sh
sudo systemctl start mariadb
sudo systemctl start mysql
```
Run
```sh
sudo mariadb-secure-installation
sudo mysql-secure-installation
    #> Enter current password for root (enter for none):
    #> Switch to unix_socket authentication [Y/n]: N
    #> Change the root password? [Y/n]: N
    #> Remove anonymous users? [Y/n]: Y
    #> Disallow root login remotely? [Y/n]: Y
    #> Remove test database and access to it? [Y/n]: Y
    #> Reload privilege tables now? [Y/n]: Y
```
### Useful commands
Connect to MariaDB / MySQL server
```sh
sudo mariadb -u root -p
sudo mysql -u root -p
```
Optional - Disable automatic start of MariaDB / MySQL with WSL
```sh
sudo systemctl disable mariadb
sudo systemctl disable mysql
```

## PHP
Install PHP
```sh
sudo apt install php libapache2-mod-php php-cli php-fpm php-json php-pdo php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath
```
Enable the specified module within the apache2 configuration.
```sh
sudo a2enmod php8.*
```
Restart Apache
```sh
sudo systemctl restart apache2
```

## phpMyAdmin
```sh
sudo apt-get install phpmyadmin
    #> In the next step, select "apache2" by pressing the spacebar.
    #> Select “Yes” when asked whether to use dbconfig-common to set up the database.
```

## Claim ownership of /var/www/html

Run the following command (replace `<username>` with the user name in WSL)
```sh
sudo chown <username> /var/www/html
```

## Create a user with all privileges in MariaDB / MySQL
Since `root` is remotely disabled in MariaDB / MySQL, we need to create a new user with all privileges
```sh
sudo mariadb -u root -p
sudo mysql -u root -p
```
And run (replace `USER` and `PASSWORD`)
```sql
GRANT ALL ON *.* TO 'USER'@'localhost' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```
Example (with a user "mariadb" and password "mariadb")
```sql
GRANT ALL ON *.* TO 'mariadb'@'localhost' IDENTIFIED BY 'mariadb' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```
phpMyAdmin is accessible at http://localhost/phpmyadmin. \
Now, you can login to the database with the new user.

### Optional - Configure Auto-login in phpMyAdmin

In order to make it easier to edit from Windows, run
```sh
sudo chown <username> /etc/phpmyadmin/config.inc.php
```
Edit, with the text editor of your choice, `/etc/phpmyadmin/config.inc.php`
- Find `$cfg['Servers'][$i]['auth_type'] = 'cookie';`
- Replace, with the user and password previously set, by
    ```php
    $cfg['Servers'][$i]['auth_type'] = 'config';
    $cfg['Servers'][$i]['user'] = 'USER';
    $cfg['Servers'][$i]['password'] = 'PASSWORD';
    ```
    
    Example (with a user "mariadb" and password "mariadb")
    ```php
    $cfg['Servers'][$i]['auth_type'] = 'config';
    $cfg['Servers'][$i]['user'] = 'mariadb';
    $cfg['Servers'][$i]['password'] = 'mariadb';
    ```
- Save

# Commands for controlling the LAMP stack

## Start
With MariaDB
```sh
sudo systemctl start apache2 && sudo systemctl start mariadb
```

With MySQL
```sh
sudo systemctl start apache2 && sudo systemctl start mysql
```

## Check
With MariaDB
```sh
sudo systemctl status apache2 && sudo systemctl status mariadb
```

With MySQL
```sh
sudo systemctl status apache2 && sudo systemctl status mysql
```

## Stop
With MariaDB
```sh
sudo systemctl stop apache2 && sudo systemctl stop mariadb
```

With MySQL
```sh
sudo systemctl stop apache2 && sudo systemctl stop mysql
```


## Optional - Set up aliases for shorter commands

With MariaDB
```sh
cd ~ && touch .bash_aliases
echo 'alias lampstatus="sudo systemctl status apache2; sudo systemctl status mariadb"' >> .bash_aliases
echo 'alias lampstart="sudo systemctl start apache2; sudo systemctl start mariadb"' >> .bash_aliases
echo 'alias lampstop="sudo systemctl stop mariadb; sudo systemctl stop apache2"' >> .bash_aliases
echo 'alias lamprestart="lampstop; lampstart"' >> .bash_aliases
```

With MySQL
```sh
cd ~ && touch .bash_aliases
echo 'alias lampstatus="sudo systemctl status apache2; sudo systemctl status mysql"' >> .bash_aliases
echo 'alias lampstart="sudo systemctl start apache2; sudo systemctl start mysql"' >> .bash_aliases
echo 'alias lampstop="sudo systemctl stop mysql; sudo systemctl stop apache2"' >> .bash_aliases
echo 'alias lamprestart="lampstop; lampstart"' >> .bash_aliases
```

With these aliases, you can use `lampstatus`, `lampstart`, `lampstop` and `lamprestart`.