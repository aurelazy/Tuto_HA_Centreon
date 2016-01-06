*****************************************
Les outils de notre cluster
*****************************************


Les outils existant
=====================


Il existe 2 outils pour administrer notre solution:
* ``pcs``
* ``crmsh``
 
Nous utiliserons pcs dans ce tutoriel.
 
Ces outils vont permettre d'administrer et configurer facilement notre cluster avec de simples lignes de commande sans entrer dans les fichiers de configuration.
Il existe également une interface graphique mais qui ne sera pas vu ici, je vous laisse faire vos recherches si vous êtes interressé.


Utilisation de pcs
====================

Nous allons voir quelques commandes utiles à l'administration et à la configuration de notre cluster.
Ne rentrez pas les commandes qui vont suivre, car ce paragraphe est plus pensé comme un "pense bête" qu'autre chose.
Je me suis dis, "autant le mettre à ce moment là, comme on en parle !".

Alors la première commande que l'on peut faire pour se familiariser avec ``pcs`` est la suivante:

.. code-block:: bash

    # pcs

    Usage: pcs [-f file] [-h] [commands]...
    Control and configure pacemaker and corosync.

    Options:
        -h, --help  Display usage and exit
        -f file     Perform actions on file instead of active CIB
        --debug     Print all network traffic and external commands run
        --version   Print pcs version information

    Commands:
        cluster     Configure cluster options and nodes
        resource    Manage cluster resources
        stonith     Configure fence devices
        constraint  Set resource constraints
        property    Set pacemaker properties
        acl         Set pacemaker access control lists
        status      View cluster status
        config      View and manage cluster configuration


Donc avec cette comande, on peut voir les possibilités que nous offre ``pcs``.
Je ne vais pas approfondir toutes les commandes que l'on voit ci-dessus.
Nous allons "survoler" les commandes suivantes:
* cluster
* resource
* constraint
* status

Je met "survoler" car je ne vais pas rentrer dans les tréfonds de ce sytème, ce tutoriel permet de faire un cluster fonctionnel rapidement et la partie cutomisation n'est pas traité.

La commande ``- -help`` ( ou ``-h``) permet d'avoir beaucoup d'informations sur chaque commande, par exemple, voyons le resultat de la commande ``status``:

.. code-block:: bash

    root@CES3-2:~# pcs status help

    Usage: pcs status [commands]...
    View current cluster and resource status
    Commands:
        [status] [--full]
                View all information about the cluster and resources (--full provides more details)

        resources   
            View current status of cluster resources

        groups
            View currently configured groups and their resources

        cluster
            View current cluster status

        corosync
            View current membership information as seen by corosync

        nodes [corosync|both|config]
            View current status of nodes from pacemaker. If 'corosync' 
            is specified, print nodes currently configured in corosync, if 'both'
            is specified, print nodes from both corosync & pacemaker.  If 'config'
            is specified, print nodes from corosync & pacemaker configuration.

        pcsd <node> ...
            Show the current status of pcsd on the specified nodes

        xml
            View xml version of status (output from crm_mon -r -1 -X)


Connaitre le status de notre cluster et de nos ressources
-----------------------------------------------------------

.. code-block:: bash

    root@CES3-2:~# pcs status
    Cluster name: centreoncluster
    Last updated: Wed Jan  6 10:52:38 2016
    Last change: Mon Jan  4 16:41:05 2016
    Stack: cman
    Current DC: CES3-2 - partition with quorum
    Version: 1.1.11-97629de
    2 Nodes configured
    8 Resources configured


    Online: [ CES3-2 CES3-2-slave ]

    Full list of resources:

     ClusterIP  (ocf::heartbeat:IPaddr2):   Started CES3-2 
     Resource Group: ClusterCentreon
            cbd  (lsb:cbd):             Started CES3-2 
            centcore    (lsb:centcore): Started CES3-2 
            centengine (lsb:centengine):   Started CES3-2 
     Master/Slave Set: MySQLDataClone [MySQLData]
            Masters: [ CES3-2 ]
            Slaves: [ CES3-2-slave ]
     MySQLFS    (ocf::heartbeat:Filesystem):    Started CES3-2 
     mysql (ocf::heartbeat:mysql): Started CES3-2 


Basculement de ressource d'un noeud à l'autre
--------------------------------------------------

.. code-block:: bash

    pcs resource move <nom_de_la_resource> <node>
    pcs resource move ClusterIP CES3-2

Demarrer le cluster
-------------------------

.. code-block:: bash

    pcs cluster start --all
    pcs cluster start CES3-2

Creation d'un ressource
-----------------------------

.. code-block:: bash

    pcs resource create cbd lsb:cbd migration-threshold=2 op monitor interval=30s --group=ClusterCentreon


``migration-threshold`` est le nombre de fois qu'il va essayer de redémarrer la ressource avant de basculer sur l'autre noeud.

Démarrage des ressources sur le même noeud
------------------------------------------------

.. code-block:: bash

    pcs constraint colocation add cbd with ClusterIP INFINITY

pour voir les changements:

.. code-block:: bash

    pcs constraint

Choisir l'ordre du démarrage des ressources
-----------------------------------------------

.. code-block:: bash

    pcs constraint order ClusterIP then cbd

Nettoyage du "Failed actions"
-----------------------------------

"Failed actions" arrive suite à une interruption de service entre les 2 noeuds, après ça, le cluster fonctionne moins bien ;-)

.. code-block:: bash

    pcs resource cleanup cbd

Basculer d'un noeud à l'autre
-------------------------------

.. code-block:: bash

    pcs cluster standby CES3-2-slave

puis ne pas oublier de le redémarrer:

.. code-block:: bash

    pcs cluster unstandby CES3-2-slave


Mettre à jour une ressource
--------------------------------

.. code-block:: bash

    pcs resource update cbd ...

Ajouter une ressource à un groupe
-------------------------------------

.. code-block:: bash

    pcs resource group add ClusterCentreon cbd --before centcore

Donc, dans cette commande le groupe se nomme ``ClusterCentreon`` et on place le service ``cbd`` avant ``centcore``.

Par défaut, lorsque l'on ajoute un service à un groupe, celui-ci ce place à la fin.







