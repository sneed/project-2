# Project2 Documentation

## STEP 1 – INSTALLING THE NGINX WEB SERVER
### Step 1 – Installing the Nginx Web Server
In order to display web pages to our site visitors, i am going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package.
Since this is our first time using apt for this session, start off by updating your server’s package index. Following that, you can use apt install to get Nginx installed:

`sudo apt update`
![Updating servers package ](../images/package-update.png)

`sudo apt install nginx`
![Nginx installation](../images/nginx-install.png)

When prompted, enter Y to confirm that you want to install Nginx. Once the installation is finished, the Nginx web server will be active and running on your Ubuntu 20.04 server.
To verify that nginx was successfully installed and is running as a service in Ubuntu, run:

`sudo systemctl status nginx`
![To Verify nginx installation](../images/nginx-serviceup.png)

If it is green and running, then you did everything correctly – you have just launched your first Web Server in the Clouds!
Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is default port that web brousers use to access web pages in the Internet.
As we know, we have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80:

Our server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).
First, let us try to check how we can access it locally in our Ubuntu shell, run:
`curl http://localhost:80`
![Verfify Server access via DNS name](../images/server-accessdnsname.png)

These 2 commands above actually do pretty much the same – they use ‘curl’ command to request our Nginx on port 80 (actually you can even try to not specify any port – it will work anyway). The difference is that: in the first case we try to access our server via DNS name and in the second one – by IP address (in this case IP address 127.0.0.1 corresponds to DNS name ‘localhost’ and the process of converting a DNS name to IP address is called "resolution"). We will touch DNS in further lectures and projects.
As an output you can see some strangely formatted test, do not worry, we just made sure that our Nginx web service responds to ‘curl’ command with some payload.

Now it is time for us to test how our Nginx server can respond to requests from the Internet.
Open a web browser of your choice and try to access following url
`http://<Public-IP-Address>:80`
![Nginx server request and responds](../images/welcome-nginx.png)

## STEP 2 — INSTALLING MYSQL
Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.

Again, use ‘apt’ to acquire and install this software:

`sudo apt install mysql-server`
![Mysql server install](../images/mysql-server.png)

When prompted, confirm installation by typing Y, and then ENTER.
When the installation is finished, log in to the MySQL console by typing:

`sudo mysql`
This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. You should see output like this:
![Log into mysql server](../images/loginmysql-server.png)

Exit the MySQL shell wit
`mysql> exit`
![My SQL shell exit](../images/mysql-exit.png)

## STEP 3 – INSTALLING PHP
### Step 3 – Installing PHP
You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.
While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.
To install these 2 packages at once, run:
`sudo apt install php-fpm php-mysql`
![Install php-fpm php-mysql](../images/phpfpm-phpmysql.png)
When prompted, type Y and press ENTER to confirm installation.


You now have your PHP components installed. Next, you will configure Nginx to use them.
## STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR
### Step 4 — Configuring Nginx to Use PHP Processor

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use projectLEMP as an example domain name.
On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.

Create the root web directory for your_domain as follows:
`sudo mkdir /var/www/projectLEMP`
![Creating root web directory](../images/root-webdir.png)

Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:
`sudo chown -R $USER:$USER /var/www/projectLEMP`
![User environment variable](../images/user-envar.png)

Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor.
`sudo nano /etc/nginx/sites-available/projectLEMP`

This will create a new blank file. Paste in the following bare-bones configuration:
![Config file for Nginx sites-available ](../images/nginx-sitesconfig.png)

Here’s what each of these directives and location blocks do:
listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
root — Defines the document root where the files served by this website are stored.
index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.
When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.

Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
`sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`

This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:
`sudo nginx -t`
You shall see following message:
![Test configuration for syntax errors](../images/tstconfg-syntaxerr.png)

If any errors are reported, go back to your configuration file to review its contents before continuing.

We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:
`sudo unlink /etc/nginx/sites-enabled/default`
![Disable default Nginx](../images/disable-defaultnginx.png)

