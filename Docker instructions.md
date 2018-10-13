Loris Docker deployment instructions:
===
* Requirements
    - [Docker](https://docs.docker.com/install/)
    - [Docker Compose](https://docs.docker.com/compose/install/)

* Steps
    - Create base Docker image with Apache, PHP and Loris project using Dockerfile in php-apache directory
        - Notice that a custom install script is used in order to properly configure debian, which is unsupported by original `install.sh`
        - From within php-apache directory run
            - `docker build -t loris-php .`
    - Since the `install.sh` script must be run in interactive mode, you must execute
        - `docker run -it --name loris-app loris-php bash`
        - From within the container
            - Switch to lorisadmin user
                - `sudo -s -u lorisadmin`
            - Execute install script
                - `cd tools`
                - `./install.sh`
                - Provide necessary script information, like project name (loris), and allow script to configure Apache
            - Restart apache server
                - `sudo service apache2 restart`
            - Exit the lorisadmin user and container 
                    - 2 x `Ctrl+D`
        - Create a new image with the changes made by the install script
            - `docker commit loris-app loris-app`
    - Run docker-compose.yml from loris-app directory in order to spin up application and database docker images
        - Create base directory for volume storage, as well as sub-directories for each of the volumes, e.g:
            - `mkdir /data/loris-volumes`
                - `mkdir /data/loris-volumes/loris-app`
                - `mkdir /data/loris-volumes/loris-data`
                - `mkdir /data/loris-volumes/incoming-data`
                - `mkdir /data/loris-volumes/mysql-data`
        - From within loris-app directory
            - Create a _.env_ file which should contain the MySQL root password for database connection and the basepath for app volumes, e.g
                - `MYSQL_ROOT_PASSWORD=loris-password`
                - `VOLUME_PATH=/data/loris-volumes`
            - Spin up the app
                - `docker-compose up` or `docker-compose up -d` if you would like to run in background
    - Refer to the [documentation online](https://github.com/aces/Loris/wiki/Installing-Loris-in-Brief#installing-the-database---1-of-2) on how to configure application and database username/password. The application is accessible from the IP of the machine where it is running, port 80, or http://localhost if you are running it locally. The MySQL database is accessible at host `db` and the username/password for the database are `root` and whichever password you specified in _.env_ file respectively

Steps to install Imaging Database
===
After configuring Loris by following all of the steps above, you may want to install the Imaging Database. In order to do that, you need to follow these steps:
- Stop and remove running loris app container and database
    - From within loris-app directory run
        - `docker-compose down`
- Build new loris-app image using Dockerfile in imaging directory
    - From within imaging directory run
        - `docker build -t loris-app .`
- Since the `imaging_install.sh` script must be run in interactive mode, you must first spin up the application and database with appropiate links 
        - From within loris-app directory
            - `docker-compose up -d`
        - Enter the container
            - `docker-compose exec loris bash`
            - Grant lorisadmin ownership over /data directory
                - `sudo chown -R lorisadmin:www-data /data`
            - Execute install script
                - `sudo -s -u lorisadmin`
                - `sudo apt-get update`
                - `./imaging_install.sh`
                - Provide necessary script information, like project name (loris) and database connection info from Loris deployment configuration. Database should be accessible at host `db`
            - Load minc environment
                - `source /home/lorisadmin/.bashrc`
            - Exit the lorisadmin user and container 
                    - 2 x `Ctrl+D`
        - Create a new image with the changes made by the install script
            - `docker commit [loris container name] loris-app`
        - Stop app
            - `docker-compose down`
        - Start again with new image
            - `docker-compose up -d`

Steps to install Data Query Tool
===
Stop running loris app
    - From within loris-app directory run
        - `docker-compose down`
Spin up a new loris app using the docker-compose.yml file located in the data_query_tool directory
    - Create, inside the data_query_tool directory, a a _.couchdb_env_ file which should contain the username and password for the couchbase db connection e.g
                - `COUCHDB_USER=root`
                - `COUCHDB_PASSWORD=loris-password`
    - Save the docker-compose.yml file inside the loris-app directory
        - From within loris-app directory run
            - `mv docker-compose.yml docker-compose.yml.backup` 
    - Copy the docker-compose file from data_query_tool directory to the loris-app directory 
        - From within loris-app directory run
            - `cp ../data_query_tool/docker-compose.yml .`
    - Spin up the app:
        - `docker-compose up -d`
Set up CouchDB database by replicating existing database and running import scripts following instructions given in https://github.com/aces/Data-Query-Tool
    - Run a container mounting the loris-app volume in order to edit config.xml file to add the new CouchDB configuration parameters
        - `docker run -it --rm -v lorisapp_loris-app:/app alpine sh`
        - `vi /app/project/config.xml`
        - Change the section `<CouchDB>...</CouchDB>` accordingly. Database host is `couchdb` and the port is 5984. Username and passwords are whatever you set in your .couchdb_env file. Database name is whatever you want (dqg is recommended), but please remember the name for subsequent steps
        - Exit the container `Ctrl+D`
    - Enter the loris-app container. From within loris-app directory run
        - `docker-compose exec loris bash`
        - Install erica and its dependencies in order to replicate existing database
            - `sudo apt-get install -yy rebar erlang-src erlang-xmerl erlang-parsetools`
            - `wget https://people.apache.org/~dch/dist/tools/erica`
            - `mv erica /usr/bin`
            - `chmod +x /usr/bin/erica`
        - Exit the container, commit changes and restart application
            - `Ctrl-D`
            - `docker commit [loris container name] loris-app`
            - `docker-compose down`
            - `docker-compose up -d`
    - Clone data query tools repo and replicate database, using configuration parameters you set in previous step
        - `docker-compose exec loris bash`
        - `git clone https://github.com/aces/Data-Query-Tool.git`
        - `cd Data-Query-Tool`
        - `erica push http://$YOURCOUCHDBADMIN:$YOURCOUCHADMINPASS@$YOURSERVERNAME:5984/$YOURDATABASENAME`
        - At this point, erica will output a message saying you can access db at a given URL. This is not possible unless you bind the port in the docker-compose file
    - In your Loris tools directory run the CouchDB_Import_* scripts
        - From /var/www/loris/tools directory run
            - php CouchDB_Import_Demographics.php
            - php CouchDB_Import_Instruments.php
            - php CouchDB_Import_MRI.php
        - Exit the container and test Loris deployment
            - `Ctrl-D`


    