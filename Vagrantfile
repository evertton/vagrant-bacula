# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "generic/debian9"
  config.vm.box_check_update = true
  #config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provision "shell", inline: <<-SHELL
    set -x
    apt-get update -y && apt upgrade -y
    apt-get install -y nano build-essential

    wget -qO- https://master.dl.sourceforge.net/project/bacula/bacula/7.0.5/bacula-7.0.5.tar.gz | tar -xvzf - -C /usr/src

    printf "\n# Endereços do ambiente Bacula\n" >> /etc/hosts
    printf "192.168.111.9\tbacula-nfs\n" >> /etc/hosts
    printf "192.168.111.10\tbacula-sd\n" >> /etc/hosts
    printf "192.168.111.11\tbacula-dir\n" >> /etc/hosts
    printf "192.168.111.12\tbacula-fd1\n" >> /etc/hosts
    printf "192.168.111.13\tbacula-fd2\n" >> /etc/hosts
  SHELL
    
#  config.vm.define "nfs", primary: true do |s|
#      s.vm.hostname = "bacula-nfs"
#      s.vm.network "private_network", ip: "192.168.111.9"
#      s.vm.provision "shell", inline: <<-SHELL
#        
#      SHELL
#  end

  config.vm.define "sd", primary: true do |s|
      s.vm.hostname = "bacula-sd"
      s.vm.network "private_network", ip: "192.168.111.10"
      s.vm.provision "shell", inline: <<-SHELL
        apt-get install -y sqlite3 libsqlite3-dev

        cd /usr/src/bacula-7.0.5
        CFLAGS="-g -Wall" \
        ./configure \
            --bindir=/sbin \
            --libdir=/usr/lib \
            --sysconfdir=/etc/bacula \
            --with-scriptdir=/etc/bacula \
            --with-archivedir=/tmp \
            --with-working-dir=/opt/bacula/working \
            --with-pid-dir=/var/run \
            --with-subsys-dir=/var/run/subsys \
            --mandir=${datarootdir}/man \
            --datadir=/usr/share \
            --with-plugindir=/usr/lib \
            --with-sqlite3 \
            --with-db-name=bacula \
            --with-db-user=bacula \
            --with-job-email=root@localhost \
            --with-smtp-host=localhost \
            --disable-conio \
            --enable-smartalloc \
            --disable-build-dird \
            --disable-afs \
            --disable-acl \
            --enable-xattr \
            --enable-batch-insert

        make install
        make install-autostart-sd
        make install-autostart-fd

        cd /etc/bacula
        ./create_sqlite3_database
        ./make_sqlite3_tables
        ./grant_sqlite3_privileges

        service bacula-sd start
        service bacula-fd start
      SHELL
  end
  
  config.vm.define "dir", primary: true do |s|
      s.vm.hostname = "bacula-dir"
      s.vm.network "private_network", ip: "192.168.111.11"
      s.vm.provision "shell", inline: <<-SHELL
        apt-get install -y postgresql libpq-dev

        cd /usr/src/bacula-7.0.5
        CFLAGS="-g -Wall" \
        ./configure \
            --bindir=/sbin \
            --libdir=/usr/lib \
            --sysconfdir=/etc/bacula \
            --with-scriptdir=/etc/bacula \
            --with-archivedir=/tmp \
            --with-working-dir=/opt/bacula/working \
            --with-pid-dir=/var/run \
            --with-subsys-dir=/var/run/subsys \
            --mandir=${datarootdir}/man \
            --datadir=/usr/share \
            --with-plugindir=/usr/lib \
            --with-postgresql \
            --with-db-name=bacula \
            --with-db-user=bacula \
            --with-db-password=bacula \
            --with-job-email=root@localhost \
            --with-smtp-host=localhost \
            --disable-conio \
            --enable-smartalloc \
            --disable-build-stored \
            --disable-afs \
            --disable-acl \
            --enable-xattr \
            --enable-batch-insert
        
        make install
        make install-autostart-dir
        make install-autostart-fd
        
        mkdir /tmp/bacula
        cp /etc/bacula/create_postgresql_database /tmp/bacula/create_postgresql_database
        cp /etc/bacula/make_postgresql_tables /tmp/bacula/make_postgresql_tables
        cp /etc/bacula/grant_postgresql_privileges /tmp/bacula/grant_postgresql_privileges
        sed -i 's/db_password=/db_password=bacula/' /tmp/bacula/grant_postgresql_privileges
        chown -R postgres:postgres /tmp/bacula
        
        su -c "/tmp/bacula/create_postgresql_database" -s /bin/sh postgres
        su -c "/tmp/bacula/make_postgresql_tables" -s /bin/sh postgres
        su -c "/tmp/bacula/grant_postgresql_privileges" -s /bin/sh postgres
        
        mkdir /opt/bacula/log
        
        sed -i 's/local   all             all                                     peer/local   all             all                                     md5/' /etc/postgresql/9.6/main/pg_hba.conf
        sudo -u postgres psql -c "ALTER USER bacula WITH PASSWORD 'bacula'" bacula
        
        service postgresql restart
        service bacula-dir start
        service bacula-fd start
        
        # instalação conforme tutorial <http://www.bacula.lat/instalacao-webacula-5-x-gui-web/>
        
        apt-get install -y apache2 php7.0 libapache2-mod-php php7.0-pgsql php-gd 
        
        mkdir /var/www
        wget -qO- https://ufpr.dl.sourceforge.net/project/webacula/webacula/5.5.1/webacula-5.5.1.tar.gz |  tar -xvzf - -C /var/www
        mv /var/www/webacula-5.5.1 /var/www/webacula
        
        wget -qO- https://packages.zendframework.com/releases/ZendFramework-1.11.5/ZendFramework-1.11.5.tar.gz | tar -xvzf - -C /tmp
        cp -r /tmp/ZendFramework-1.11.5/library/ /var/www/webacula/
        
        sed -i 's/db_user=\"root\"/db_user=\"bacula\"/' /var/www/webacula/install/db.conf
        sed -i 's/, 12);/, 14);/' /var/www/webacula/html/index.php
        
        su -c "cd /var/www/webacula/install/PostgreSql; ./10_make_tables.sh" - postgres
        su -c "cd /var/www/webacula/install/PostgreSql; ./20_acl_make_tables.sh" - postgres
        sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO bacula;" bacula
        sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO bacula;" bacula
        sudo -u postgres psql -c "UPDATE webacula_users SET PWD = MD5('root') WHERE login = 'root';" bacula
        
        sed -i 's/catalog = all/catalog = all, !skipped, !saved/' /etc/bacula/bacula-dir.conf
        
        service bacula-dir restart
        
        sed -i 's/memory_limit = 128M/memory_limit = 32M/' /etc/php/7.0/apache2/php.ini
        sed -i 's/max_execution_time = 30/max_execution_time = 3600/' /etc/php/7.0/apache2/php.ini

        cp /var/www/webacula/install/apache/webacula.conf /etc/apache2/conf-available/
        
        sed -i 's|/usr/share/webacula|/var/www/webacula|' /etc/apache2/conf-available/webacula.conf
        sed -i 's|^/Directory>|<\/Directory>|' /etc/apache2/conf-available/webacula.conf
        sed -i 's|   # Allow from <your network>|   Allow from 192.168.111.0/24|' /etc/apache2/conf-available/webacula.conf
        
        a2enmod rewrite
        a2enconf webacula
        
        systemctl restart apache2
        
        chown -R www-data. /var/www/webacula
        
        sed -i 's|db.adapter = PDO_MYSQL|db.adapter = PDO_PGSQL|' /var/www/webacula/application/config.ini
        sed -i 's|db.config.username = root|db.config.username = bacula|' /var/www/webacula/application/config.ini
        sed -i 's|db.config.password =|db.config.password = bacula|' /var/www/webacula/application/config.ini
        sed -i 's|def.timezone = "Europe/Minsk"|def.timezone = "America/Maceio"|' /var/www/webacula/application/config.ini
        sed -i 's|bacula.sudo = "/usr/bin/sudo"|bacula.sudo = ""|' /var/www/webacula/application/config.ini
        
        chown www-data /sbin/bconsole
        chmod u=rwx,g=rx,o= /sbin/bconsole
        chown www-data /etc/bacula/bconsole.conf
        chmod u=rw,g=r,o= /etc/bacula/bconsole.conf
        chmod u=rwx,g=rwx,o=x /etc/bacula
      SHELL
  end
  
  config.vm.define "fd1", primary: true do |s|
      s.vm.hostname = "bacula-fd1"
      s.vm.network "private_network", ip: "192.168.111.12"
      s.vm.provision "shell", inline: <<-SHELL
        cd /usr/src/bacula-7.0.5
        CFLAGS="-g -Wall" \
        ./configure \
            --bindir=/sbin \
            --libdir=/usr/lib \
            --sysconfdir=/etc/bacula \
            --with-scriptdir=/etc/bacula \
            --with-archivedir=/tmp \
            --with-working-dir=/opt/bacula/working \
            --with-pid-dir=/var/run \
            --with-subsys-dir=/var/run/subsys \
            --mandir=${datarootdir}/man \
            --datadir=/usr/share \
            --with-plugindir=/usr/lib \
            --disable-conio \
            --enable-smartalloc \
            --enable-client-only \
            --disable-afs \
            --disable-acl \
            --enable-xattr \
            --enable-batch-insert

        make install
        make install-autostart-fd

        service bacula-fd start
    SHELL
  end
  
  config.vm.define "fd2", primary: true do |s|
      s.vm.hostname = "bacula-fd2"
      s.vm.network "private_network", ip: "192.168.111.13"
      s.vm.provision "shell", inline: <<-SHELL
        cd /usr/src/bacula-7.0.5
        CFLAGS="-g -Wall" \
        ./configure \
            --bindir=/sbin \
            --libdir=/usr/lib \
            --sysconfdir=/etc/bacula \
            --with-scriptdir=/etc/bacula \
            --with-archivedir=/tmp \
            --with-working-dir=/opt/bacula/working \
            --with-pid-dir=/var/run \
            --with-subsys-dir=/var/run/subsys \
            --mandir=${datarootdir}/man \
            --datadir=/usr/share \
            --with-plugindir=/usr/lib \
            --disable-conio \
            --enable-smartalloc \
            --enable-client-only \
            --disable-afs \
            --disable-acl \
            --enable-xattr \
            --enable-batch-insert

        make install
        make install-autostart-fd

        service bacula-fd start
    SHELL
  end
end
