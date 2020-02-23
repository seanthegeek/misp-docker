
FROM debian:buster

# Install core components
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y locales apt-utils && rm -rf /var/lib/apt/lists/* \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

RUN apt-get update && apt-get dist-upgrade -y && apt-get autoremove -y && apt-get clean && \
    apt-get install -y software-properties-common && \
    apt-get install -y postfix && \
    apt-get install -y mariadb-client curl gcc git gnupg-agent \
        make python openssl redis-server sudo vim nano less zip


# Apache
RUN apt-get install -y apache2 apache2-doc apache2-utils && \
    a2dismod status && \
    a2dissite 000-default
# PHP
RUN apt-get install -y libapache2-mod-php php php-dev php-json \
        php-mysql php-redis php-xml php-mbstring \
        php-gd php-pear php-opcache \
        pkg-config libbson-1.0 libmongoc-1.0-0

# Fix php.ini with recommended settings
RUN sed -i \
        -e "s/max_execution_time = 30/max_execution_time = 300/" \
        -e "s/memory_limit = 128M/memory_limit = 2048M/" \
        -e "s/upload_max_filesize = 2M/upload_max_filesize = 50M/" \
        -e "s/post_max_size = 8M/post_max_size = 50M/" \
        /etc/php/7.3/apache2/php.ini

RUN apt-get install -y python3-dev python3-pip python3-setuptools \
        python3-lxml libjpeg-dev \
        libxml2-dev libxslt1-dev zlib1g-dev libfuzzy-dev && \
    apt-get install -y cron logrotate supervisor syslog-ng-core && \
    apt-get clean

WORKDIR /var/www
RUN chown www-data:www-data /var/www
USER www-data
RUN git clone https://github.com/MISP/MISP.git
WORKDIR /var/www/MISP
RUN git config core.filemode false && \
    git submodule update --init --recursive && \
    git submodule foreach --recursive git config core.filemode false

USER root
RUN pip3 install git+https://github.com/STIXProject/python-stix.git \
                 git+https://github.com/CybOXProject/python-cybox.git \
                 git+https://github.com/CybOXProject/mixbox.git \
                 git+https://github.com/MAECProject/python-maec.git \
                 /var/www/MISP/cti-python-stix2 \
                 plyara

USER www-data
WORKDIR /var/www/MISP
RUN git submodule init && git submodule update
WORKDIR /var/www/MISP/app

# FIX COMPOSER
RUN curl --fail --location -o composer-setup.php https://getcomposer.org/installer && \
    EXPECTED_SIGNATURE="$(curl https://composer.github.io/installer.sig)"; php -r "if (hash_file('sha384', 'composer-setup.php') == '$(echo $EXPECTED_SIGNATURE)' ) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');"
# END FIX

RUN php composer.phar update && \
    php composer.phar config vendor-dir Vendor && \
    php composer.phar install --ignore-platform-reqs
USER root
RUN phpenmod redis
USER www-data
RUN cp -fa /var/www/MISP/INSTALL/setup/config.php /var/www/MISP/app/Plugin/CakeResque/Config/config.php

# Fix permissions
USER root
RUN chown -R www-data:www-data /var/www/MISP && \
    chmod -R 750 /var/www/MISP && \
    chmod -R g+ws /var/www/MISP/app/tmp && \
    chmod -R g+ws /var/www/MISP/app/files && \
    chmod -R g+ws /var/www/MISP/app/files/scripts/tmp && \
    chmod +x /var/www/MISP/app/Console/cake

RUN cp /var/www/MISP/INSTALL/misp.logrotate /etc/logrotate.d/misp

# Preconfigure setting for packages
#RUN echo "postfix postfix/main_mailer_type string Local only" \
#  | debconf-set-selections && \
#   echo "postfix postfix/mailname string localhost.localdomain" | \
#   debconf-set-selections

# Redis Setup
RUN sed -i 's/^\(daemonize\s*\)yes\s*$/\1no/g' /etc/redis/redis.conf

# Add a healthcheck endpoint
COPY healthcheck.patch healthcheck.patch
RUN patch /var/www/MISP/INSTALL/apache.misp.ubuntu < healthcheck.patch

# Apache Setup
RUN cp /var/www/MISP/INSTALL/apache.misp.ubuntu /etc/apache2/sites-available/misp.conf && \
    a2dissite 000-default && \
    a2ensite misp && \
    a2enmod rewrite && \
    a2enmod headers

# MISP base configuration
RUN sudo -u www-data cp -a /var/www/MISP/app/Config/bootstrap.default.php /var/www/MISP/app/Config/bootstrap.php && \
    sudo -u www-data cp -a /var/www/MISP/app/Config/database.default.php /var/www/MISP/app/Config/database.php && \
    sudo -u www-data cp -a /var/www/MISP/app/Config/core.default.php /var/www/MISP/app/Config/core.php && \
    sudo -u www-data cp -a /var/www/MISP/app/Config/config.default.php /var/www/MISP/app/Config/config.php && \
    chown -R www-data:www-data /var/www/MISP/app/Config && \
    chmod -R 750 /var/www/MISP/app/Config

# Replace the default salt
RUN sed -i -E "s/'salt'\s=>\s'(\S+)'/'salt' => '`openssl rand -base64 32|tr "/" "-"`'/" /var/www/MISP/app/Config/config.php

# Enable workers at boot time
#RUN chmod a+x /var/www/MISP/app/Console/worker/start.sh && \
#    echo "sudo -u www-data bash /var/www/MISP/app/Console/worker/start.sh" >>/etc/rc.local

# Install templates & stuff
WORKDIR /var/www/MISP/app/files
RUN rm -rf misp-objects && git clone https://github.com/MISP/misp-objects.git && \
    rm -rf misp-galaxy && git clone https://github.com/MISP/misp-galaxy.git && \
    rm -rf warninglists && git clone https://github.com/MISP/misp-warninglists.git ./warninglists && \
    rm -rf taxonomies && git clone https://github.com/MISP/misp-taxonomies.git ./taxonomies && \
    chown -R www-data:www-data misp-objects misp-galaxy warninglists taxonomies

# Install MISP build requirements
RUN sudo apt-get install cmake python3-dev python3-pip libpq5 libjpeg-dev tesseract-ocr libpoppler-cpp-dev imagemagick virtualenv libopencv-dev zbar-tools libzbar0 libzbar-dev libfuzzy-dev build-essential -y

# Install misp-modules inside a virtualenv
RUN sudo -u www-data virtualenv -p python3 /var/www/MISP/venv
WORKDIR /usr/local/src/
RUN chown -R www-data .
RUN git clone https://github.com/stricaud/faup.git
WORKDIR /usr/local/src/faup/build
RUN cmake .. && make
RUN sudo make install
RUN sudo ldconfig
WORKDIR /usr/local/src/faup/src/lib/bindings/python
RUN sudo -u www-data /var/www/MISP/venv/bin/pip install .
WORKDIR /usr/local/src
RUN sudo git clone https://github.com/MISP/misp-modules.git
WORKDIR /usr/local/src/misp-modules
RUN touch misp_modules/modules/expansion/_ransomcoindb/__init__.py
RUN chown -R www-data .
RUN sudo -u www-data /var/www/MISP/venv/bin/pip install -I -r REQUIREMENTS
RUN sudo -u www-data /var/www/MISP/venv/bin/pip install .

# Supervisord Setup
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
 
# Modify syslog configuration
RUN sed -i -E 's/^(\s*)system\(\);/\1unix-stream("\/dev\/log");/' /etc/syslog-ng/syslog-ng.conf

# Add run script
# Trigger to perform first boot operations
ADD run.sh /run.sh
RUN chmod 0755 /run.sh && touch /.firstboot.tmp

# Make a backup of /var/www/MISP to restore it to the local moint point at first boot
WORKDIR /var/www/MISP
RUN tar czpf /root/MISP.tgz .

VOLUME /var/www/MISP
EXPOSE 80
ENTRYPOINT ["/run.sh"]
