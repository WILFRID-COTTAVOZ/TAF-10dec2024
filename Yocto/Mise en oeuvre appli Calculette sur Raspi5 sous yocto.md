Les lignes qui commencent par "=>" sont les choses à faire pour lancer le démonstrateur. Le reste, c'est des explications.

# Prérequis :

2 cartes Raspi 5 sous yocto avec le Run-Time vsomeip et le package net-tools

NB : Le package net-tools est requis pour ajouter une route multicast (commande : route add ...)


Fichiers requis :

Côté serveur :
/etc/network/interfaces
/etc/vsomeip/vsomeip-udp-service.json
/usr/bin/respinit.sh
/usr/bin/respcalc.sh
/usr/bin/respcalc

Côté client :
/etc/network/interfaces
/etc/vsomeip/vsomeip-udp-client.json
/usr/bin/reqinit.sh
/usr/bin/reqcalc.sh
/usr/bin/reqcalc


# Configuration réseau des carte Raspi 5

La configuration réseau des cartes Raspi 5 est spécifiée dans le fichier /etc/network/interfaces

Son contenu :
******************************
auto lo
iface lo inet loopback

iface eth0 inet static
        address 192.168.7.1   
        netmask 255.255.255.0 
******************************

NB : l'adresse IP est 192.168.7.1/24 sur la carte Raspi serveur et 192.168.7.2/24 sur l'autre.


# Côté serveur

=> NB : pour l'instant, le seul moyen de savoir si on est sur le client ou sur le serveur, c'est de voir quel script d'initialisation est présent.


## Lancement Raspi

(c'est pas encore testé, mais ça devrait marcher si on lance un script sur une raspi avant d'allumer l'autre raspi)
=> login : root
=> MdP : yocto


## Configuration vsomeip

Les fichiers de configuration du RunTime de vsomeip (fichiers .json) sont dans le répertoire /etc/vsomeip

La présence de nombreux fichiers de configuration dans ce répertoire engendre des messages du type "Multiple definition of ..."
Ce répertoire ne doit contenir que le fichier de configuration applicable. Les autres fichiers de configuration peuvent être déplacés dans un sous-répertoire si besoin.

Il faut donc que les raspi soient "spécialisées" : l'une ne fait que serveur et l'autre ne fait que client.

Ce fichier est vsomeip-udp-service.json


## Initialisation et lancement de l'appli Calculette


### Script d'initialisation

Le fichier /usr/bin/respinit.sh initialise le réseau. Il active l'interface eth0 (Ethernet filaire) et désactive l'interface wlan0 (Wifi)
La configuration réseau est spécifiée dans le fichier /etc/network/interfaces

Son contenu :
***************
ifup eth0
ifdown wlan0
***************

=> Executer :
  /usr/bin/respinit.sh
  (c'est normal si ça affiche "interface wlan0 not configured")


### Script de lancement

Le fichier /usr/bin/respcalc.sh 
  - déclare la route pour joindre l'adresse IP de multicast spécifiée pour la découverte des services offerts
  - affecte les variables d'environnement requises
  - execute l'application Calculette

Son contenu :
********************************************************************
route add -nv 224.244.224.245 dev eth0
route
export LD_LIBRARY_PATH=/usr/lib
export VSOMEIP_CONFIGURATION=/etc/vsomeip/vsomeip-udp-service.json
export VSOMEIP_APPLICATION_NAME=service-sample
/usr/bin/respcalc
*********************************************************************

=> Executer :
  /usr/bin/respcalc.sh




# Côté client


## Configuration vsomeip

Les fichiers de configuration du RunTime de vsomeip (fichiers .json) sont dans le répertoire /etc/vsomeip

Le fichier de configuration est vsomeip-udp-client.json


## Initialisation et lancement de l'appli Calculette


### Script d'initialisation

Le fichier /usr/bin/reqinit.sh initialise le réseau. Il active l'interface eth0 (Ethernet filaire) et désactive l'interface wlan0 (Wifi)
La configuration réseau est spécifiée dans le fichier /etc/network/interfaces

Son contenu :
***************
ifup eth0
ifdown wlan0
***************

=> Executer :
  /usr/bin/reqinit.sh
  (c'est normal si ça affiche "interface wlan0 not configured")


### Script de lancement

Le fichier /usr/bin/reqcalc.sh 
  - déclare la route pour joindre l'adresse IP de multicast spécifiée pour la découverte des services offerts
  - affecte les variables d'environnement requises
  - execute l'application Calculette

Son contenu :
********************************************************************
route add -nv 224.244.224.245 dev eth0
route
export LD_LIBRARY_PATH=/usr/lib
export VSOMEIP_CONFIGURATION=/etc/vsomeip/vsomeip-udp-client.json
export VSOMEIP_APPLICATION_NAME=client-sample
/usr/bin/reqcalc
*********************************************************************

=> Executer :
  /usr/bin/reqcalc.sh
