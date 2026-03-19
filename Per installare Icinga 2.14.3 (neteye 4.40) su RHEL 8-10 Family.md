https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/subscription/
https://repo.wuerth-phoenix.com/ubuntu-xenial/neteye-4.40/icinga2-agents/neteye-4.40/

----------------------------------------------------------------------------------------------------------------------------------------------
RHEL 8 FAMILY:
----------------------------------------------------------------------------------------------------------------------------------------------

Per installare Icinga 2.14.3 (neteye 4.40) su RHEL 8.x Family:

sudo timedatectl set-timezone Europe/Rome
sudo timedatectl set-ntp true
yum install cifs-utils -y

Rimuovere eventuali vecchie versioni deprecate:
sudo rpm -e icinga2

Scaricare in locale i pacchetti rpm:

wget https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/centos-8/Packages/i/icinga2-common-2.14.3-1.el8.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/centos-8/Packages/i/icinga2-bin-2.14.3-1.el8.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/centos-8/neteye-4.40-icinga2-agents/Packages/i/icinga2-bin-debuginfo-2.14.3-1.el8.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/centos-8/neteye-4.40-icinga2-agents/Packages/i/icinga2-debuginfo-2.14.3-1.el8.x86_64.rpm \

*Se necessario su rhel 8.x installare:
 sudo dnf install boost-iostreams-1.66.0-13.el8
 sudo dnf install boost-atomic boost-chrono boost-context boost-coroutine boost-date-time 
 sudo dnf install boost-filesystem boost-program-options boost-regex boost-system boost-thread

installare in un unica soluzione:
                
sudo dnf install icinga2*
systemctl daemon-reload

Installare Nagios Plugins (se non presenti):x86_64
Copiare il contenuto di nagios_plugins.zip in /usr/lib64/nagios/plugins

#se non è presente la folder plugins eseguire il seguente comando:
sudo yum install nagios-plugins-all
ls -d /usr/lib*/nagios/plugins

----------------------------------------------------------------------------------------------------------------------------------------------
RHEL 9 FAMILY:
----------------------------------------------------------------------------------------------------------------------------------------------

Per installare Icinga 2.14.3 (neteye 4.40) su RHEL 9.x Family:

sudo timedatectl set-timezone Europe/Rome
sudo timedatectl set-ntp true
yum install cifs-utils -y

Rimuovere eventuali vecchie versioni deprecate:
sudo rpm -e icinga2

Scaricare in locale i pacchetti rpm:
wget https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/subscription/rhel-9/Packages/i/icinga2-2.14.3-1.el9.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/subscription/rhel-9/Packages/i/icinga2-common-2.14.3-1.el9.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/subscription/rhel-9/Packages/i/icinga2-bin-2.14.3-1.el9.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/subscription/rhel-9/Packages/i/icinga2-bin-debuginfo-2.14.3-1.el9.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/icinga2-agents/neteye-4.40/subscription/rhel-9/Packages/i/icinga2-debuginfo-2.14.3-1.el9.x86_64.rpm \

installare in un unica soluzione:
                
sudo dnf install icinga2*
systemctl daemon-reload

Installare Nagios Plugins (se non presenti):
Copiare il contenuto di nagios_plugins.zip in /usr/lib64/nagios/plugins
                
----------------------------------------------------------------------------------------------------------------------------------------------
RHEL 10 FAMILY:
----------------------------------------------------------------------------------------------------------------------------------------------

Per installare Icinga 2.14.3 (neteye 4.40) su RHEL 10 Family:

sudo timedatectl set-timezone Europe/Rome
sudo timedatectl set-ntp true
yum install cifs-utils -y

Rimuovere eventuali vecchie versioni deprecate:
sudo rpm -e icinga2

wget https://repo.wuerth-phoenix.com/ubuntu-xenial/neteye-4.40/icinga2-agents/neteye-4.40/fedora-41/Packages/i/icinga2-2.14.3-1.fc41.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/ubuntu-xenial/neteye-4.40/icinga2-agents/neteye-4.40/fedora-41/Packages/i/icinga2-common-2.14.3-1.fc41.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/ubuntu-xenial/neteye-4.40/icinga2-agents/neteye-4.40/fedora-41/Packages/i/icinga2-bin-2.14.3-1.fc41.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/ubuntu-xenial/neteye-4.40/icinga2-agents/neteye-4.40/fedora-41/Packages/i/icinga2-bin-debuginfo-2.14.3-1.fc41.x86_64.rpm \
wget https://repo.wuerth-phoenix.com/ubuntu-xenial/neteye-4.40/icinga2-agents/neteye-4.40/fedora-41/Packages/i/icinga2-debuginfo-2.14.3-1.fc41.x86_64.rpm \

installare in un unica soluzione:
                
sudo dnf install icinga2*
systemctl daemon-reload
systemctl enable icinga2.service
systemctl start icinga2.service
systemctl status icinga2.service

Installare Nagios Plugins (se non presenti):
Copiare il contenuto di nagios_plugins.zip in /usr/lib64/nagios/plugins oppure in  /usr/lib/nagios/plugins (dipende dalla versione)
chmod 755 /usr/lib/nagios/plugins/*
/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /
/usr/lib/nagios/plugins/check_disk -h

https://github.com/Damocle77/Opensearch.git
