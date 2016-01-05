***********************
Configurer notre OS
************************

# Création d'alias:
.. |prompt1| replace:: [root@CES3-2]# 
.. |prompt2| replace:: [root@CES3-2-slave]# 



Tout d'abord, nous allons faire un petit point sur l'architecture.

Pour ce projet, nous partirons sur un cluster actif/passif sur 2 noeuds:
* Le premier noeud sera **CES3-2** avec pour @IP **192.168.56.120**
* Le second noeud sera **CES3-2-slave** avec pour @IP **192.168.56.121**

Nous voulons que notre supervision soit accessible depuis @IP virtuelle **192.168.56.122**

Nous supposons que les 2 machines peuvent "parler" entre elles.


Installer les paquets
**********************

Comme d'habitude, nous allons faire une petite mise à jour de nos dépots:

.. code-block:: bash

    |prompt1| yum update



.. code-block:: bash

    |prompt2| yum update



