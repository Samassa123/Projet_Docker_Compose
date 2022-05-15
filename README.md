# Projet_Docker_Compose

#Objectif du projet  
le projet consiste à mettre en place un docker compose qui va etre constitué d'un client , d'un serveur et d'un firewall et de faire interagir ces 3 elements entre eux.

#Installation du docker
Sur notre machine virtual debian on installe le docker selon les etapes suivantes:

#Installation docker sur la machine virtuelle:

sudo apt-get install docker-ce

#Mise en place du conteneur sur debian:

docker container run debian cmd

#Verification de l'installation de docker:

docker run top

#Installation de docker compose sur le debian:

#Téléchargez le binaire Docker Compose dans le /usr/local/bin avec wget ou curl :

sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#Utilisez chmod pour rendre l'exécutable binaire Compose:

sudo chmod +x /usr/local/bin/docker-compose

#Pour vérifier l'installation, utilisez la commande suivante qui imprime la version Composer:

docker-compose --version

#La sortie ressemblera à ceci:

docker-compose version 1.23.1, build b02f1306 

#Construction du projet :

Une fois qu'on a installé docker compose dans laquelle on allait placé nos containers maintenant on va creer un site web(serveur) en python qui contiendra une phrase et qui va etre récupérée par un programme (client) en python

On aura:

-un fichier docker-compose.yml qui contiendra les instructions nécessaires à la creation des differents services

-Un conteneur serveur qui contiendra des fichiers necessaires à la mise en place du serveur

-Un conteneur client qui contiendra les fichiers nécessaires à la mise en place du client

-Un conteneur firewall qui contiendra les fichiers necessaires à la mise en place de notre pare feu

#Contenu de notre serveur:

-Un fichier server.py qui contiendra le code du serveur

-Un fichier index.html qui contiendra la phrase à afficher

-Un fichier Dockerfile qui contiendra les instructions nécessaires pour créer l'environnement du serveur

fichier server.py:

#!/usr/bin/env python3

import http.server

import socketserver

#Cette variable va gérer les requêtes de notre client sur le serveur:

handler = http.server.SimpleHTTPRequestHandler

#Ici nous définissons que nous voulons démarrer le serveur sur le port 1234:

with socketserver.TCPServer(("", 1234), handler) as httpd :

#Cette instruction va maintenir le serveur en fonctionnement, en attendant les requêtes du client:

httpd.serve_forever()

#Contenu de notre fichier index.html:

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Docker Compose</title>
</head>
<body>
    <h1>j'aime Docker Compose</h1>
</body>
</html>

#On créera ensuite un Dockerfile qui sera en charge de l'execution de notre fichier Python

#Nous écrivons 'python' pour le nom de l'image et 'latest' pour la version:

FROM python:latest

#Nous plaçons les fichiers dans le dossier '/server/' de l'image.

ADD server.py /server/
ADD index.html /server/

'WORKDIR'

#Cette commande change le répertoire de base de votre image.

#Ici nous définissons '/server/' comme répertoire de base (où toutes les commandes seront exécutées).

WORKDIR /server/

#Contenu de notre conteneur client:

#Un fichier client.py qui contiendra le code client)

#Un fichier Dockerfile qui contiendra les instructions nécessaires pour créer l'environnement du client

#Contenu de notre fichier client.py:

import urllib.request

fp = urllib.request.urlopen("http://localhost:1234/")

#encodedContent correspond à la réponse du serveur encodée (index.html).

encodedContent = fp.read()

#decodedContent correspond à la réponse du serveur décodée (ce que nous voulons afficher)

decodedContent = encodedContent.decode("utf8")

#Affiche le fichier du serveur : 'index.html'

print(decodedContent)

#Fermez la connexion au serveur.
fp.close()

#On créera ensuite un Dockerfile qui sera en charge de l'execution de notre fichier Python

FROM python:latest

ADD client.py /client/

WORKDIR /client/

#Contenu de notre conteneur firewall:

Fichier clean.sh : 

#!/bin/bash

yes | apt update

yes | apt-get install iptables

#On affiche nos chaines initiales

iptables -L

#On efface les configurations dans les chaines INPUT, OUTPUT, FORWARD

iptables -F INPUT

iptables -F OUTPUT

iptables -F FORWARD

#On définit les polices INPUT, OUTPUT, FORWARD en ACCEPT

iptables -P INPUT ACCEPT

iptables -P OUTPUT ACCEPT

iptables -P FORWARD ACCEPT

Fichier firewall.sh

#Autorise tout le trafic de loopback (lo0) et supprime tout le trafic vers 127/8 qui n'utilise pas lo0

iptables -A INPUT -i lo -j ACCEPT

iptables -A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT

#Accepte toutes les connexions entrantes établies

iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#Autorise tout le trafic sortant

iptables -A OUTPUT -j ACCEPT

#Autorise les connexions HTTP et HTTPS de n'importe où (les ports normaux pour les sites Web)iptables -A INPUT -p tcp --dport 80 -j ACCEPT

iptables -A INPUT -p tcp --dport 443 -j ACCEPT

#Autorise les connexions SSH

iptables -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

#Autoriser les pings

iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

#Rejeter tous les autres messages entrants - refus par défaut :

iptables -A INPUT -j REJECT

iptables -A FORWARD -j REJECT

#Pour permettre aux noeuds du LAN avec des adresses IP privées de communiquer avec les réseaux public externes

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
 

#Retransmettre des requêtes HTTP à notre système de Serveur HTTP 

iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 1234 -j DNAT --to 172.17.0.2:1234

#Pour garder le container actif

while true
do
        sleep 1
done

Fichier Dockerfile (firewall) :

FROM python:latest

ADD clean.sh /firewall/

ADD firewall.sh /firewall/

WORKDIR /firewall/

#Nous allons maintenant creer notre fichier docker-compose.yml:

#Nous utilisons "3" car c'est la dernière version pour le moment

version : "3"

#Vous devez savoir que docker-compose fonctionne avec des services

#1 service = 1 conteneur

#Nous utilisons le mot clé 'services' pour commencer à créer des services

serveur :

#Le mot clé "build" vous permettra de définir le chemin vers le Dockerfile à utiliser pour créer l'image qui vous permettra d'exécuter le service

build : server/

#La commande suivante exécutera "python ./server.py"

commande : python ./server.py

#Récupérer le port 1234 dans le conteneur (car c'est sur ce port que nous diffusons le serveur)
ports :

    - 1234:1234

#Deuxième service (conteneur) : le client

#Nous utilisons le mot clé 'client' pour le serveur

client : 

build : client/

#La commande suivante exécutera "python ./client.py"

commande : python ./client.py

#Le mot clé 'network_mode' est utilisé pour définir le type de réseau donc nous définissons que le conteneur peut accéder à 'localhost' de l'ordinateur

network_mode : host

 #Le mot clé 'depends_on' permet de définir si le service doit attendre que d'autres services soient prêts avant de se lancer. Ici, nous voulons que le service 'client' attende que le service 'serveur' soit prêt

depends_on :

    - serveur

firewall:

    build: firewall/

    container_name: Firewall

    cap_add:

      - NET_ADMIN

    sysctls:

      - net.ipv4.ip_forward=1

    command: >

      bash -c "./clean.sh && ./firewall.sh"

    depends_on:

      - client

#Après la configuration de tous les conteneur , on lance avec les commandes suivantes qui lancera nos 3 containeurs:

docker-compose build

docker-compose up



