#
#   _____                  __  __             _   _        ____  _  ______  
#  | ____|__ _ ___ _   _  |  \/  | __ _ _   _| |_(_) ___  |  _ \| |/ /  _ \ 
#  |  _| / _` / __| | | | | |\/| |/ _` | | | | __| |/ __| | | | | ' /| |_) |
#  | |__| (_| \__ \ |_| | | |  | | (_| | |_| | |_| | (__  | |_| | . \|  _ < 
#  |_____\__,_|___/\__, | |_|  |_|\__,_|\__,_|\__|_|\___| |____/|_|\_\_| \_\
#                   |___/                                                    
#
#      Pull & Play, extremely verbose, KISS Dockerfile for Mautic 5.1.1
#
#
# Intended uses: 
# - A simple way to test Mautic.
# - An live overview of Mautic requirements.
# - A starting point for Mautic code exploration and tinkering.
# It is NOT meant as a production ready image.

# If you have any ideas how to make it clearer or simpler, please submit a PR or an Issue.
# Otherwise, please consider one of these other repos: 
# kissmautic/mautic-advanced, For production deployments.
# kissmautic/mautic-dev, For development stacks.

# We use ARGuments to parametrize the build-time parameters.
# While Debian is the only OS supported by this Dockerfile,
# you should be able to try other versions of Debian by adjusting the OS_VER ARG.
ARG BASE_OS=debian
ARG OS_VER=12-slim

# By default, effectively converted to: FROM debian:12-slim
FROM ${BASE_OS}:${OS_VER}

#Prevent the CLI from asking questions during build time.
ARG DEBIAN_FRONTEND=noninteractive

# Define versions for the different components of the stack.
ARG PHP_VER=8.2
ARG MARIADB_VER=11.4
ARG MAUTIC_VER=5.1.1


###--------------------------------###
###  Prepare for the installation  ###
###--------------------------------###

# Update the package lists and upgrade any packages to their latest version.
RUN apt update && apt upgrade -y

# Install necessary packages for the setup preparation steps.
# wget for downloading files from the internet.
# gnupg2 for package verification.
# lsb-release: To determine the Debian version.
# ca-certificates for SSL/TLS verification.
# apt-transport-https to be able to use use HTTPS with apt repositories.
# curl for downloading files and making web requests.
RUN apt-get install -y wget curl gnupg2 lsb-release ca-certificates apt-transport-https software-properties-common

# Download the GPG key for the PHP repository.
# This key is used to verify the integrity of the PHP packages we'll install.
RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg

# Add the PHP repository to the list of sources.
# This allows us to install specific PHP versions not available in the default Debian repositories.
RUN echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list

# Create a directory for APT keyrings.
# This is where we'll store the MariaDB GPG key.
RUN mkdir -p /etc/apt/keyrings

# Download the MariaDB GPG key.
# This key is used to verify the integrity of the MariaDB packages.
RUN curl -o /etc/apt/keyrings/mariadb-keyring.pgp 'https://mariadb.org/mariadb_release_signing_key.pgp'

# Add the MariaDB repository to the list of sources.
# This allows us to install a specific version of MariaDB.
RUN echo "deb [signed-by=/etc/apt/keyrings/mariadb-keyring.pgp] https://dlm.mariadb.com/repo/mariadb-server/${MARIADB_VER}/repo/debian $(lsb_release -sc) main" > /etc/apt/sources.list.d/mariadb.list


###--------------------------------###
###   Install the stack packages   ###
###--------------------------------###

# Update package lists after adding new repositories.
# This ensures we have the latest information about packages in the newly added repositories.
RUN apt update && apt upgrade -y

# Install the components of the stack and other packages these require.
# Apache2: The HTTP server.
# libapache2-mod-fcgid: FastCGI module for Apache, needed for PHP-FPM.
# MariaDB: MySQL compatible Database. 
# ntp: Time service.
# openssl: 
# unzip: To extract zip files, like the mautic releases from GitHub.
RUN apt install -y apache2 libapache2-mod-fcgid mariadb-server ntp openssl unzip
EXPOSE 3306

