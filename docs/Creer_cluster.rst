****************************************
Création de notre cluster actif/passif
****************************************

Avant de créer notre premier cluster, nous allons vérifier les informations de nos noeuds, savoir s'ils sont connectés entre eux, s'ils sont "online", voir leur configuration, ...

  
.. code-block:: bash

    root@CES3-2:~# pcs status
    Error: cluster is not currently running on this node

Ha mince ! J'ai oublié de démarrer mon cluster

.. code-block:: bash

    root@CES3-2:~# pcs cluster start CES3-2

Je démarre mon noeud principal, et voici ce que va me donner la commande ``pcs status``

.. code-block:: bash

    Cluster name: centreoncluster
    Last updated: Thu Jan  7 11:33:54 2016
    Last change: Wed Jan  6 10:09:52 2016
    Stack: cman
    Current DC: CES3-2 - partition with quorum
    Version: 1.1.11-97629de
    2 Nodes configured
    8 Resources configured


    Online: [ CES3-2 ]
    OFFLINE: [ CES3-2-slave ]


Bon, on voit que nous avons configuré 2 noeuds, puis que nous avons 8 ressources également configurées (vous devez en avoir 0 à ce stade)

Puis on voit bien que notre 1er noeud est en ligne alors que le 2nd ne l'est pas.

Ah oui !

.. code-block:: bash

    root@CES3-2:~# pcs cluster start CES3-2-slave
    CES3-2-slave: Starting Cluster..


.. code-block:: bash

    root@CES3-2:~# pcs status
    ...

    Online: [ CES3-2 CES3-2-slave ]


C'est bon, les 2 sont en ligne.

On peut voir le fichier de configuration écrit en XML depuis la commande:

.. code-block:: bash

    root@CES3-2:~# pcs cluster cib
    <cib admin_epoch="0" cib-last-written="Wed Jan  6 10:09:52 2016" crm_feature_set="3.0.9" \ 
    epoch="132" have-quorum="1" num_updates="54" validate-with="pacemaker-2.0" dc-uuid="CES3-2">
      <configuration>
          <crm_config>
                <cluster_property_set id="cib-bootstrap-options">
                    <nvpair id="cib-bootstrap-options-dc-version" name="dc-version" \
                    value="1.1.11-97629de"/>
                    <nvpair id="cib-bootstrap-options-cluster-infrastructure" \
                    name="cluster-infrastructure" value="cman"/>
                    <nvpair id="cib-bootstrap-options-stonith-enabled" name="stonith-enabled" \
                    value="false"/>
                    <nvpair id="cib-bootstrap-options-no-quorum-policy" name="no-quorum-policy" \ 
                    value="ignore"/>
                    <nvpair id="cib-bootstrap-options-last-lrm-refresh" name="last-lrm-refresh" \ 
                    value="1451914735"/>
                </cluster_property_set>
            </crm_config>
            <nodes>
                <node id="CES3-2-slave" uname="CES3-2-slave">
                    <instance_attributes id="nodes-CES3-2-slave"/>
                </node>
                <node id="CES3-2" uname="CES3-2">
                    <instance_attributes id="nodes-CES3-2"/>
                </node>
            </nodes>
            ...

Bon on vérifie si nous n'avons aucune erreur dans la configuration, c'est pas mal de le faire avant de faire toute modification.

.. code-block:: bash

    root@CES3-2:~# crm_verify -L -V

Bon moi je n'ai auncune erreur, mais normalement vous devriez avoir une (ou plusieurs) erreur concernant le STONITH, que nous n'utiliserons pas ici.

Nous allons le désacvtiver:

.. code-block:: bash

    root@CES3-2:~# pcs property set stonith-enabled=false
    root@CES3-2:~# crm_verify -L

Plus d'erreur ? Tant mieux, sinon bah il va falloir chercher pourquoi ! 

