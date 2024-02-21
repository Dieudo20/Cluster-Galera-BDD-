# Cluster-Galera-BDD-
# Mise en place d'un cluster Galera pour la gestion de bases de données distribuées

# Installation de MariaDB:
# Exécutez ces instructions sur les deux serveurs Srv1-D12 et Srv2-D12. Commencez par importer la clé GPG publique de MariaDB:

    apt-key adv --fetch-keys https://mariadb.org/mariadb_release_signing_key.pgp

# Créez le fichier /etc/apt/sources.list.d/mariadb.list avec le contenu suivant:

    deb [arch=amd64] http://ftp.igh.cnrs.fr/pub/mariadb/repo/10.5/ubuntu bionic main

# Mettez à jour le cache apt et installez MariaDB:

    apt-get update
    apt-get install mariadb-server galera-4 -y

# Une fois l'installation terminée, on va configurer Galera sur chaque serveur.

# Configuration de Galera:
# je commence par sauvegarder le fichier de configuration original de MariaDB:

    cp /etc/mysql/mariadb.conf.d/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf.orig

# Ensuite, il est nécessaire d'éditez le ficher de configuration /etc/mysql/mariadb.conf.d/50-server.cnf et d'insérez 
# les paramètres suivants en remplaçant les valeurs par défaut si nécessaire:

 # [mysqld]
    binlog_format=ROW
    default_storage_engine=InnoDB
    innodb_autoinc_lock_mode=2
    bind-address=<PRIVATE_IP>

# Galera Provider Configuration
    wsrep_provider=/usr/lib/galera/libgalera_smm.so
    wsrep_cluster_name='my_wsrep_cluster'
    wsrep_cluster_address="gcomm://<PRIVATE_IP_SRV1>,<PRIVATE_IP_SRV2>"

# Galera Synchronization Congiguration
    wsrep_sst_method=rsync

# Galera Node Configuration
    wsrep_node_address='<PRIVATE_IP>'
    wsrep_node_name='<HOSTNAME>'
# Remplacez <PRIVATE_IP_SRV1>, <PRIVATE_IP_SRV2>, <PRIVATE_IP> et <HOSTNAME> par 
# les valeurs appropriées pour chaque machine. Il est important d'utiliser les adresses IP privées dans le cadre de ce scénario.

Par exemple, sur Srv1-D12:

# [mysqld]
    ...
    wsrep_cluster_address="gcomm://10.132.149.151,10.132.149.152"
    wsrep_node_address='10.132.149.151'
    wsrep_node_name='Srv1-D12'
    ...

# Et sur Srv2-D12:

# [mysqld]
    ...
    wsrep_cluster_address="gcomm://10.132.149.151,10.132.149.152"
    wsrep_node_address='10.132.149.152'
    wsrep_node_name='Srv2-D12'
    ...

# Redémarrez MariaDB pour appliquer les changements:

    systemctl restart mysql

# Création de la nouvelle base de données galera_cluster 
# et accordez les droits minimums au compte racine pour celle-ci:

    CREATE DATABASE IF NOT EXISTS galera_cluster CHARACTER SET utf8 COLLATE utf8_general_ci;
    GRANT ALL PRIVILEGES ON galera_cluster.* TO 'root'@'localhost' IDENTIFIED BY '<ROOT_PASSWORD>';
    FLUSH PRIVILEGES;

# Mise en place du cluster Galera:
# je me connecte à MariaDB sur un des serveurs en mode superutilisateur:

mysql -u root -p

# Entrez le mot de passe lorsque prompted.

# Vérifiez l'état du cluster:

show status like 'wsrep%';
