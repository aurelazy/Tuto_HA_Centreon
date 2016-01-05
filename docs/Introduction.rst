********************
Introduction
********************

Pour des raisons professionnelles, j'ai besoin de mettre en place une solution de supervision qui doit être "up" 24h/24 7j/7.
La solution de supervision choisi est Centreon, celle-ci est déjà utilisée et adoptée par les superviseurs. On ne va donc pas changer une équipe qui gagne.
Pour la Haute Disponibilitée, nous partirons sur une solution depuis longtemps approuvé qui est Corosync et Pacemaker avec en plus DRBD pour pouvoir faire du stockage distribuée.
Nous verrons tout cela au cours de ce tutoriel.

Vous pourrez trouver les articles/tutos sur lesquels je me suis basé:

`ClusterLabs <http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Cluster_from_Scratch/_configure_corosync.html>`_

`Encore ClusterLabs <http://clusterlabs.org/doc/en-US/Pacemaker/1.1/html/Pacemaker_Explained/>`_

`DRBD <http://www.dbsysnet.com/2015/09/drbd-sur-debian-6/>`_

`DRBD bis <http://www.dbsysnet.com/2015/09/un-cluster-drbdmysql-avec-heartbeat-sur-debian-7/>`_

`En cas d'erreur DRBD <https://www.guillaume-leduc.fr/recuperer-drbd-de-letat-standalone-unknown.html>`_


Dans ce tuto, nous allons parler de l'installation des outils sur une distribution CentOS (CES3.2, Centreon Enterprise Server), mais pas de Centreon en lui-même (voir mon autre tuto sur le sujet), ensuite nous passerons sur la configuration, les tests, puis nous parlerons de l'installation de DRBD et de sa configuration.

Je vous souhaite une bonne lecture et n'hésitez pas à me remonter toute anomalie sur lesquelles vous pourrez tomber.
