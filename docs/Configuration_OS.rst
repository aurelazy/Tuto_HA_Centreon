***************************
Configurer notre OS
***************************

.. |prompt1| replace:: [root@CES3-2]# 
.. |prompt2| replace:: [root@CES3-2-slave]# 
.. |ces1| replace:: CES3-2
.. |ces2| replace:: CES3-2-slave
.. |ip1| replace:: 192.168.56.120
.. |ip2| replace:: 192.168.56.121
.. |ipvirt| replace:: 192.168.56.122

***************************
Configurer notre OS
***************************

Tout d'abord, nous allons faire un petit point sur l'architecture.

Pour ce projet, nous partirons sur un cluster actif/passif sur 2 noeuds:
* Le premier noeud sera **CES3-2** avec pour @IP **192.168.56.120**
* Le second noeud sera **CES3-2-slave** avec pour @IP **192.168.56.121**

Nous voulons que notre supervision soit accessible depuis @IP virtuelle **192.168.56.122**

Nous supposons que les 2 machines peuvent "parler" entre elles.


Premiers pas
=================

Comme d'habitude, nous allons faire une petite mise à jour de nos dépots:


.. parsed-literal::

    |prompt1| yum update


.. parsed-literal::

    |prompt2| yum update


Ok, nos 2 noeuds sont à jour, nous allons vérifier le "hostname" de celle-ci (à faire sur les 2 noeuds:

.. parsed-literal::

    |prompt1| uname -u

Ce qui me donne 

.. code-block:: bash

    CES3-2.local

et pareil sur le 2nd mais avec CES3-2-slave.

Maintenant nous devons faire en sorte que nos 2 noeuds puissent communiquer grâce à leur nom:

.. parsed-literal::

    |prompt1| vim /etc/hosts

Et faire en sorte que les 2 dernières lignes soient au moins identique à ça:

.. code-block:: bash

    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

    192.168.56.120 CES3-2.local CES3-2
    192.168.56.121 CES3-2-slave.local CES3-2-slave

A faire sur les 2 encore une fois !

On vérifie:

.. parsed-literal::

    |prompt1| ping -c 3 |ces2|

Ok, tout fonctionne ! On va pouvoir continuer.

Vous pouvez à ce niveau créer une clé SSH pour pouvoir discuter plus facilement entre vos 2 noeuds, je vous laisse le plaisir de chercher ou de pas, si vous savez deja le faire.


Installation des paquets
===========================

Nous allons maintenant rentrer dans le vif du sujet et partir sur l'installation des paquets necessaires à notre cluster. Ces commandes sont à faire sur les 2 noeuds.

.. code-block:: bash 

    # yum install -y pacemaker pcs psmisc policycoreutils-python cman


Lorsque tous les paquets sont installés, nous devons dire au système de lancer le service pcsd au demarrage de nos machines.

.. code-block:: bash

    # chkconfig pcsd --add
    # service pcsd start

Lors de l'installation, l'utilisateur ``hacluster`` sera créé.
Nous devons lui ajouter un mot de passe sur les 2 noeuds:

.. code-block:: bash

    # passwd hacluster


.. note:: A partir de maintenant et jusqu'à nouvel ordre, nous ferons les commandes sur 1 noeuds.


Configurer Corosync
=======================

Nous devons dire à Corosync de s'authentifier avec l'utilisateur ``hacluster``

.. parsed-literal::
    
    |prompt1| pcs cluster auth |ces1| |ces2|
    Username: hacluster
    Password:
    |ces1|: Authorized
    |ces2|: Authorized

Ensuite, il faut générer et synchroniser la configuration:

.. parsed-literal::

    |prompt1| pcs cluster setup --name ``mycluster`` |ces1| |ces2|

Dans l'option "--name" vous mettez ce que vous voulez comme "CentreonCluster" par exemple.

.. note:: Alors pour ma part, j'ai eu une erreur à ce moment là:
    
    .. code-block:: bash

        Error connecting to <node> - (HTTP error: 500)
        Error : Unable to set cluster.conf

    il suffit de créer le dossier "cluster":

    .. code-block:: bash

        mkdir /etc/cluster







