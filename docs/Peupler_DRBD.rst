***************************************
Quoi mettre sur notre nouveau disque ?
***************************************

Nous devons, ici, refléchir à ce que nous allons mettre sur notre nouveau disque qui va voyager entre nos 2 noeuds et qui va permettre de restituer toutes les informations utiles à Centreon et MySQL.

Centreon utilise pas mal de fichiers plats pour fonctionner, normalement j'ai récupéré et fait les essaies mais j'ai pû faire des oublies donc n'hésitez pas à me le remonter.

Dans ce chapitre nous verrons tous les fichiers utiles au bon fonctionnement et le moyen de les deplacer sans que Centreon et MySQL ne s'en appercoivent.

Voici les dossiers/fichiers que nous devrons mettre sur DRBD:

.. code-block:: bash

    /etc/centreon/
    /etc/centreon-broker/
    /etc/centreon-engine/
    /etc/centreon-syslog/

    /etc/httpd/conf.d/10-centreon.conf

    /etc/cron.d/centreon
    /etc/cron.d/centreon-syslog
    /etc/cron.d/centreon-centstorage

    /usr/share/centreon/
    /usr/share/centreon-engine/
    /usr/share/centreon-syslog/

    /var/lib/centreon/
    /var/lib/centreon-broker/
    /var/lib/centreon-engine/
    /var/lib/mysql/

    /var/log/centreon/
    /var/log/centreon-broker/
    /var/log/centreon-engine/
    /var/log/centreon-syslog/

    /usr/lib/nagios/plugins/Aurel/ # C'est le dossier ou je met mes scripts perso


Pour etre lisible, j'ai créer les dossiers avec le chemin exacte de nos dossiers:

.. code-block:: bash

    mkdir -p /mnt/r0/etc/{centreon,centreon-broker,centreon-engine,centreon-syslog,httpd/conf.d,cron.d}
    mkdir -p /mnt/r0/usr/share/{centreon,centreon-engine,centreon-syslog}
    mkdir -p /mnt/r0/var/lib/{centreon,centreon-broker,centreon-engine,mysql}
    mkdir -p /mnt/r0/var/log/{centreon,centreon-broker,centreon-engine,centreon-syslog}
    mkdir -p /plugins/Aurel


Attention pour MySQL, il est préférable de stopper le service avant de faire la suite et de changer le nom du pid:

.. code-block:: bash

    vim /etc/my.cnf.d/centreon.cnf

    #Ajouter a la fin
    pid-file="/var/lib/mysql/mysql.pid"


Ensuite, on copie/efface/lie:

