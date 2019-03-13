# bowls101-deployed-linux
Author: Perla Jarillo

Describe the Linux Server Configuration to deploy bowls101


## IP address and ports.
The static ip address for the Linux Ubuntu Server is 34.192.248.53. The firewall has been configured to only allow for connections for

- SSH port 2200
- HTTP port 80
- NTP port 123

## The complete URL for the web application.
As new features will be included in this app, a domain was acquired at [Google Domains - Google](https://domains.google/#/). Then, a DNS Zone was added in the Amazon Lightsail and configured to associate the domain with the server's IP address. The complete URL for the web application is as follow:

http://mostlyplants101.com/

## A summary of software you installed and configuration changes made.
The first step was creating an Ubuntu Linux instance in  [Amazon Lightsail](https://lightsail.aws.amazon.com). Then connected using ssh to install packages and configure services in the terminal.

### Updating packages
To be sure that the server us up-to-date, all packages has been updated using this commands:

`sudo apt-get update`
`sudo apt-get upgrade`
`sudo do-release-upgrade`

To update packages kept back:
`sudo apt-get update && sudo apt-get dist-upgrade`

Then perform `sudo reboot` to apply changes.

### Packages included
This is the list of software installed in the server using the command sudo `apt-get install` :

- apache2
- libapache2-mod-wsgi
- postgresql
- python-pip
- git

## Python dependencies

The web app requires the follow dependencies to work properly:

- httplib2. It is HTTP client library
  `sudo pip install httplib2`

- oauth2client. Makes it easy to interact with OAuth2-protected resources
  `sudo pip install oauth2client`
  
- Flask. It is a microframework for Python.
  `sudo pip install Flask`
  
- Flask-HTTPAuth. It is a simple extension that simplifies the use of HTTP authentication with Flask routes.
  `sudo pip install flask_httpauth`

- SQLAlchemy. It is the Python SQL toolkit and Object Relational Mapper.
  `sudo pip install sqlalchemy`
  
- requests. Allows to send organic, grass-fed HTTP/1.1 requests, without the need for manual labor.

## Configurations
### Firewall 
The ubuntu server firewall was configured following this steps:
1.  Denying incoming requests
`sudo ufw default deny incoming`

2.  Allowing outcoming requests
`sudo ufw default allow incoming`

3. Alowing protocols and ports
Normaly you will follow this sintaxis:
`sudo ufw allow protocol/port`

### Apache server
Once Apache was installed, the /etc/apache2/sites-enabled/000-default.conf file was edited and the next lines were included before the "tag" `</VirtualHost>`:

`       ServerName 34.192.248.53
        ServerAdmin admin@34.192.248.53
        WSGIScriptAlias / /var/www/html/myapp.wsgi `
        ``<Directory /var/www/html/bowls101/>
                        Order allow,deny
                        Allow from all
        </Directory>``
        `Alias /static /var/www/html/bowls101/static`
        ``<Directory /var/www/html/bowls101/static/>
                        Order allow,deny
                        Allow from all
        </Directory>``
       ` Alias /templates/var/www/html/bowls101/templates`
       `` <Directory /var/www/html/bowls101/templates/>
                        Order allow,deny
                        Allow from all
        </Directory>
``
This allows to enable the virtual host by specifying the server name, server administrator contact, name of the script wsgi and directories that contain the app resources.

### Database
In the development version of the app, it was being used sqlite as an "embedded" database to run de app server-less. However, PostgreSQL works based on a client-server model and provides more features. Having a Linux server it makes much more sense to use PostgreSQL.

After installing PostgreSQL I logged into the database server using

`sudo -u postgres psql postgres`

Then I created the database:

`CREATE DATABASE bowls101`

I also changed the default password of postgres using the `\password` command.

The rest of the work was adapt the database model, storage in the file model.py, as well as the populate_bowls101db.py file.

I substituted the line: 

`engine = create_engine('sqlite:///bowls101.db')`

For the next:

`` db_string = "postgres://user:admindb@localhost/bowls101" ``
` engine = create_engine(db_string)`

### Web application

The application's principal script was storage under the name of bowls101.py. To work with the wsgi application I changed the name of that file to __init__.py

Inside this file was necesary to make some adjustments for the production version. For example, the database string was changed as follow:

`` db_string = "postgres://user:admindb@localhost/bowls101" ``
` engine = create_engine(db_string)`

Also, as we are using a diferent way to locate the resources of the app, we need to provide a full path for the client_secrets.json. All the lines making reference to this resource changed like this:

``/var/www/html/bowls101/client_secrets.json'``

Finally, the main's conditional changed to:

``if __name__ == '__main__': 
  app.run(threaded=False)``

The wsgi includes the app secret key.

## Grader User
The grader user was created and included into the sudoers directory. Also, a private key was generated using the ssh-keygen tool and saved inside the file:

`/home/grader/.ssh/authorized_keys`

The permissions to the .ssh were changed: ` sudo chmod 700 /.ssh`
As well as the authorized_keys: `sudo chmod 644 /.ssh/authorized_keys`

The user can't use a password to login into it's session.

## Third-party resources.

The web app uses the Google Sign. You can find more information in [Google Identity Platform  \|  Google Developers](https://developers.google.com/identity/) 

The domain was obtained in [Google Domains - Google](https://domains.google/#/)

The Linux instance was created using AWS [Amazon Lightsail](https://lightsail.aws.amazon.com)

## Information sources:

[SQLite vs PostgreSQL - Which database to use and why? \| TablePlus](https://tableplus.io/blog/2018/08/sqlite-vs-postgresql-which-database-to-use-and-why.html)

[Google Identity Platform  \|  Google Developers](https://developers.google.com/identity/)

[How To Deploy a Flask Application on an Ubuntu VPS \| DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

[Creating a DNS zone to manage your domain’s DNS records in Amazon Lightsail \| Lightsail Documentation](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-create-dns-entry)

[Understanding public network ports and firewall settings in Amazon Lightsail \| Lightsail Documentation](https://lightsail.aws.amazon.com/ls/docs/en/articles/understanding-firewall-and-port-mappings-in-amazon-lightsail)

[Ubuntu Server message says packages can be updated, but apt-get does not update any - Server Fault](https://serverfault.com/questions/265410/ubuntu-server-message-says-packages-can-be-updated-but-apt-get-does-not-update)

[Getting Started with UFW (Uncomplicated Firewall) on Ubuntu 15.04](https://www.howtoforge.com/tutorial/ufw-uncomplicated-firewall-on-ubuntu-15-04/)


