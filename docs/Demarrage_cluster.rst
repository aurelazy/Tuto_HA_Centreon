***************************************
Démarrage et vérification du cluster
***************************************

Bon, aintenant que nous avons configuré Corosync et que nous avons vu quelques commandes avec l'outil ``pcs``, nous allons pouvoir démarrer la bête.

Alors, pour se faire, nous avons 2 possibilités:

.. code-block:: bash

    root@CES3-2:~# pcs cluster start CES3-2
    CES3-2: Starting Cluster...

    root@CES3-2:~# pcs cluster start CES3-2-slave
    CES3-2-slave: Starting Cluster...

ou sinon, plus rapide:

.. code-block:: bash

    root@CES3-2:~# pcs cluster start --all


En faisant de la sorte, nous demarrons le cluster manuellement.
Cette façon permet de vérifier un noeud avant qu'il ne rerentre dans le cluster si celui-ci tombe. Dans ce cas, nous pouvons investiguer sur le pourquoi du comment.

Vérifier Corosync
******************

.. code-block:: bash

    root@CES3-2:~# corosync-cfgtool -s
    Printing ring status.
    Local node ID 1
    RING ID 0
        id  = 192.168.56.120
        status  = ring 0 active with no faults

et on fait la même sur le second noeud.


Vérifier Pacemaker
**********************

.. code-block:: bash

    root@CES3-2:~# ps -axf | grep pacemaker
    Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
    25409 pts/0    S+     0:00          \_ grep pacemaker
    2279 ?        S      0:00 pacemakerd
    2285 ?        Ss     0:01  \_ /usr/libexec/pacemaker/cib
    2286 ?        Ss     0:00  \_ /usr/libexec/pacemaker/stonithd
    2287 ?        Ss     0:02  \_ /usr/libexec/pacemaker/lrmd
    2288 ?        Ss     0:00  \_ /usr/libexec/pacemaker/attrd
    2289 ?        Ss     0:00  \_ /usr/libexec/pacemaker/pengine
    2290 ?        Ss     0:01  \_ /usr/libexec/pacemaker/crmd

Bon si tout va bien, (j'espère !!) on peut vérifier le status du cluster:

.. code-block:: bash

    pcs status

Voila, nous en avons fini avec ce paragraphe.
Dans le prochain, nous verrons la mise en place de notre cluster actif/passif.
