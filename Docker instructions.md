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
        - From within loris-app directory
            - Create a _.env_ file which should contain the MySQL root password for database connection, e.g
                - `MYSQL_ROOT_PASSWORD=loris-database`
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
        - From within the container
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


    