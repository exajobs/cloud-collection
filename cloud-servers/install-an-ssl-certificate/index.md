---
permalink: install-an-ssl-certificate
audit_date: '2022-06-16'
title: Install an SSL certificate
type: article
created_date: '2022-06-16'
created_by: Alberto Blanquel
last_modified_date: '2022-06-16'
last_modified_by: Miguel Salgado
product: Cloud Servers
product_url: cloud-servers
---

After you [generate a certificate signing request (CSR)](/support/how-to/generate-a-csr)
and [purchase or renew a Secure Socket Layer (SSL) certificate](/support/how-to/purchase-or-renew-an-ssl-certificate/),
you'll need to install it. This article shows you how to install an SSL
certificate on various servers and operating systems.

If you want to install an SSL certificate on a load balancer, see
[Configure SSL certificates on Cloud Load Balancers](/support/how-to/configure-SSL-certificates-on-cloud-load-balancers).

The following sections provide instructions for the installation process according to each platform:

- [Prerequisites](#prerequisites)

- [Install certificate on Microsoft&reg; Windows&reg; 2008 R2 and 2012 servers](#install-certificate-on-windows-servers)

- [Install certificate in Apache Web Server](#install-certificate-in-apache-web-server)

- [Install certificate in NGINX with php-fpm](#install-certificate-in-nginx-with-php-fpm)

- [Install certificate on Managed Hosting solutions](#install-certificate-on-managed-hosting-solutions)

- [Install certificate on a custom Microsoft&reg; Azure&reg; domain](#install-certificate-on-a-custom-microsoft-azure-domain)

- [Test the certificate](#test-the-certificate)

After you have installed your certificate, you should reload your web server
service.

### Prerequisites

Before you install your certificate, make sure that you have the following items:

- A certificate from your preferred SSL vendor stored on your server. If you
don't already have a certificate, see
[Generate a CSR](/support/how-to/generate-a-csr/) and
[Purchase or renew an SSL certificate](/support/how-to/purchase-or-renew-an-ssl-certificate/)
for instructions.

- The Certificate Authority (CA) bundle with the root and intermediate
certificates provided by the SSL vendor.
- The **.key** file that was generated when you created the CSR.
- An installed web server such as Apache or NGINX.
- An Internet Protocol (IP) address for your SSL certificate.

#### Copy the files into the default location on your server

A vendor-provided SSL certificate contains three components: the SSL certificate,
the CA file, and the SSL key. When you receive your SSL
certificate from your CA, upload it to your server by using the following
steps:

1. Copy all the contents of the certificate, including the `BEGIN CERTIFICATE`
and `END CERTIFICATE` lines. Save the copied text as `domain.com.crt`.

2. Copy the certificate and private key into the server directory
in which you plan to store your certs. For example, the default Apache
directories are: `/usr/local/apache/conf/ssl.crt/` or
`/etc/httpd/conf/ssl.crt/`.

### Install certificate on Windows servers

The following sections show you how to install and bind an SSL certificate on
Windows servers by using the Internet Information Services (IIS) Manager.

#### Install the certificate

Prerequisite: You should already have the certificate provided by your
preferred SSL vendor.

If you got your CSR by using anything other than IIS, skip to
[Import an SSL certificate from another server](#import-an-ssl-certificate-from-another-server).

Use the following steps if you got your CSR by using IIS, which pairs the public
key from your vendor with the private key generated by IIS.

1. In the IIS Manager, select the server and double-click **Server Certificates**.
2. Under **Actions**, click **Complete Certificate Request**.
3. In the wizard, select the location of the certificate file provided by your
SSL vendor.
4. For *Windows Server&reg; 2012 only*, name the file and choose your storage
location.
5. Click **OK**.

#### Import an SSL certificate from another server

1. In the IIS Manager, double-click **Server Certificates**.
2. Under **Actions**, click **Import**.
3. Select the location of your certificate file, enter the password (if you set
one), and choose your certificate storage location (*Windows Server 2012 only*).
4. Click **OK**.

#### Set up the bindings

1. In the IIS Manager, right-click your site and select **Edit Bindings**.
2. In the **Site Bindings** window, click **Add**.
3. In the **Add Site Binding** dialog box, perform the following steps:
    a. Set the value of **Type** to **https**.
    b. For *Windows Server 2012 only*, specify the host name if necessary.
    c. From the **SSL certificate** list, select your certificate.
    d. Click **OK**.

After you set up the bindings, the **Site Bindings** window shows the binding for
HTTPS.

### Install certificate in Apache Web Server
The following sections show you how to install and bind SSL certificate on Apache web servers 

**NOTE:** Be aware that the commands are run by the root user, if you do not have access to the root user but you have sudo permissions, use the sudo command at the beginning of each line.

#### Prerequisite
The first step in adding a vhost is to make sure the server has apache installed and it is running, This can be done with the following commands. (If you already know that the Apache is up and running you can skip this step).

##### RHEL 6/CentOS 6
```sh
$ service httpd status
```

##### RHEL 7/CentOS 7+
```sh
$ systemctl status httpd  
```

##### Debian 8+/Ubuntu 16+
```sh
$ systemctl status apache2
```

At this point, if Apache is not installed, you will receive an error that the process cannot be found. You can check if the server is running Apache with the following command:
```sh
$ netstat -plnt | awk '$4 ~ /:(80|443)$/'
```

If you find the server is not running Apache, this is not the correct option for you. Please try another process listed in this article.

#### Check that the SSL Apache module is available

**mod_ssl** is an optional module for the Apache HTTP Server. It provides strong cryptography for the Apache webserver. Without this the Apache webserver will not be able to utilise or provide SSL encryption.

To check if the SSL module has been enabled run the following command in order to what OS are you using:

##### RHEL / CentOS
```sh 
$ httpd -M | grep ssl
```

If no output is received the mod_ssl (shared) requires installation to serve SSL traffic.

Run the following command to install mod_ssl:

```sh
# For RHEL 7/ CentOS 7
$ yum install mod_ssl

# For RHEL 8/ CentOS 8 / Alma Linux / Rocky Linux.
$ dnf install mod_ssl
```

To check if the SSL module has been enabled run the following command:
```sh
$ apachectl -M | grep ssl
 ssl_module (shared)
```

##### Debian/Ubuntu
To check if the SSL module has been enabled run the following command:
```sh
$ apachectl -M | grep ssl
 ssl_module (shared)
```

If no output is shown, the **mod_ssl (shared)** requires enabling to serve SSL traffic.
Run the following command to enable **mod_ssl**
```sh
$ a2enmod ssl
```
Once mod_ssl has been installed, Apache will require a restart to enable. Run the appropriate distribution command to restart Apache, you can find it int the reload or restar web server section.

#### Install and secure SSL Components on the server
Now that you have the **SSL Certificate**, **Private key** and **CA Bundle** you are ready to add these certificates to the server. The best practices dictates that the certificates are named (expirationYear-domain, eg: 2022-example.com.crt) and stored in the following locations.

##### RHEL/CentOS
Using your preferred text editor, create new files at the locations provided below. Be sure to change the example.com to the name of the domain on the server.
```txt
SSL Certificate - /etc/pki/tls/certs/2022-example.com.crt
SSL CA Bundle  - /etc/pki/tls/certs/2022-example.com.CA.crt
SSL Private Key - /etc/pki/tls/private/2022-example.com.key
```

##### Debian/Ubuntu
Using your preferred text editor, create new files at the locations provided below. Be sure to change the example.com to the name of the domain on the server.

SSL Certificate - /etc/ssl/certs/2022-example.com.crt
SSL CA Bundle  - /etc/ssl/certs/2022-example.com.CA.crt
SSL Private Key - /etc/ssl/private/2022-example.com.key

**NOTE:** When a private key is installed with world-readable permissions, it allows anyone with access to even a user account on the server to decipher any information encrypted with its corresponding certificate.   It is important to ensure that the certificate and key have the correct permissions.

Once the SSL certificate components hace been installed at the proper locations, it is important that we set the correct permissions for the Private Key file.

``` sh
$ chmod 600 /path/to/private/key/file.key
``` 

#### Create or modify VirtualHost file
##### RHEL/CentOS
Virtual host files are what specify the configuration of our separate sites and dictate how the Apache web server will respond to various domain requests.

Rackspace best practice dictates that the virtual host is located in the following locations:
- ***/etc/httpd/conf.d/example.com.conf***
    
Alternative locations for Apache Virtual host can be:
- ***/etc/httpd/conf/httpd.conf*** (default apache configuration file, also utilized extensively by Webmin)
- ***/etc/httpd/conf.d/ssl.conf*** (global default configuration file for SSL)
- ***/etc/httpd/vhost.d/example.com.conf*** (vhost.d is a user created directory and although reasonably common it is not the best practice)
    
**NOTE:**: Due to the configurations that we have outlined, all virtual host files must end in .conf. For the purposes of this example, we will assume a virtual host for port 80 already exists in its own config file.

Start by opening the virtual host file in your preferred text editor:
```sh
$ vim /etc/httpd/conf.d/example.com.conf
```

If the port 80 Virtual Host was configured by Rackspace, in most cases, a dummy 443 block would have been created as a placeholder. If so, you can remove the comment character at the start of each line and modify the section as needed.

There are a few things to verify:
- ServerName/ServerAlias matches for your domain
- The DocumentRoot matches the the port 80 block
- The Directory section matches the port 80 block

Place the following in your virtual host for the block 443:
```apache
SSLEngine on
SSLCertificateFile      /etc/pki/tls/certs/2022-example.com.crt
SSLCertificateChainFile /etc/pki/tls/certs/2022-example.com.CA.crt
SSLCertificateKeyFile   /etc/pki/tls/private/2022-example.com.key
```

##### Debian/Ubuntu
Virtual host files are what specify the configuration of our separate sites and dictate how the Apache web server will respond to various domain requests.

Rackspace best practice dictates that the virtual host is located in the following locations:
- ***/etc/apache2/sites-available/example.com.conf***

This directory is the location where the config file is stored. It is then symlinked to ***/etc/apache2/sites-enabled/example.com.conf***

**NOTE**: Due to the configurations that we have outlined, all virtual host files must end in .conf. For the purposes of this example, we will assume a virtual host for port 80 already exists in its own config file.

Start by opening the virtual host file in your preferred text editor:
```sh
$ vim /etc/apache2/sites-available/example.com.conf
```

If the port 80 Virtual Host was configured by Rackspace, in most cases, a dummy 443 block would have been created as a placeholder. If so, you can remove the comment character at the start of each line and modify the section as needed.

There are a few things to verify:
- ServerName/ServerAlias matches for your domain
- The DocumentRoot matches the the port 80 block
- The Directory section matches the port 80 block

Place the following in your virtual host for the block 443:
```apache
<VirtualHost *:443>
        ServerName      example.com
        ServerAlias     www.example.com
        DocumentRoot    /var/www/vhosts/example.com
        <Directory /var/www/vhosts/example.com>
                Options -Indexes +FollowSymLinks -MultiViews
                AllowOverride All
        </Directory>

        CustomLog   /var/log/apache2/example.com-access.log combined
        ErrorLog    /var/log/apache2/example.com-error.log

        SSLEngine on
        SSLCertificateKeyFile   /etc/ssl/private/2022-example.com.key
        SSLCertificateFile      /etc/ssl/certs/2022-example.com.crt
        SSLCertificateChainFile /etc/ssl/certs/2022-example.com.ca.crt
</VirtualHost>
```

#### Check syntax and enable virtual host in Apache
Now that the Document Root and Virtual Host config files have been created, we can check the syntax of the config files and reload Apache.

**WARNING**: The following steps can lead to breaking the Apache process. Make sure that these steps are followed exactly and that you verify your work after making the changes. If the changes you make result in Apache process stopping, this means the domains on the server are now down. Revert any changes made an ensure the Apache process is running or **Call to Rackspace Global Support**

##### RHEL/CentOS 6

In order to verify the syntax of the configuration files are correct, you will need to run the following command:
```sh
$ httpd -t
Syntax OK
```
If everything checks out, you will get a message that indicates **Syntax OK**.

If you do not see this message, it indicates there may be a syntax issue in your Apache config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message.

Once the syntax check returns OK, run the following command to reload Apache:
```sh
$ apachectl graceful
```

Once Apache has been reloaded, remember to check that Apache is running as expected as described in Prerequisites section.

##### RHEL/CentOS 7

In order to verify the syntax of the configuration files are correct, you will need to run the following command:
```sh
$ httpd -t
Syntax OK
```
If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your Apache config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message.

Once the syntax check returns OK, run the following command to reload Apache:
```sh
$ apachectl graceful
```

Once Apache has been reloaded, remember to check that Apache is running as expected as described in Prerequisites section.

##### Debian/Ubuntu

In order to verify the syntax of the configuration files are correct, you will need to run the following command:
```sh
$ apachectl -t
```

If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your Apache config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message

Once the syntax check returns OK, run the following command to reload Apache:
```sh
$ apachectl graceful
```

Once Apache has been reloaded, remember to check that Apache is running as expected as described in Prerequisites section.

### Install certificate in Nginx with PHP-FPM
The following sections show you how to install and bind SSL certificate on NGINX web servers 

**NOTE:** Be aware that the commands are run by the root user, if you do not have access to the root user but you have sudo permissions, use the sudo command at the beginning of each line.

#### Prerequisite
The first step in adding a server block is to make sure the server has Nginx installed and it is running. This can be done with the following commands.

##### RHEL/CentOS 6
```sh
$ service nginx status
```
##### RHEL 7+/CentOS 7+/Debian 8+/Ubuntu 16+
```sh
$ systemctl status nginx
```

At this point, if NGINX is not installed, you will receive an error that the process cannot be found. You can check if the server is running  Nginx with the following command: 
```sh
$ netstat -plnt | awk '$4 ~ /:(80|443)$/'
```

If you find the server is not running NGINX, this is not the correct procces. Pleas try another process that is describe in this article.

#### Install and secure SSL components on the server
Now that you have the **SSL Certificate**, **Private key** and **CA Bundle** you are ready to add these certificates to the server. The best practices dictates that the certificates are named (expirationYear-domain, eg: 2022-example.com.crt) and stored in the following locations.

##### RHEL/CentOS
NGINX only utilizes two certificate files for each server block. The CA Bundle and Certificate files are combined into a single file. There are two ways to go about setting up the chained certificate for use with Nginx.

Using your preferred text editor, create new files at the locations provided below. Be sure to change the **example. com** to the name of the domain on the server.

The first method is simply adding both the CRT and the CA Bundle contents to a single file in the proper certificate location. This is shown in the example below:
- SSL Certificate/Chain - /etc/pki/tls/certs/2022-example.com.chained.crt
- SSL Private Key - /etc/pki/tls/private/2022-example.com.key
    
The second method includes an additional step in which you add the CA bundle as it's own file to the server and then concatenate the files into a new file. This is shown in the example below.
```text
SSL Certificate - /etc/pki/tls/certs/2022-example.com.crt
SSL CA Bundle  - /etc/pki/tls/certs/2022-example.com.CA.crt
SSL Private Key - /etc/pki/tls/private/2022-example.com.key
```

Once the files have been created, you can run the following command:
```sh
cat /etc/pki/tls/certs/2022-example.com.crt /etc/pki/tls/certs/2022-example.com.CA.crt > /etc/pki/tls/certs/2022-example.com.chained.crt
```

##### Debian/Ubuntu
NGINX only utilizes two certificate files for each server block. The CA Bundle and Certificate files are combined into a single file. There are two ways to go about setting up the chained certificate for use with Nginx.

Using your preferred text editor, create new files at the locations provided below. Be sure to change the example.com to the name of the domain on the server.

The first method is simply adding both the CRT and the CA Bundle contents to a single file in the proper certificate location. This is shown in the example below:

SSL Certificate/Chain - /etc/ssl/certs/2022-example.com.chained.crt
SSL Private Key - /etc/ssl/private/2022-example.com.key

The second method includes an additional step in which you add the CA bundle as it's own file to the server and then concatenate the files into a new file. This is shown in the example below.
```text
SSL Certificate - /etc/ssl/certs/2022-example.com.crt
SSL CA Bundle  - /etc/ssl/certs/2022-example.com.CA.crt
SSL Private Key - /etc/ssl/private/2022-example.com.key
```

Once the files have been created, you can run the following command:
```sh
cat /etc/ssl/certs/2022-example.com.crt /etc/ssl/certs/2022-example.com.CA.crt > /etc/ssl/certs/2022-example.com.chained.crt
```

**NOTE:** When a private key is installed with world-readable permissions, it allows anyone with access to even a user account on the server to decipher any information encrypted with its corresponding certificate. It is important to ensure that the certificate and key have the correct permissions.

Once the SSL certificate components have been installed at the proper locations, it is important that we set the correct permissions for the Private Key file. 
```sh
    chmod 600 /path/to/private/key/file.key
```

#### Create of modify Server Blocks in Virtual Host

##### RHEL/CentOS
Virtual host files are what specify the configuration of our separate sites and dictate how the Nginx web server will respond to various domain requests.

Rackspace best practice dictates that the virtual host is located in the following locations:
- ***/etc/nginx/conf.d/example.com.conf***

Alternative locations for Nginx Virtual host can be..
- ***/etc/nginx/conf/httpd.conf*** (default apache configuration file, also utilized extensively by Webmin)
- ***/etc/nginx/conf.d/ssl.conf*** (global defaullt configuration file for SSL)
- ***/etc/nginx/vhost.d/example.com.conf*** (vhost.d is a user created directory and although reasonably common it is not Rackspace best practice)

**NOTE**: Due to the configurations that we have outlined, all virtual host files must end in .conf. For the purposes of this example, we will assume a virtual host for port 80 already exists in its own config file.**

Start by opening the virtual host file in your preferred text editor:
```sh
$ vim /etc/nginx/conf.d/example.com.conf
```

There are a few things to verify:
- ServerName/ServerAlias matches for your domain
- The DocumentRoot matches the the port 80 block
- The Directory section matches the port 80 block

You will need to change the example.com for your website name and place it in your virtual host:

```nginx
server {
    listen              443;

    server_name         example.com www.example.com;
    root                /var/www/vhosts/example.com;
    index               index.html;

    ssl                 on;
    ssl_certificate     /etc/pki/tls/certs/2022-example.com.chained.crt;
    ssl_certificate_key /etc/pki/tls/private/2022-example.com.key;

    access_log          /var/log/nginx/example.com_ssl_access.log main;
    error_log           /var/log/nginx/example.com_ssl_error.log;
}
```

Once the changes have been made to the appropriate config files, save and quit the file.

##### Debian/Ubuntu
Virtual host files are what specify the configuration of our separate sites and dictate how the Nginx web server will respond to various domain requests.

Rackspace best practice dictates that the virtual host is located in the following locations:
- ***/etc/nginx/sites-available/example.com.conf***

This directory is the location where the config file is stored. It is then symlinked to **/etc/nginx/sites-enabled/example.com.conf**

**NOTE**: Due to the configurations that we have outlined, all virtual host files must end in .conf. For the purposes of this example, we will assume a virtual host for port 80 already exists in its own config file.

Start by opening the virtual host file in your preferred text editor:
```sh
vim /etc/nginx/sites-available/example.com.conf
```

There are a few things to verify:
- ServerName/ServerAlias matches for your domain
- The DocumentRoot matches the the port 80 block
- The Directory section matches the port 80 block

You will need to change the example.com for your website name and place it in your virtual host:

```nginx
server {
    listen              443;

    server_name         example.com www.example.com;
    root                /var/www/vhosts/example.com;
    index               index.html;

    ssl                 on;
    ssl_certificate     /etc/pki/tls/certs/2022-example.com.chained.crt;
    ssl_certificate_key /etc/pki/tls/private/2022-example.com.key;

    access_log          /var/log/nginx/example.com_ssl_access.log main;
    error_log           /var/log/nginx/example.com_ssl_error.log;
}
```
Once the changes have been made to the appropriate config files, save and quit the file.

### Check syntax and enable server blocks in Nginx
Now that the Document Root and Server Blocks have been configured we can check the syntax of the config and reload Nginx.

**WARNING**: The following steps can lead to breaking the Apache process. Make sure that these steps are followed exactly and that you verify your work after making the changes. If the changes you make result in Apache process stopping, this means the domains on the server are now down. Revert any changes mad an ensure the Apache process is running or **Call to Rackspace Global Support**

##### RHEL/CentOS 6

In order to verify the syntax of the configuration files are correct, you will need to run the following command:
```sh
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your NGINX config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message

Once the syntax check returns OK, run the following command to reload Nginx:
```sh
$ service nginx reload
```

Once NGINX has been reloaded, remember to check that Nginx is running as expected as described in Prerequisites section. 

If you made any changes with PHP-FPM, you will also need to check the syntax of PHP-FPM and reload.
```sh
$ php-fpm -t
[16-Jun-2022 09:25:05] NOTICE: configuration file /etc/php-fpm.conf test is successful
```

If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your PHP-FPM config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message

Once the syntax check returns OK, run the following command to reload PHP=FPM:
```sh
$ service php-fpm reload
``` 
##### RHEL/CentOS 7

In order to verify the syntax of the configuration files are correct, you will need to run the following command:
```sh
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your Nginx config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message

Once the syntax check returns OK, run the following command to reload Nginx:
```sh
$ systemctl reload nginx
```

Once Nginx has been reloaded, remember to check that Nginx is running as expected as described in Prerequisites section. 

If you made any changes with PHP-FPM, you will also need to check the syntax of PHP-FPM and reload.
```sh
$ php-fpm -t
[16-Jun-2022 09:25:05] NOTICE: configuration file /etc/php-fpm.conf test is successful
```

If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your PHP-FPM config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message

Once the syntax check returns OK, run the following command to reload PHP=FPM:
```sh
$ systemctl reload php-fpm    
``` 
##### Debian/Ubuntu

In order to verify the syntax of the configuration files are correct, you will need to run the following command:
```sh
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your Nginx config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message

Once the syntax check returns OK, run the following command to reload Nginx:
```sh
$ systemctl reload nginx
```

Once Nginx has been reloaded, remember to check that Nginx is running as expected as described in Prerequisites section.

If you made any changes with PHP-FPM, you will also need to check the syntax of PHP-FPM and reload.
```sh
$ php-fpm -t
[16-Jun-2022 09:25:05] NOTICE: configuration file /etc/php-fpm.conf test is successful
```

If everything checks out, you will get a message that indicates **Syntax OK**. 

If you do not see this message, it indicates there may be a syntax issue in your PHP-FPM config files. 

**WARNING:** These errors will need to be addressed before completing the next step. Do not proceed until you've received a Syntax OK message

Once the syntax check returns OK, run the following command to reload PHP=FPM:
```sh
$ systemctl reload php-fpm    
```

### Install certificate on Managed Hosting solutions

If you have requested an SSL certificate for your Rackspace managed
server by submitting a Rackspace ticket, Rackspace installs the certificate
for you.  You should provide details including where you want the certificate
installed and your private key file.

### Install certificate on a custom Microsoft Azure domain

By default, Azure secures the **.azurewebsites.net** wildcard domain with a
single SSL certificate, so you can already access your application by using the
**https://\<appname>.azurewebsites.net** URL.

However, the default Azure SSL certificate does not work if you use a custom
domain for your application. A custom domain with its own SSL certificate is
more secure than the default. The following sections describe how to add an
SSL certificate to an application with a custom domain.

#### Prerequisite

You need your login credentials for the Azure portal. For information about
how to log in to the Azure portal, see
[Sitecore Cloud portals and account management](/support/how-to/sitecore-cloud-portals-and-account-management/).

#### Get an SSL certificate

If you do not already have SSL certificate, you need to get one from a trusted
CA. The certificate must meet all of the following requirements:

- Signed by a trusted CA (no private CA servers).
- Contains a private key.
- Created for key exchange and exported to a **.pfx** file.
- Uses a minimum of 2048-bit encryption.
- Has a subject name that matches the custom domain it needs to secure. To
secure multiple domains with one certificate, you need to use a wildcard name
(for example, **.contoso.com**) or specify the `subjectAltName` values.

- Merged with all intermediate certificates used by your CA. Otherwise, you
might experience irreproducible interoperability problems on some clients.

For more information on getting a certificate, see
[generate a certificate signing request (CSR)](/support/how-to/generate-a-csr)
and [purchase or renew a Secure Socket Layer (SSL certificate](/support/how-to/purchase-or-renew-an-ssl-certificate/).

#### Add the SSL certificate to Microsoft Azure

1. Log in to the [Azure portal](https://portal.azure.com).

2. In the left-side navigation pane, click **App services**.

3. Select the application to which you want to assign the certificate.

4. Navigate to **Settings** and then click **SSL certificate**.

5. Click **Upload Certificate**.

6. Select the **.pfx** file that contains your SSL certificate and enter the
password that you want to use for this certification.

7. Click **Upload**.

   You can now navigate to the SSL certificate through the application pane.

8. In the **SSL bindings** section of the SSL certificate pane, click
**Add bindings**.

   A new pane labeled **SSL Bindings** appears.

9. Use the drop-down menus to select the custom domain URL you want to secure
by using SSL, followed by the name of SSL certificate. You can also select
whether to use **Server Name Indication (SNI) SSL** or **IP-based SSL**.

10. Click **Add binding**.

    SSL is now enabled for your custom domain.

### Test the certificate

The best way to test a certificate is to use a third-party tool like the
Qualys&reg; [SSLLabs scanner](https://www.ssllabs.com/ssltest/). If you need
assistance in improving the security configuration of your certificate, contact
Rackspace Support.

**NOTE**: If you browse to your website by using the Hypertext Transfer Protocol
Secure (HTTPS) protocol directive, the padlock icon on your browser is displayed
in the locked position if your certificates are installed correctly and the server
is properly configured for SSL.

Another way to test the certificate is to go to [whynopadlock.com](https://www.whynopadlock.com/).
Enter your URL in **Secure Address**, and it shows any discrepancies that could
cause the site to be unsecure, such as mixed content issues.</br>

Use the Feedback tab to make any comments or ask questions. You can also [start a conversation with us](https://www.rackspace.com/contact).