.. code-block:: bash

    cp -Rda /etc/centreon/*         /mnt/r0/etc/centreon
    cp -Rda /etc/centreon-broker/*  /mnt/r0/etc/centreon-broker
    cp -Rda /etc/centreon-engine/*  /mnt/r0/etc/centreon-engine
    cp -Rda /etc/centreon-syslog/*  /mnt/r0/etc/centreon-syslog

    cp -Rda /etc/httpd/conf.d/10-centreon.conf     /mnt/r0/etc/httpd/conf.d/

    cp -Rda /usr/share/centreon/*           /mnt/r0/usr/share/centreon
    cp -Rda /usr/share/centreon-engine/*    /mnt/r0/usr/share/centreon-engine
    cp -Rda /usr/share/centreon-syslog/*    /mnt/r0/usr/share/centreon-syslog

    cp -Rda /var/lib/centreon/*         /mnt/r0/var/lib/centreon
    cp -Rda /var/lib/centreon-broker/*  /mnt/r0/var/lib/centreon-broker
    cp -Rda /var/lib/centreon-engine/*  /mnt/r0/var/lib/centreon-engine
    cp -Rda /var/lib/mysql/*            /mnt/r0/var/lib/mysql

    cp -Rda /var/log/centreon/*         /mnt/r0/var/log/centreon
    cp -Rda /var/log/centreon-broker/*  /mnt/r0/var/log/centreon-broker
    cp -Rda /var/log/centreon-engine/*  /mnt/r0/var/log/centreon-engine
    cp -Rda /var/log/centreon-syslog/*  /mnt/r0/var/log/centreon-syslog

    ###################################################
    # On met les bons droits sur nos dossiers
    chown mysql:mysql                       /mnt/r0/var/lib/mysql
    chown centreon:centreon                 /mnt/r0/var/lib/centreon
    chown centreon-broker:centreon-broker   /mnt/r0/var/lib/centreon-broker
    chown centreon-engine:centreon-engine   /mnt/r0/var/lib/centreon-engine

    chown centreon:centreon                 /mnt/r0/etc/centreon
    chown centreon-broker:centreon-broker   /mnt/r0/etc/centreon-broker
    chown centreon-engine:centreon-engine   /mnt/r0/etc/centreon-engine
    chown syslog:apache                     /mnt/r0/etc/centreon-syslog

    chown syslog:syslog                     /mnt/r0/usr/share/centreon-syslog

    chown centreon:centreon                 /mnt/r0/var/log/centreon
    chown centreon-broker:centreon-broker   /mnt/r0/var/log/centreon-broker
    chown centreon-engine:centreon-engine   /mnt/r0/var/log/centreon-engine
    chown syslog:syslog                     /mnt/r0/var/log/centreon-syslog

    #############################################
    # On efface les dossiers/fichiers
    rm -rf /etc/centreon
    rm -rf /etc/centreon-broker
    rm -rf /etc/centreon-engine
    rm -rf /etc/centreon-syslog

    rm -rf /var/log/centreon
    rm -rf /var/log/centreon-broker
    rm -rf /var/log/centreon-engine
    rm -rf /var/log/centreon-syslog

    rm -rf /usr/share/centreon
    rm -rf /usr/share/centreon-engine
    rm -rf /usr/share/centreon-syslog

    rm -rf /etc/httpd/conf.d/10-centreon.conf

    rm -rf /etc/cron.d/centreon
    rm -rf /etc/cron.d/centreon-syslog
    rm -rf /etc/cron.d/centreon-glpi
    rm -rf /etc/cron.d/centstorage

    rm -rf /var/lib/centreon
    rm -rf /var/lib/centreon-broker/
    rm -rf /var/lib/SAVE_centreon-engine/
    rm -rf /var/lib/centreon-engine/
    rm -rf /var/lib/mysql/

    ####################################
    # On crée nos liens symboliques
    ln -s /mnt/r0/etc/centreon/ /etc/
    ln -s /mnt/r0/etc/centreon-broker/ /etc/
    ln -s /mnt/r0/etc/centreon-engine/ /etc/
    ln -s /mnt/r0/etc/centreon-syslog/ /etc/

    ln -s /mnt/r0/var/log/centreon/ /var/log
    ln -s /mnt/r0/var/log/centreon-broker/ /var/log
    ln -s /mnt/r0/var/log/centreon-engine/ /var/log
    ln -s /mnt/r0/var/log/centreon-syslog/ /var/log

    ln -s /mnt/r0/usr/share/centreon-engine/ /usr/share
    ln -s /mnt/r0/usr/share/centreon-syslog/ /usr/share
    ln -s /mnt/r0/usr/share/centreon/ /usr/share

    ln -s /mnt/r0/etc/httpd/conf.d/10-centreon.conf /etc/httpd/conf.d/

    ln -s /mnt/r0/etc/cron.d/centreon /etc/cron.d/
    ln -s /mnt/r0/etc/cron.d/centreon-glpi /etc/cron.d/
    ln -s /mnt/r0/etc/cron.d/centreon-syslog /etc/cron.d/
    ln -s /mnt/r0/etc/cron.d/centstorage /etc/cron.d/

    ln -s /mnt/r0/var/lib/centreon/ /var/lib
    ln -s /mnt/r0/var/lib/centreon-broker/ /var/lib
    ln -s /mnt/r0/var/lib/centreon-engine/ /var/lib
    ln -s /mnt/r0/var/lib/mysql/ /var/lib


C'est long ;-) Vous pouvez l'utiliser en script pour aller plus vite (pas testé, sauf ``rm`` et ``ln`` sur mon second noeud, donc à vos risques et periles)

Comme dis juste au dessus, j'ai utiliser ``rm`` et ``ln`` sur mon second noeud après avoir fais une bascule de mon cluster pour être sûr que le disque soit bien dispo sur mon noeud.

Pour vérifier, il suffit de faire un ``ls`` sur mon disque:

.. code-block:: bash

    ls /mnt/r0/

Si tous les dossiers précédement créé sur le noeud principal sont présent c'est que tout se passe bien, sinon, bah ...


