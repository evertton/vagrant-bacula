# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "generic/debian9"
  config.vm.box_check_update = true
  #config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provision "shell", inline: <<-SHELL
    set -x
    apt update -y && apt upgrade -y
    apt install -y nano build-essential
    
    wget -qO- https://master.dl.sourceforge.net/project/bacula/bacula/7.0.5/bacula-7.0.5.tar.gz | tar -xvzf - -C /usr/src
    cd /usr/src/bacula-7.0.5
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
        apt install -y sqlite3 libsqlite3-dev
        
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
        apt install -y postgresql libpq-dev
      
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
        chown -R postgres:postgres /tmp/bacula
        
        su - postgres
          cd /tmp/bacula
          ./create_postgresql_database
          ./make_postgresql_tables
          ./grant_postgresql_privileges
        exit
        
        mkdir /opt/bacula/log
        
        service bacula-dir start
        service bacula-fd start
      SHELL
  end
  
  config.vm.define "fd1", primary: true do |s|
      s.vm.hostname = "bacula-fd1"
      s.vm.network "private_network", ip: "192.168.111.12"
      s.vm.provision "shell", inline: <<-SHELL
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
