# How install Wordpres with LAMP on Ubuntu 20.04

### 1 Create a Mysql database for Wordpress
access your Mysql in root acount with this command:
```bash
mysql -u root -p
```

insed of Mysql create a database for Wordpress
```bash
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

create a user and grant privileges for him
```bash
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY '123456';
```

```bash
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';
```

update Mysql privileges
```bash
FLUSH PRIVILEGES;
```

exit from Mysql type
```bash 
EXIT;
```

### 2 Install PHP extensions
update your repositories and install the extensions
```bash
sudo apt update
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

Now restart Apache for apply the new extensions
```bash
sudo systemctl restart apache2
```

### 3 Adjusting Apache config for .htaccess consider Substitutions and Rewrites
Open your site apache config file
```bash
sudo nano /etc/apache2/sites-available/your_domain.conf
```

To allow .htaccess files, we need to set the AllowOverride directive inside a Directory block pointing to our document root. Add the following block of text inside the VirtualHost block to your configuration file, making sure to use the correct web root directory:
```html
<Directory /var/www/wordpress/>
    AllowOverride All
</Directory>
```
after insert save and exit.

Enabling the Rewrite module
Then we can enable mod_rewrite so we can use WordPress' permalink (or permanent link) feature:
```bash
sudo a2enmod rewrite
```
Enabling changes
Before implementing the changes we've made, check to make sure we don't make syntax errors:
```bash
sudo apache2ctl configtest
```
The result may have a message that looks like this:
```bash
Output
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK
If you want to suppress the top line, just add a ServerName directive to your main (global) apache configuration file in Apache in /etc/apache2/apache2.conf. ServerName can be your server's domain or IP address. However, this is a message only and does not affect the functioning of our website. As long as the output shows Syntax OK, you are ready to continue.
```
Restart Apache to implement the changes:
```bash
sudo systemctl restart apache2
```

### 4 Download Wordpres
Change to a writable directory and download the zipped version file by typing:
```bash
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
```
Extract the compressed file to create the WordPress directory structure:
```bash
tar xzvf latest.tar.gz
```

Create .htaccess archive:
```bash
touch /tmp/wordpress/.htaccess
```

We'll also copy the example configuration file over the filename so that in practice WordPress will read the following:
```bash
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```

We can also create the upgrade directory so that WordPress doesn't run into permission issues when trying to do this on its own after an update to your software:
```bash
mkdir /tmp/wordpress/wp-content/upgrade
```

Now we can copy the entire contents of the directory to our document root. We're using a period at the end of our source directory to indicate that everything inside the directory should be copied, including hidden files (like the .htaccess file we created):
```bash
sudo cp -a /tmp/wordpress/. /var/www/your_domain
```

### 5 Config Wordpress directory
Adjusting Ownership and Permissions typing this commands:
```bash
sudo chown -R www-data:www-data /var/www/your_domain
sudo find /var/www/your_domain/ -type d -exec chmod 750 {} \;
sudo find /var/www/your_domain/ -type f -exec chmod 640 {} \;
```

### 6 Setting up the WordPress configuration file (wp-config.php)

We need to modify some of the database connection settings at the beginning of the file. You need to set the database name, database user and associated password that we set up in MySQL.

The other change we need to make is to define the method that WordPress should use to write to the file system. Once we've given the web server permission to write where it needs to, we can explicitly set the file system method to “direct”. Failing to define this access method - using our current settings, would cause WordPress to ask for FTP credentials when we perform some actions.

This setting can be added below the database connection settings or elsewhere in the file:

```bash
sudo nano /var/www/wordpress/wp-config.php
```

```php
. . .

define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', '123456');

. . .

define('FS_METHOD', 'direct');
```

### 6 Completing the Installation via the Web Interface
In your web browser, navigate to your server's domain name or public IP address:

https://server_domain_or_IP
