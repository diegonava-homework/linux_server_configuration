# Linux Server Configuration

### Project Overview
> A baseline installation of a Linux server and preparing it to host our web applications, 
securing it from a number of attack vectors, installing and configuring a database server, and deploying 
our one of your existing web applications onto it.


#### Link to Project: [ItemCatalog](http://163.172.136.244/)

* **Public IP Address:** 163.172.136.244
* **Accessible SSH port:** 2200

The project don't have passphrase
The Private key file is attached on the zip file

## Steps to Configure Linux server
#### 1. Launch VM and access SSH to the instance 
  * Create machine in [Scaleway](https://scaleway.com) and use ssh to connect

   
#### 2. Create New User
  
  * Add User grader
  
  ```
    $ sudo adduser grader
  ```
  * Give Sudo Access without password to grader 
  
  ```
    $ sudo nano /etc/sudoers.d/grader
  ```
  * Add following line to this file for 
  
  ```
    grader ALL=(ALL:ALL) NOPASSWD:ALL
  ```

#### 3. Configure the key-based authentication for grader user
  *  Generate an encryption key 
  In this example, key is made on remote server, but optionally can be done on local server and copy to remote authorized_keys the public key
  
   i. Login to VM with root, and login as grader:
   
   ```
    $ su grader
    $ ssh-keygen -t rsa
   ```
      followed by the name of the key. By default, the keys will be stored in the ~/.ssh directory within your user's 
      home directory.
      
   ii. Get the content of public key:
   
   ```
    $ cat ~/.ssh/id_rsa.pub
   ```
   
   
   iii. Copy the content to authorized_keys
   
   ```
    $ cat ~/.ssh/id_rsa.pub
   ```
   
   iiii. Get the contents of ~/.ssh/id_rsa.pub and ~/.ssh/id_rsa  and copy locally for use to connect, in this example: *example.pub* and *example.key*
  
   ```
    $ cat ~/.ssh/id_rsa.pub 
    $ cat ~/.ssh/id_rsa
   
    # Locally paste the content of files (example.pub, example.key)
   ```
   
   iiiii. Exit grader for return to root
   
   ```
    $ exit
   ```


#### 4. Enforce key-based authentication (user root)
	
  * Run `$ nano /etc/ssh/sshd_config`.
  * Find the **PasswordAuthentication** line and edit it to no.
  * Save the file.
  * Run `$ service ssh restart` to restart the service.

#### 5. Change the SSH port from 22 to 2200
  * Find the **Port line** in the same file above, i.e */etc/ssh/sshd_config* and edit it to 2200.
  * Save the file.
  * Run `$ service ssh restart` to restart the service.
  
#### 6. Disable login for root user
  * Find the **PermitRootLogin** line in the same file above, i.e */etc/ssh/sshd_config* and edit it to no.
  * Save the file.
  * Run `$ service ssh restart` to restart the service.

Now we can login into remote VM through SSH with following command
```
 $ ssh -i example.key grader@XX.XX.XX.XX -p 2200
```

Source: [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1739013)

#### 7. Configure local timezone to UTC 
  * Change the timezone to UTC using following command: `$ sudo timedatectl set-timezone UTC`.
  * You can also open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.
  * Install ntp daemon ntpd for a better synchronization of the server's time over the network connection:
  
  ```
   $ sudo apt-get install ntp
  ```
 Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)

#### 8. Update all currently installed packages
  * `$ sudo apt-get update`.
  * `$ sudo apt-get upgrade`.

#### 9. Configure the Uncomplicated Firewall (UFW)
  ```
   $ sudo ufw default deny incoming
   $ sudo ufw default allow outgoing
   $ sudo ufw allow 2200/tcp
   $ sudo ufw allow www
   $ sudo ufw allow ntp
   $ sudo ufw enable
  ```

#### 10. Configure cron scripts to automatically manage package updates
  * Install unattended-upgrades if not already installed using command:
  
  ```
   $ sudo apt-get install unattended-upgrades
  ```
  * Enable it using command:
  
  ```
   $ sudo dpkg-reconfigure --priority=low unattended-upgrades
  ```

#### 11. Install and Configure Apache2, mod-wsgi and Git
 ```
  $ sudo apt-get install apache2 libapache2-mod-wsgi git
 ```
 * Enable mod_wsgi:
 
 ```
  $ sudo a2enmod wsgi
 ```

#### 12. Install and configure PostgreSQL
  * Installing PostgreSQL Python dependencies:
  
  ```
   $ sudo apt-get install libpq-dev python-dev
  ```
  * Installing PostgreSQL:

   ```
     $ sudo apt-get install postgresql postgresql-contrib
   ```
  * Check if no remote connections are allowed :

   ```
     $ sudo cat /etc/postgresql/9.3/main/pg_hba.conf
   ```
   
  * Login as *postgres* User (Default User), and get into PostgreSQL shell:

   ```
     $ sudo su - postgres
     $ psql
   ```
    * Create a new User named *catalog*:  `# CREATE USER catalog WITH PASSWORD 'catalog';`
    * Create a new DB named *catalog*: `# CREATE DATABASE catalog WITH OWNER catalog;`
    * Connect to the database *catalog* : `# \c catalog`
    * Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`
    * Lock down the permissions only to user *catalog*: `# GRANT ALL ON SCHEMA public TO catalog;`
    * Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`

  * Inside the Flask application, the database connection is now performed with:
   
   ```
   engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
   ```

#### 13. Install Flask and other dependencies

```
    $ sudo apt-get install python-pip
    $ sudo pip install Flask
    $ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
    $ sudo pip install requests
  ```
Source: [Flask Documentation](http://flask.pocoo.org/docs/0.12/installation/)

#### 14. Clone the Catalog app from Github

  * Make a *catalog* named directory in */var/www*

    ```
      $ sudo mkdir /var/www/catalog
    ```

  * Change the owner of the directory *catalog*

    ```
     $ sudo chown -R grader:grader /var/www/catalog
    ```

  * Clone the **project** to the catalog directory:

    ```
     $ cd /var/www 
     $ git clone https://github.com/diegonava6/item_catalog catalog
    ```


  * Make a main.wsgi file to serve the application over the mod_wsgi. with content:

    ```
     $ touch main.wsgi && nano main.wsgi
    ```

    ```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from main import app as application
    ```
  * Inside *main.py*, *database_setup.py* and *dummyBooks.py*  database connection is now performed with:

    ```
     engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
    ```
  * Run the database_setup.py and dummyBooks.py once to setup database with dummy data:
  
  ```
   $ python database_setup.py
   $ python dummybooks.py
  ```

#### 15. Edit the default Virtual File with following content:

  ```
    $  sudo nano /etc/apache2/sites-available/000-default.conf
  ```


  ```
  <VirtualHost *:80>
    ServerAdmin example@gmail.com
    WSGIScriptAlias / /var/www/catalog/main.wsgi
    <Directory /var/www/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/static
    <Directory /var/www/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
  </VirtualHost>
  ```
Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)

#### 16. Restart Apache to launch the app

   ```
    $ sudo service apache2 restart
   ```










