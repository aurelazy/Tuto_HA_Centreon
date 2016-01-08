*************************************
Ajouter Centreon à notre cluster
*************************************

Nous allons, dans ce chapitre, ajouter les services qui sont liés à CEntreon dans notre cluster.
La première chose à faire, est de connaître les services qui le font tourner.

Alors, après des années de recherches (et d'études), j'ai trouvé les 3 services qui font que Centreon tourne:

 * cbd
 * centcore
 * centengine

Nous avons également besoin de MySQL (ici MariaDB), mais ceci est développé dans un autre chapitre (il est pas encore fait donc pas de lien encore vers celui-ci).

Bon c'est bien beau de connaître les services de Centreon, mais il doit surement y avoir un lien entre ces services donc une priorité sur le démarrage et l'arrêt.
Et oui, il y a un lien, et bibi à refait des année de recherches la-dessus.

 1. cbd
 2. centcore
 3. centengine


Ce qui serait pas mal aussi, serait de faire un groupe de ces services, ce qui nous permettrait de ne gerer que le groupe et non service par service.
Et oui c'est possible et nous allons voir comment le faire.

Mais d'abord, nous devons arrêter les services et les empêcher de démarrer au démarrage du server, car pacemaker/corosync s'en chargeront pour nous:

.. code-block:: bash

    chkconfig cbd off
    chkconfig centcore off
    chkconfig centengine off
    service cbd stop
    service centcore stop
    service centengine stop

Voila, nos services sont arrêtés et ne demarrerons plus au démarrage.

.. note:: ATTENTION !! L'arret des services est à faire sur les 2 serveurs, sinon les services tournerons toujours sur un serveur même s'il n'est pas le principal !!


Bon maintenant on peut ajouter "Centreon" à notre cluster en créent les ressources:

.. code-block:: bash

    pcs resource create cbd lsb:cbd migration-threshold=2 op monitor interval=30s --group=ClusterCentreon
    pcs resource create centcore lsb:centcore migration-threshold=2 op monitor interval=30s --group=ClusterCentreon
    pcs resource create centengine lsb:centengine migration-threshold=2 op monitor interval=30s --group=ClusterCentreon


Alors voila ici nous utiliserons le type de ressource ``lsb`` car les scripts "init" de Centreon ne sont pas créé pour ``ocf``

Voici une explication de comment doit être configuré un script init ``LSB`` pour pouvoir fonctionner avec pacemaker/corosync:

`Conformité du script init pour LSB <http://clusterlabs.org/doc/en-US/Pacemaker/1.1/html/Pacemaker_Explained/ap-lsb.html>`_

Par défaut, Pacemaker essaie de redemarrer la ressource avant de basculer, donc si vous pensez que l'arrêt d'un service est critique, il vaut mieux ajouter l'option "migration-threshold" lors de notre création et dire à Pacemaker de basculer s'il n'arrive pas à redemarrer après 2 essais.

Ensuite, on voit, comme dans le chapitre précédent, que nous demandons de vérifier toutes les 30 secondes l'état de notre service.
Puis nous ajoutons le service/ressource dans un groupe que nous avons nommé ``ClusterCentreon`` avec l'option "- -group"

Bien sûr, l'ordre dans lequel nous rentrons les ressources dans ce groupe est important ! 
Par défaut, la ressource va se placer à la fin. 
Il est toutefois possible de changer cela avec l'option "- -before":

.. code-block:: bash
    
    pcs resource group add ClusterCentreon cbd --before centcore


Bon là, ça ne fonctionne pas forcément comme on le voudrait.
Si vous faites un petit ``pcs status``, il se pourrait que les ressources soient sur notre "slave" ou qu'elles soient réparties sur les 2.
Pas super !

On va dire que notre ressource principale est notre ``ClusterIP`` et que toutes les autres ressources doivent démarrer sur le noeud où celle-ci est démarré.

Ce qui bien sûr titille notre réflexion sur le faite qu'avec cette régle, nous avons 2 contraintes:

 1. ClusterIP doit démarrer avant toutes les autres ressources
 2. Toutes les ressources doivent être liées à ClusterIP


Lancer les ressources sur le même hôte
==========================================

Voici comment faire pour que nos ressources se lancent sur le même hôte, comme mis plus haut, nous lanceront toutes les ressources sur la même machine que ClusterIP. Ce qui veut dire que si ClusterIP n'est as démarré les autres ressources ne démarrerons pas.

.. code-block:: bash

    root@CES3-2:~# pcs constraint colocation add ClusterCentreon with ClusterIP INFINITY
    root@CES3-2:~# pcs constraint
    Location Constraints:
    Ordering Constraints:
    Colocation Constraints:
      ClusterCentreon with ClusterIP (score:INFINITY


S'assurer de l'ordre de démarrage/arrêt des ressources
========================================================

Comme dis plus haut, ClusterIP doit être la première ressource démarré sur le noeud:

.. code-block:: bash

    root@CES3-2:~# pcs constaint order ClusterIP then ClusterCentreon
    root@CES3-2:~# pcs constraint
    Location Constraints:
    Ordering Constraints:
      start ClusterIP then start ClusterCentreon (kind:Mandatory)
    Colocation Constraints:
      ClusterCentreon with ClusterIP (score:INFINITY

Préférer un noeud plutôt qu'un autre
==========================================

Pacemaker ne va pas par lui même décider qu'elle machine est la mieux pour être l'hôte principal de notre cluster, il va falloir lui dire:

.. code-block:: bash

    root@CES3-2:~# pcs constraint location ClusterCentreon prefers CES3-2=50


Bon bon bon ! J'ai fais un ``pcs status`` et mes ressources sont encore sur "CES3-2-slave" ! POURQUOI ??

Rappelez-vous ! nous avions mis notre "stickiness" à 100, donc avec un score de 50 nous sommes en dessous, nous ne passerons donc pas sur notre noeud préféré automatiquement, et le downtime alors !

Nous devons le faire manuellement.


Basculer les ressources manuellement
=======================================

Bon pour jouer un peu avec les "contraintes", nous allons l'utiliser pour basculer nos ressources, mais sachez que des commandes plus simple et plus rapides sont disponibles (nous les verrons à la fin de ce paragraphe)

Voici comment passer d'un noeud à l'autre:

.. code-block:: bash

    root@CES3-2:~# pcs constaint location ClusterCentreon prefers CES3-2=INFINITY

Faites un ``pcs status`` vous verrez votre cluster basculer.

Qu'est-ce qu'on vient de faire ? On a dit à Pacemaker qu'il fallait préféré CES3-2 comme noeud principal, mais en même temps, on lui dis aussi que nous voulons TOUJOURS que le noeud principal soit CES3-2, pas terrible si nous devons faire du ménage, de la lessive, ..., sur celui-ci.

Bon on va enlever cette contrainte, il faut d'abord la retrouver:

.. code-block:: bash

    root@CES3-2:~# pcs constaint --full
    Location Constraints:
      Resource: ClusterCentreon
        Enabled on: CES3-2 (score: INFINITY) (id:location-ClusterCentreon-CES3-2-INFINITY)
    Ordering Constraints:
      start ClusterIP then start ClusterCentreon (kind:Mandatory) (id:order-ClusterIP-ClusterCentreon-mandatory)
    Colocation Constraints:
      ClusterCentreon with ClusterIP (score:INFINITY) (id:colocation-ClusterCentreon-ClusterIP-INFINITY)


On récupère l'ID de notre contrainte, ici "location-ClusterCentreon-CES3-2-INFINITY" pour l'enlever

.. code-block:: bash

    root@CES3-2:~# pcs constaint remove location-ClusterCentreon-CES3-2-INFINITY


Faites un ``pcs constraint``, moi j'ai la flemme ! Et vous verrez que cette contrainte a disparu. Et notre ressource est toujours sur CES3-2 (``pcs status``)

Ok ! C'est long non ? Et en plus source d'erreurs ? On va faire plus rapide et "safe":


.. code-block:: bash

    root@CES3-2:~# pcs cluster standby CES3-2-slave

Toutes les ressources basculent ! Génial ! Ne pas oublier de faire un:


.. code-block:: bash

    root@CES3-2:~# pcs cluster unstandby CES3-2-slave


Si on ne le fait pas, on risque de faire des dégats si CES3-2 tombe et que CES3-2-slave est encore en standby !


Compatibilité de nos inits
===============================

Alors, en fait là si votre centreon crash, il ne va pas basculer ! Je vous relaisse (ou "laisse" pour les mauvais élèves) lire lecet `article <http://clusterlabs.org/doc/en-US/Pacemaker/1.1/html/Pacemaker_Explained/ap-lsb.html>`_ et faire les tests avec nos 3 scripts init de nos services.
Et mince, les inits ne sont pas compatibles.

Alors nous allons changer quelques lignes de nos services ``cbd`` et ``centcore``.
Les scripts se trouvent dans le dossier ``/etc/init.d/``.

Pour le service ``cbd``:

Il va falloir changer quelques lignes de la fonction "status_all()"

A la ligne 238:

.. code-block:: bash

    cat ${master_file} | grep -v '^#' | grep 'central' | while read line; do
    ...

A la ligne 243 et suivantes:

.. code-block:: bash

    #if [ "${running}" = "y" ]; then
    if [ -f "${pidfile}" ]; then
        status -p ${pidfile} ${cbd}
    else
        status -p ${pidfile} ${cbd}
        return 3
    fi
    done

Pour le service ``centcore``:

A la ligne 43

    Changer return 1 => return 0

A la ligne 94:

.. code-block:: bash

    if ...
        status "$binary"
        exit 3
    else
        ...