# Install PHP and PHP Extensions required to run Mautic.
#RUN apt install php8.2 php-bcmath php-curl php-igbinary php-intl php-mbstring php-xml
#RUN apt install php8.2-imap php8.2-bcmath php8.2-bz2 php8.2-cli php8.2-common php8.2-curl php8.2-gd \
#    php8.2-gmp php8.2-igbinary php8.2-intl php8.2-mbstring php8.2-mysql php8.2-readline php8.2-phpdbg \
#    php8.2-xml php8.2-zip php8.2-soap php8.2-xmlrpc php8.2-tidy -y

RUN apt install -y php${PHP_VER}-fpm php-bcmath php-curl php-igbinary php-intl php-mbstring php-xml
RUN apt install -y php${PHP_VER}-imap php${PHP_VER}-bcmath php${PHP_VER}-bz2 php${PHP_VER}-cli php${PHP_VER}-common php${PHP_VER}-curl php${PHP_VER}-gd \
    php${PHP_VER}-gmp php${PHP_VER}-igbinary php${PHP_VER}-intl php${PHP_VER}-mbstring php${PHP_VER}-mysql php${PHP_VER}-readline php${PHP_VER}-phpdbg \
    php${PHP_VER}-xml php${PHP_VER}-zip php${PHP_VER}-soap php${PHP_VER}-xmlrpc php${PHP_VER}-tidy -y

#RUN apt-get install -y php${PHP_VER} php${PHP_VER}-{apcu,bcmath,fpm,mysql,amqplib,curl,igbinary,intl,mbstring,gd,imap,opcache,xml,zip}

# Set the installed PHP version as the default.
# This ensures that when we run 'php' commands, we use the version we just installed.
RUN update-alternatives --set php /usr/bin/php${PHP_VER}

# Enable the Apache FCGI module.
# This is necessary for Apache to communicate with PHP-FPM.
RUN a2enmod fcgid rewrite
#THAT WAS PART OF a2enmod php${PHP_VER} rewrite

###--------------------------------###
###    Get configuratrion files    ###
###--------------------------------###

# Get the configuration files for the different components of the stack.
# This file contains the configuration for FPM
RUN wget https://files.mktg.dev/pub/dkr/511/www.conf -O /etc/php/8.2/fpm/pool.d/www.conf
#This file contains the configuration for the event MPM
RUN wget https://files.mktg.dev/pub/dkr/511/mpm_event.conf -O /etc/apache2/mods-enabled/mpm_event.conf
# This file contains the Apache2 Vhost configuration
RUN wget https://files.mktg.dev/pub/dkr/511/000-default.conf -O /etc/apache2/sites-available/000-default.conf
# This file contains the configuration for MariaDB
#RUN wget https://files.mktg.dev/pub/dkr/511/50-server.cnf -O /etc/mysql/mariadb.conf.d/50-server.cnf
# This script inserts some required config values into php.ini.
RUN wget https://files.mktg.dev/pub/dkr/511/insert-php82.ini
# This file contains the Mautic cronjobs 
RUN wget https://files.mktg.dev/pub/dkr/511/m9cronjobs
# This file makes your prompt sexier.
RUN wget --user prv1 --password Hormiga2023 https://files.mktg.dev/prv/m8/base/bashrc -O ~/.bashrc
# This file greets you when you login to your Mautic container.
RUN wget --user prv1 --password Hormiga2023 https://files.mktg.dev/prv/m8/base/hi5 -O /usr/local/bin/hi5

RUN chmod +x /usr/local/bin/hi5
RUN chmod +x insert-php82.ini
RUN ./insert-php82.ini
RUN rm insert-php82.ini


###---------------------------------###
###  Prepare MariaDB configuration  ###
###---------------------------------###

# Create directory for MariaDB data
RUN mkdir -p /var/lib/mysql

