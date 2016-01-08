**********************************
Installer DRBD sur notre CentOS
**********************************

Avant de rentrer dans le vif du sujet, regardons ce qu'est DRBD.

DRBD (Distributed Replicated Block Device) peut-être comparé à un RAID 1 mais en réseau, c'est à dire que 2 disques, partitions, ..., peuvent être repliqué d'un disque à un autre via un réseau ethernet ou fibre.

Cela permet donc d'assurer la disponibilité de vos données en cas de crash complet d'une machine.
Ce que ne permet pas de faire un RAID classique. [#f1]_


Préparation des disques
=============================

Pour chacune de nos machines, nous devons créer un disque dur virtuel.
Si vous voulez juste faire un test vous n'avez pas besoin de beaucoup d'espace, sinon c'est à vous de voir.

C'est fait ? Cool, nous pouvons créer une partition sur nos nouveaux disques.
Mais tout d'abord, commençons par vérifier les disques présent pour ne pas faire de boulette:


.. code-block:: bash

    root@CES3-2:~# df -h

Alors de mon coté, il n'y a que *sda* d'utilisé, donc à priori on peut partir sur *sdb* que l'on peut trouver dans */dev* 

.. code-block:: bash

    root@CES3-2:~# fdisk /dev/sdb
    ...
    Command (m for help): n
    Command action
    e extend
    p primary partition (1-4)
    p
    Partition number (1-4): 1
    First cylinder (1-130, default 1):
    Using default value 1
    Last cylinder, +cylinder or +size{K,M,G} (1-130, default 130):
    Using default value 130
    Command (m for help): w
    The partition table has been altered!
    Calling ioctl() to re-read partition table.
    Syncing disks.

Voila, vous pouvez faire la même chose sur le second noeud (CES3-2-slave).

Nous allons maintenant installer les paquets nécessairepour utiliser DRBD.


Installation et configuration DRBD
====================================

.. note:: Ces commandes seront à faire sur les 2 noeuds !

Il va falloir installer au préalable le dépôt "elrepo":

.. code-block:: bash

    rpm --import http://elrepo.org/RPM-GPG-KEY-elrepo.org
    rpm -Uvh http://elrepo.org/elrepo-release-6-5.el6.elrepo.noarch.rpm

Bien sûr, il faudra changer l'adresse du dépôt en fonction de votre release.

Ensuite, faites un petit:

.. code-block:: bash

    yum update

Puis:

.. code-block:: bash

    yum install drbd84-utils kmod-drbd84

Lorsque les paquets sont installés, lancez la commande suivante:

.. code-block:: bash

    modprobe drbd

Maintenant que les disques et DRBD sont en place nous pouvons configurer la réplication entre les 2 disques.

Nous devons créer un fichier, ``drbd1.res``, dans le dossier ``/etc/drbd.d/``

Avant tout cela, faites un ``hostname`` pour récupérer le nom de votre machine.
Pour moi, ``CES3-2.local`` pour la première, et ``CES3-2-slave.local`` pour la seconde.

Puis on se rend dans ``/etc/drbd.d/`` et l'on crée notre fichier ``drbd1.res``

Voici à quoi doit ressembler notre fichier (ATTENTION remplacez avec vos informations):

.. code-block:: bash

    resource r0 {
        # Taux de transfert 10 M pour 100mbits
        # 100M pour du 1Gbits
        protocol C;
        device /dev/drbd0;
        meta-disk internal;

        startup {
            wfc-timeout 30;
            degr-wfc-timeout 15;
        }
        disk {
            on-io-error detach;
        }
        syncer {
            rate 10M;
        }
        on CES3-2.local {
            disk /dev/sdb1;
            address 192.168.56.120:7788;
        }
        on CES3-2-slave.local {
            disk /dev/sdb1;
            address 192.168.56.121:7788;
        }
    }


l

.. [#f1] `Site de Denis Rosenkranz <http://denisrosenkranz.com/tuto-ha-drbd-sur-debian-6/>`_
