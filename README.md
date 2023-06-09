# Install Nextcloud with Nginx and PHP-FPM on Fedora

## Prepare your virtual machine
Follow the guide here to prepare your Fedora Server VM for Nginx hosting.
https://github.com/endangeredtoast/howto/blob/main/prepare-fedora-38-server-for-webhosting-with-nginx.md

## Set up Database with phpMyAdmin
Set root password.
```
sudo mysql_secure_installation
```
Login to phpMyAdmin at http://yourserverip/phpMyAdmin with the password you set earlier.
In the left sidebar, click on New. Enter a database name and choose utf8mb4_unicode_ci, then click Create.
On the top navigation bar, click on Privileges. Click on Add User Account.
Enter a username and password for the new account. Scroll down and click on Go. The database and user is now created.

## Install Nextcloud
### Install Dependencies
Run the extension install script. You can just press enter at the various prompts during installation.
```
sudo ./install-php-extensions.php
```

### Install Nextcloud
Run the Nextcloud installer script.
```
sudo ./install-nextcloud.sh
```

## Tuning
### Install OPCache
Install OPCache with dnf
```
sudo dnf install php-opcache
```
Enable OPCache in php.ini
```
sudo nano /etc/php.ini
```
Use CTRL+W to search for opcache. Remove the semicolon before 'see /etc/php.d/10-opcache.ini'
```
sudo nano /etc/php.d/10-opcache.ini
```
Make sure the following lines are present.
```
opcache.enable=1
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.memory_consumption=1024
opcache.save_comments=1
opcache.revalidate_freq=1
```

### Tune PHP-FPM
Change the following lines in '/etc/php-fpm.d/www.conf'
```
pm = dynamic
pm.max_children = 120
pm.start_servers = 12
pm.min_spare_servers = 6
pm.max_spare_servers = 18
```


### Activate Redis and Memory Cache
Edit the '/config/config.php' file in your nextcloud directory.
```
sudo nano /config/config.php
```
Add the following lines:
```
  'default_phone_region' => 'CA',
  'memcache.local' => '\OC\Memcache\APCu',
  'memcache.distributed' => '\OC\Memcache\Redis',
  'filelocking.enabled' => true,
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' => array(
       'host' => 'localhost',
       'port' => 6379,
       'timeout' => 0.0,
      ),
```


### Enable System Variables
Enable the following lines in the '/etc/php-fpm.d/www.conf' file (remove the semicolon at the front)
```
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```