# Copy initialization script
COPY init-db.sql /docker-entrypoint-initdb.d/

# Expose MariaDB port
EXPOSE 3306

# Start MariaDB service
# CMD ["mysqld"]

# Create a volume for MariaDB data
VOLUME /var/lib/mysql



###---------------------------------###
###   Download and install Mautic   ###
###---------------------------------###

# Set the working directory
WORKDIR /var/www/html
# Download Mautic.
RUN wget https://github.com/mautic/mautic/releases/download/${MAUTIC_VER}/${MAUTIC_VER}.zip
# Extract the Mautic archive.
RUN unzip ${MAUTIC_VER}.zip
# We don't need the zip file after extraction.
RUN rm ${MAUTIC_VER}.zip
# Set the correct ownership for the web files, ensures that the Apache process can read and write these files.
RUN chown -R www-data:www-data /var/www/html
# Set the correct permissions for directories.
# 755 allows the owner to read, write, and execute, and others to read and execute.
RUN find /var/www/html -type d -exec chmod 755 {} +
# Set the correct permissions for files.
# 644 allows the owner to read and write, and others to read.
RUN find /var/www/html -type f -exec chmod 644 {} +


###---------------------------------###
###     Cleanup and final steps     ###
###---------------------------------###

# Remove unused packages.
RUN apt-get autoremove -y
# Clean the APT cache to further reduce image size.
RUN apt-get clean
# Clean up the APT cache again to keep the image size down.
RUN rm -rf /var/lib/apt/lists/*

EXPOSE 80

## HEALTHCHECK
# Add a healthcheck to ensure the container is running correctly.
# This curls localhost every 30 seconds, marking the container as unhealthy if it fails.
# It's useful for orchestration systems to determine if the container is functioning properly.
#HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
#  CMD curl -f http://localhost/ || exit 1

# Specify the command to run when the container starts.
# This starts MariaDB, PHP-FPM, and Apache.
# Apache is started in the foreground to keep the container running.
CMD service mariadb start && service php${PHP_VER}-fpm start && apache2ctl -D FOREGROUND && echo "Hey there Kevin!"

#
#
#                                 --                                   
#                           ((((((()))))))                              
#                      ((((((((         ))))))                          
#                   (((((                      ,                     
#                ((((                        ***    )))                  
#              ((((                        ****      )))           
#             ((((          ,             ****        )))                 
#            (((           ****         ****           )))              
#           (((           *******     *****             )))             
#           (((          ********** ******  **          )))             
#           (((         ****   *********   ****         )))             
#           (((        ****     ******      ****        )))             
#           (((       ****         *         ****       )))             
#            (((     ****                     ****     )))              
#             ((((                                   ))))               
#               (((                                 )))                 
#                (((((                           )))))                  
#                   (((((                     )))))                     
#                       (((((((         )))))))                        
#                            (((((()))))))                              
#                                 --                                   
#                      _     _                _                   
#                     | |   | |              | |                  
#           ________  | | __| |_  ____     __| |  ___ __   __     
#          |  _   _ \ | |/ /| __|/ _  |   / _  | / _ \  \ / /           
#          | | | | | ||   < | |_| (_| | _| (_| ||  __/ \ V /      
#          |_| |_| |_||_|\_\ \__|\___ |(_)\____| \___|  \_/       
#                         _       __/ |                           
#                        | |     |___/                            
#   ________   ____ _   _| |_ ___  ____ _______        ___  ____ ____ 
#  |  _   _ \ / _  | | | | __/ _ \/ _  |  _   _ \     / _ \|  __/ _  |
#  | | | | | | (_| | |_| | ||  __/ (_| | | | | | | _  |(_) | | | (_| |
#  |_| |_| |_|\__,_|\__,_|\__\___|\__,_|_| |_| |_|(_) \___/|_|  \__, |
#                                                                __/ |
#                                                               |___/ 
#                                                                         

## Hi there 👋

<!--
**kissmautic/kissmautic** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