Pour ma part, je n'ai pas besoin de mettre en place un STONITH (Shoot The Other Node In The Head) car si un noeud tombe, il ne reprendra pas la main automatiquement, il faudra le faire manuellement, donc pas de risque.
Bon dans les bonnes pratique c'est préconisé, donc si vous voulez approfondir le sujet je ne vais pas vous en empecher.


Ajouter une ressource
======================

Ouais cool ! On va commencer à jouer !

Notre première ressource sera la création de l'@IP virtuelle (192.168.56.122). 
Celle-ci va se monter sur le noeud principal et basculera en cas de souci.

Alors pour chaque ressource nous devons lui donner un nom, ici nous la nommerons ``ClusterIP``.
Nous devons également lui dire le delai entre chaque vérification, ici nous metterons 30 secondes.

.. code-block:: bash

    root@CES3-2:~# pcs resource create ClusterIP ocf:heartbeat:IPaddr2 ip=192.168.56.122 \ 
    cidr_netmask=24 op monitor interval=30s


Cette ligne est assez compréhensible, "resource create" permet de créer une ressource du nom de "ClusterIP", "ip=" permet de lui assigner une @ip, avec son "cidr_netmask=", "op monitor" on l'a vu plus haut.

Ce qui est très important dans cette commande est l'information ``ocf:heartbeat:IPaddr2``:
 * ocf = le type de script pour la ressource et où la trouver
 * heartbeat = dit dans quel espace de nom de trouve l'OCF
 * IPaddr2 = le nom du script pour notre ressource.

Alors, si vous voulez visualiser les ressources disponibles, vous pouvez lancer les commandes suivantes:

.. code-block:: bash

    root@CES3-2:~# pcs resource standards
    ocf
    lsb
    service
    stonith

Pour afficher la liste des fournisseurs disponibles pour OCF:

.. code-block:: bash
    
    root@CES3-2:~# pcs resource providers
    heartbeat
    linbit
    pacemaker

Finalement, si l'on veut voir tous les agents des ressources disponibles pour un fournisseur OCF:

.. code-block:: bash

    root@CES3-2:~# pcs resource agents ocf:heartbeat
    CTDB
    Delay
    Dummy
    Filesystem
    IPaddr
    IPaddr2
    IPsrcaddr
    LVM
    MailTo
    Route
    SendArp
    Squid
    VirtualDomain
    Xinetd
    apache
    conntrackd
    db2
    dhcpd
    ethmonitor
    exportfs
    iSCSILogicalUnit
    mysql
    named
    nfsnotify
    nfsserver
    nginx
    pgsql
    postfix
    rsyncd
    symlink
    tomcat

Voila en gros les commandes de visualisations que l'on peut adapter bien sûr.
Je vous laisse vous renseigner sur les autres types de ressources surtout ``lsb`` et bien sûr ``ocf``.
De toute façon, nous en reparlerons un peu plus loin lors de l'ajout d'autres ressources.

Alors ! Notre cluster est fonctionnel ou pas ?

.. code-block:: bash

    root@CES3-2:~# pcs status
    Cluster name: centreoncluster
    Last updated: Thu Jan  7 15:30:22 2016
    Last change: Thu Jan  7 14:36:33 2016
    Stack: cman
    Current DC: CES3-2 - partition with quorum
    Version: 1.1.11-97629de
    2 Nodes configured
    8 Resources configured


    Online: [ CES3-2 CES3-2-slave ]

    Full list of resources:

     ClusterIP  (ocf::heartbeat:IPaddr2):   Started CES3-2
    ...

Youhou !! Trop bien, mon @IP virtuelle est démarré sur mon 1er noeud ! Fais voir:

.. code-block:: bash

    root@CES3-2:~# ip addr
    ...
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 08:00:27:40:0f:82 brd ff:ff:ff:ff:ff:ff
        inet 192.168.56.120/24 brd 192.168.56.255 scope global eth1
        inet 192.168.56.122/32 brd 192.168.56.255 scope global eth1:ClusterIP
        inet6 fe80::a00:27ff:fe40:f82/64 scope link 
            valid_lft forever preferred_lft forever

