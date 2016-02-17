*******************************
Ajouter MySQL à notre cluster
*******************************

Comme pour Centreon, nous devons ajouter MySQL au cluster, nous avons deja ajouter les fichiers utiles à son bon fonctionneent sur le DRDB. 
Nous devons juste faire (avec le service MySQL arrêter):

.. code-block:: bash

    # Au cas ou:
    chkconfig mysql off
    service mysql stop

    pcs resource create mysql ocf:heartbeat:mysql user="mysql" group="mysql" \ 
    pid="/var/lib/mysql/mysql.pid" op monitor interval=20s
    pcs constraint colocation add mysql with ClusterIP INFINITY
    pcs constraint order CentreonFS then mysql