When you are ready, reload Nginx to apply the changes:
`sudo systemctl reload nginx`
![Reload nginx ](../images/reload-nginxserv.png)

Your new website is now active, but the web root /var/www/projectLEMP is still empty. 
Create an index.html file in that location so that we can test that your new server block works as expected:
`sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html`
![Create html file in web root ](../images/indexhtml-webroot.png)

Now go to your browser and try to open your website URL using IP address:
`http://<Public-IP-Address>:80`
![Open Website URL using IP address](../images/websiteurl-ipaddress.png)

If you see the text from ‘echo’ command you wrote to index.html file, then it means your Nginx site is working as expected.
In the output you will see your server’s public hostname (DNS name) and public IP address. You can also access your website in your browser by public DNS name, not only by IP – try it out, the result must be the same (port is optional)
`http://<Public-DNS-Name>:80`
![Public hostname (DNS name) and public IP address](../images/dnsname-ipaddress.png)
You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.



## STEP 5 – TESTING PHP WITH NGINX
### Step 5 – Testing PHP with Nginx
Your LEMP stack should now be completely set up.

At this point, your LAMP stack is completely installed and fully operational.
You can test it to validate that Nginx can correctly hand .php files off to your PHP processor.
You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:
`sudo nano /var/www/projectLEMP/info.php`
![Create Test php file](../images/testphp-file.png)

Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:
![File populated with php code](../images/testphp-file1.png)

You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:
`http://`server_domain_or_IP`/info.php`
You will see a web page containing detailed information about your server:
![Web page with server information](../images/webpage-phpserverinfo.png)

After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:

`sudo rm /var/www/your_domain/info.php`
![Removal of sensitive information](../images/remove-infophp.png)
You can always regenerate this file if you need it later


## STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)
### Step 6 — Retrieving data from MySQL database with PHP

In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.
At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.
We will create a database named example_database and a user named example_user, but you can replace these names with different values.
First, connect to the MySQL console using the root account

`sudo mysql`
![My SQL console root connection](../images/rootuser-sqlconn.png)

To create a new database, run the following command from your MySQL console:
`mysql> CREATE DATABASE `example_database`;`
![Create database](../images/create-db.png)

Now you can create a new user and grant him full privileges on the database you have just created.
The following command creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.
`mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`


Now we need to give this user permission over the example_database database:
`mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';`

This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.
Now exit the MySQL shell with:
`mysql> exit`
![Exit MySQL](../images/exit-sql.png)

You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:
`mysql -u example_user -p`
![Test user logging permissions into MySQL](../images/logging-permissions.png)

Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user. After logging in to the MySQL console, confirm that you have access to the example_database database:
`mysql> SHOW DATABASES;`
This will give you the following output:
![SHOW DATABASES; Output](../images/showdb-output.png)

Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:
`mysql> CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT, content VARCHAR(255), PRIMARY KEY(item_id));`
![Create todo-list](../images/createtodo-list.png)

Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:
`mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");`
![Inser content intest table](../images/content-testtable.png)

To confirm that the data was successfully saved to your table, run:
`mysql>  SELECT * FROM example_database.todo_list;`
![Confirm saved data](../images/saved-data.png)

After confirming that you have valid data in your test table, you can exit the MySQL console:
`mysql> exit`
![Exit SQL](../images/exit-sql.png)

Now you can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that:
`nano /var/www/projectLEMP/todo_list.php`
![Create php file](../images/create-phpfile.png)

The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.
Copy this content into your todo_list.php script:
![Create php script](../images/todo-listphp.png)

Save and close the file when you are done editing.
You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:
`http://<Public_domain_or_IP>/todo_list.php`
![public IP address  with todo list](../images/url-todo_listphp.png)

You should see a page like this, showing the content you’ve inserted in your test table:
![Page with content in test table](../images/pagecontent-testtable.png)

That means your PHP environment is ready to connect and interact with your MySQL server.

Congratulations!