Oh là là ! Je suis trop fort !


Executer un basculement
========================

Bon maintenant pourquoi ne pas essayer un petit basculement de noeud, c'est un peu le but ultime de ce tutoriel, non ?

On va vérifier sur quel noeud est notre @IP virtuelle (``pcs status``), noralement elle doit se trouver sur notre machine "maitre".

Alors soit on stop Pacemaker/Corosync sur la machine:

.. code-block:: bash

    root@CES3-2:~# pcs cluster stop CES3-2
    CES3-2: Stopping Cluster (pacemaker)...

Ce qui donne:

.. code-block:: bash

    root@CES3-2-slave:~# pcs status
    Cluster name: centreoncluster
    Last updated: Thu Jan  7 16:01:08 2016
    Last change: Thu Jan  7 14:36:34 2016
    Stack: cman
    Current DC: CES3-2 - partition with quorum
    Version: 1.1.11-97629de
    2 Nodes configured
    8 Resources configured


    Node CES3-2: standby
    Online: [ CES3-2-slave ]

    Full list of resources:

     ClusterIP  (ocf::heartbeat:IPaddr2):   Started CES3-2-slave

Bien sûr cette commande est à faire sur le 2nd noeud (le premier est stoppé !!)

Soit on fait un standby:

.. code-block:: bash

    root@CES3-2:~# pcs cluster standby CES3-2-slave


Entre temps, j'ai redémarré CES3-2 (mettre "start" à la place de "stop") !! et d'une pierre de coup je repasse la main à ma machine principale.

.. code-block:: bash

    root@CES3-2:~# pcs status
    Cluster name: centreoncluster
    Last updated: Thu Jan  7 16:04:53 2016
    Last change: Thu Jan  7 16:04:23 2016
    Stack: cman
    Current DC: CES3-2-slave - partition with quorum
    Version: 1.1.11-97629de
    2 Nodes configured
    8 Resources configured


    Node CES3-2-slave: standby
    Online: [ CES3-2 ]

    Full list of resources:

     ClusterIP  (ocf::heartbeat:IPaddr2):   Started CES3-2


Pas mal hein ?! allez-y, amusez-vous avec ces commandes ! Ce sont celles que nous utiliserons le plus en prod, ou pas.

Ne pas oublier de sortir CES3-2-slave du standby ! Mais comment ?

.. code-block:: bash

    root@CES3-2:~# pcs cluster unstandby CES3-2-slave

Bah comme ça !

.. note:: Les commandes "stop", "start", "standby", ..., peuvent bien sûr être lancé depuis n'importe quel noeud.

.. note:: Si vous avez plusieurs noeud (au dessus de 2) vous devrez utiliser un "Qurorum", je vous laisse faire vos recherches là-dessus !!

Pour résumer, après l'arrêt d'un noeud sa/ses ressources sont basculées automatiquement sur l'autre noeud. 
Au redémarrage de la machine, la ressource ne revient pas automatiquement sur celle-ci, ce qui est pas mal surtout si la machine contient encore quelques problème et que nous voulons les resoudre avant de lui redonner la main ! 
Par contre, sur d'anciennes version de Pacemaker ce comportement n'est pas automatique !


Prévenir le basculement des ressources après redemarrage
============================================================

Il faut le plus possible éviter de basculer des ressources d'un noeud à l'autre.
Basculer une ressource veut dire que celle-ci sera indisponible pendant un court instant.

Nous allons donc utiliser le principe de "stickiness" qui va dire combien nous preferons rester sur le noeud sur lequel on est.
Ce concept peut-être assez abstrait. Beaucoup de lectures avec du café à gogo et l'esprit libéré peut venir à bout de celui-ci.

Bon on va changer la "viscosité", ou "adhérence", ou soyons fou, le "stickiness" par défaut (on peut le changer aussi sur chaque ressource):

.. code-block:: bash

    root@CES3-2:~# pcs resource defaults resource-stickiness=100
    root@CES3-2:~# pcs resource defaults
    resource-stickiness: 100


