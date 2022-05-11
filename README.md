# Projet_Docker_Compose

#Objectif du projet  
le projet consiste à mettre en place un docker compose qui va etre constitué d'un client , d'un serveur et d'un firewall et de faire interagir ces 3 elements entre eux.

#Installation du docker
Sur notre machine virtual debian on installe le docker selon les etapes suivantes:
Installation docker sur la machine virtuelle:
sudo apt-get install docker-ce

Mise en place du conteneur sur debian
 docker container run debian cmd

Verification de l'installation de docker
docker run top

#Installation de docker compose sur le debian
Téléchargez le binaire Docker Compose dans le /usr/local/bin avec wget ou curl :
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
Utilisez chmod pour rendre l'exécutable binaire Compose:
sudo chmod +x /usr/local/bin/docker-compose

Pour vérifier l'installation, utilisez la commande suivante qui imprime la version Composer:
docker-compose --version
La sortie ressemblera à ceci:
docker-compose version 1.23.1, build b02f1306 



