# Table of Contents

- [Prerequisites](#prerequisites)
- [LAMP Setup](#lamp-setup)
- [Installation](#installation)
  * [Apache](#apache)
    + [Useful commands](#useful-commands)
  * [MariaDB / MySQL (pick one only)](#mariadb--mysql-pick-one-only)
    + [Useful commands](#useful-commands-1)
  * [PHP](#php)
  * [phpMyAdmin](#phpmyadmin)
  * [Create a user with all privileges in MariaDB / MySQL](#create-a-user-with-all-privileges-in-mariadb--mysql)
    + [Optional - Configure Auto-login in phpMyAdmin](#optional---configure-auto-login-in-phpmyadmin)
- [Commands for controlling the LAMP stack](#commands-for-controlling-the-lamp-stack)
  * [Start](#start)
  * [Check](#check)
  * [Stop](#stop)
  * [Optional - Set up aliases for shorter commands](#optional---set-up-aliases-for-shorter-commands)
- [Additional stuff (Git, NodeJS, Composer, Symfony CLI)](#additional-stuff-git-nodejs-composer-symfony-cli)
  * [Git](#git)
  * [NodeJS](#nodejs)
  * [Composer](#composer)
  * [Symfony CLI](#symfony-cli)
- [Backup WSL](#backup-wsl)

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
Automatically start Apache with WSL
```sh
sudo systemctl enable apache2
```

## MariaDB / MySQL (pick one only)
Install MariaDB or MySQL
```sh
sudo apt install mariadb-server
sudo apt install mysql-server
```
Start MariaDB with
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
Automatically start MariaDB / MySQL with WSL
```sh
sudo systemctl enable mariadb
sudo systemctl enable mysql
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

## Create a user with all privileges in MariaDB / MySQL
Since `root` is remotely disabled in MariaDB / MySQL, we need to create a new user with all privileges
```sh
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
sudo chmod 777 /etc/phpmyadmin/config.inc.php
```
Edit, with the text editor of your choice, `/etc/phpmyadmin/config.inc.php`
- Find `$cfg['Servers'][$i]['auth_type'] = 'cookie';`
- Replace, with the user and password previously set, by
    ```
    $cfg['Servers'][$i]['auth_type'] = 'config';
    $cfg['Servers'][$i]['user'] = 'USER';
    $cfg['Servers'][$i]['password'] = 'PASSWORD';
    ```
    
    Example (with a user "mariadb" and password "mariadb")
    ```
    $cfg['Servers'][$i]['auth_type'] = 'config';
    $cfg['Servers'][$i]['user'] = 'mariadb';
    $cfg['Servers'][$i]['password'] = 'mariadb';
    ```
- Save

Revert to the original permissions
```sh
sudo chmod 644 /etc/phpmyadmin/config.inc.php
```

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

Restart WSL by running the following command in PowerShell
```powershell
wsl --shutdown
```
With these aliases, you can use `lampstatus`, `lampstart`, `lampstop` and `lamprestart`.

# Additional stuff (Git, NodeJS, Composer, Symfony CLI)

## Git
Installation
```sh
sudo apt install git
git config --global user.name "Your Name"
git config --global user.email "Your Email"
git config --global init.defaultBranch main
git config --global color.ui auto
git config --global pull.rebase false
```

Generate SSH key (replace `<YourEmail>`)
```sh
ssh-keygen -t ed25519 -C <YourEmail>
```

To get the SSH key
```sh
cat ~/.ssh/id_ed25519.pub
```

## NodeJS
For more information, https://github.com/nvm-sh/nvm

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
```
Then
```sh
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

Install LTS
```sh
nvm install --lts
```

Use LTS (as of this date, LTS is 18.17.0)
```sh
nvm use 18.17.0
```

## Composer
For more information, https://getcomposer.org/download/
```sh
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e21205b207c3ff031906575712edab6f13eb0b361f2085f1f1237b7126d785e826a450292b6cfd1d64d92e6563bbde02') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

## Symfony CLI
For more information, https://symfony.com/download
```sh
curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
sudo apt install symfony-cli
```

# Backup WSL

You can backup your WSL setup with `PowerShell` by running (replace `<username>`)
```powershell
wsl --export Debian C:\Users\<username>\Documents\WSL_Backups\debian_backup.tar
```