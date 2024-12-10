Install Debian 12.6 avec gnome, serveur SSH, serveur web, utilitaires avec langue & clavier français, utilisateur rescue

Executer :
  login (rescue)
  su
  apt-get install git


# Télécharger yocto

Executer :
  cd /home/rescue
  mkdir yocto
  cd yocto
  git clone git://git.yoctoproject.org/poky


# Générer l'environnement yocto
 
Executer :
  cd poky
  mkdir BUILD
  source ./oe-init-build-env BUILD


# Générer une 1ere image yocto

Executer :
  bitbake core-image-minimal

Pb : locale en_US.UTF-8 manquant
=>
Executer :
   locale    //Pour voir la liste des locales installées
   sudo locale-gen "en_US.UTF-8"
   sudo dpkg-reconfigure locales


Executer :
  bitbake core-image-minimal

Pb : tools missing (as specified by HOSTTOOLS)
ar as chrpath diffstat g++ gawk gcc ld lz4c make nm objcopy objdump ranlib readelf rpcgen strings strip wget

Executer :
  sudo apt install g++
  sudo apt install chrpath
  sudo apt install diffstat
  sudo apt install gawk
  sudo apt install make
  sudo apt install wget
  sudo apt install lz4


Executer :
  bitbake core-image-minimal

Pb : Do not use bitbake as root !!

Executer :
  exit
  cd /home/rescue/yocto/poky
  chown -R rescue *
  cd BUILD
  ../bitbake/bin/bitbake core-image-minimal


>>>> LONGUE COMPILATION


# Installer Qemu

Executer :
  su
  apt install qemu-kvm 
  apt install libvirt-daemon-system
  apt install bridge-utils
  apt install virt-manager


# Tester le lancement de Qemu

Dans le panel des applications cliquer sur l'icone "Gestionnaire de machines virtuelles"
Vérifier que Qemu se lance
Quitter Qemu


# Lancer l'image générée dans Qemu

L'image générée est exploitée par Qemu pour générer une machine virtuelle au moyen du script runqemu.
Le script runqemu est inclus dans yocto.

Rajouter le path pour bitbake et runqemu dans $PATH
Executer :
  export PATH=$PATH:/home/rescue/yocto/poky/bitbake/bin:/home/rescue/yocto/poky/scripts

Rajouter ce path dans le profil de l'utilisateur rescue
Executer :
  cd 
  vi .profile
  Saisir (en fin de ce fichier) :
    export PATH=$PATH:/home/rescue/yocto/poky/bitbake/bin:/home/rescue/yocto/poky/scripts  
  source .profile

Executer :
  cd /home/rescue/yocto/poky/BUILD/tmp/deploy/images/qemux86-64
  su
  runqemu

=> Une console terminal s'ouvre, boot, login
login : root (pas de mot de passe)


## Configurer le clavier en AZERTY sur la VM Qemu

Pb : le clavier est en QWERTY


### Ajouter la recette kbd

Executer :
  login rescue (mdp = yocto)
  cd ~/yocto/poky/meta
  find . -name 'kbd*.bb'
    => ./recipes-core/kbd/kbd_2.6.4.bb
  
  vi ~/yocto/poky/BUILD/conf/local.conf
  Saisir (en fin de fichier) :
    IMAGE_INSTALL:append = " kbd"
    
  cd /home/rescue/yocto/poky/BUILD
  bitbake core-image-minimal


>>>> COMPILATION INCREMENTALE avec kbd_2.6.4 (et dépendances) en complément


Executer :
  cd /home/rescue/yocto/poky/BUILD/tmp/deploy/images/qemux86-64
  su
  runqemu

=> Une console terminal s'ouvre, boot, login

Executer :
  login : root (pas de mot de passe)
  loadkeys fr.keymap

Pb : le fichier fr.keymap n'est pas trouvé

Executer :
  find / | grep fr.keymap
    => le fichier n'existe pas 


### Récupérer le fichier fr.keymap

Il faut transferer le fichier fr.keymap sur la VM Qemu

Pb : ssh n'est pas installé sur la VM Qemu


>>>> Sur PC host : Transfert du fichier fr.keymap (fourni par Romain) vers le PC host dans /home/rescue/Téléchargements


#### Installer ssh sur la VM Qemu

Executer :
  cd ~/yocto/poky/meta
  find . -name '*ssl*.bb'
    => ./recipes-connectivity/openssl/openssl_3.3.1.bb
    
  find . -name '*ssh*.bb'
    => ./recipes-support/libssh2/libssh2_1.11.0.bb
    => ./recipes-connectivity/openssh/openssh_9.9p1.bb
    => ./recipes-connectivity/ssh-pregen-hostkeys/ssh-pregen-hostkeys_1.0.bb
    => ./recipes-core/packagegroups/packagegroup-core-ssh-openssh.bb
    => ./recipes-core/packagegroups/packagegroup-core-ssh-dropbear.bb


Executer :
  vi ~/yocto/poky/BUILD/conf/local.conf
  Saisir (en fin de fichier) :
    IMAGE_INSTALL:append = " openssl libssh2 openssh ssh-pregen-hostkeys packagegroup-core-ssh-openssh kbd"

NB : packagegroup-core-ssh-openssh.bb et packagegroup-core-ssh-dropbear.bb se sont avérés incompatibles à la compilation
En conséquence packagegroup-core-ssh-dropbear.bb n'est pas pris en compte

Executer :
  cd /home/rescue/yocto/poky/BUILD
  bitbake core-image-minimal


>>>> COMPILATION INCREMENTALE avec openssl_3.3.1 libssh2_1.11.0 openssh_9.9p1 ssh-pregen-hostkeys_1.0 packagegroup-core-ssh-openssh (et dépendances) en complément


Executer :
  cd /home/rescue/yocto/poky/BUILD/tmp/deploy/images/qemux86-64
  su
  runqemu

=> Une console terminal s'ouvre, boot, login

Executer :
  login : root (pas de mot de passe)
  ssh -l rescue <@IP host>           // 192.168.7.1, génération des clés SHA256, ssh OK


#### Transferer le fichier fr.keymap vers la VM Qemu

>>>> Sur PC host : Transfert du fichier fr.keymap (fourni par Romain) vers la VM Qemu

     Executer :
       scp /home/rescue/Téléchargements/fr.keymap root@<@IP VM Qemu>:/usr/share


#### Activer le fichier fr.keymap sur la VM Qemu

>>>> Sur la VM Qemu

Executer :
  cd /usr/share
  loadkeys fr.keymap       // Test clavier FR OK

//La commande doit être executée à chaque boot. Elle est donc ajoutée au profil root dans le fichier /home/root/.profile
Executer :
  vi ~/.profile  (nouveau fichier)
  Saisir :
    loadkeys /usr/share/fr.keymap
  
  chmod a+x ~/.profile


# Installer git sur la VM Qemu

Executer :
  cd ~/yocto/poky/meta
  find . -name '*git*.bb'
    => ./recipes-devtools/git/git_2.46.1.bb

  vi ~/yocto/poky/BUILD/conf/local.conf
  Saisir (en fin de fichier) :
    IMAGE_INSTALL:append = " git openssl libssh2 openssh ssh-pregen-hostkeys packagegroup-core-ssh-openssh kbd"


  cd /home/rescue/yocto/poky/BUILD
  bitbake core-image-minimal


>>>> COMPILATION INCREMENTALE avec git_2.46.1 (et dépendances) en complément

Executer :
  cd /home/rescue/yocto/poky/BUILD/tmp/deploy/images/qemux86-64
  su
  runqemu

=> Une console terminal s'ouvre, boot, login
Executer :
  login : root (pas de mot de passe)
  git          // git OK


# Installer l'environnement de dev (gcc, g++, make, ...) sur la VM Qemu

Executer :
  vi ~/yocto/poky/BUILD/conf/local.conf

Par défaut la ligne 149 contient EXTRA_IMAGE_FEATURES ?= "debug-tweaks "

Executer :
  Saisir (complèter cette ligne 149 du fichier) :
    EXTRA_IMAGE_FEATURES ?= "debug-tweaks tools-sdk "


Executer :
  cd /home/rescue/yocto/poky/BUILD
  bitbake core-image-minimal


>>>> COMPILATION INCREMENTALE avec la feature tools-sdk en complément


Executer :
  cd /home/rescue/yocto/poky/BUILD/tmp/deploy/images/qemux86-64
  su
  runqemu

=> Une console terminal s'ouvre, boot, login

Executer :
  login : root (pas de mot de passe)
  gcc          // gcc OK
  g++          // g++ OK
  make          // make OK

Pb : gcc et g++ sont en version 14.2.0 ! Il faut la version 11.4 pour compiler vsomeip


# Télécharger les fichiers source de vsomeip par git sur la VM Qemu

Executer :
  useradd rescue
  passwd rescue (mdp = yocto)
  cd /home/rescue
  git clone https://github.com/COVESA/vsomeip

Pb : nom d'hôte github.com non résolu. 

Executer :
  nslookup github.com 
    => Server : 127.0.0.1
    (...)

Cause : aucun DNS configuré

Solution : rajouter le host github.com dans le fichier /etc/hosts

Executer :
  nslookup github.com 1.1.1.1     //solliciter le DNS 1.1.1.1 pour la résolution du nom github.com
  => (...) Address 1: 140.82.121.3 (...) 

  vi /etc/hosts
  Saisir (en fin de fichier) :
    140.82.121.3   github.com

  git clone https://github.com/COVESA/vsomeip


## Compiler la librarie vsomeip sur la VM Qemu

Création des répertoire de build et tentative de génération de vsomeip

Executer :
  cd /home/rescue/vsomeip
  mkdir build
  cd build
  cmake ..

Pb : cmake n'existe pas


### Installer cmake sur la VM Qemu

Executer :
  cd ~/yocto/poky/meta
  find . -name 'cmake*.bb'
    => ./recipes-devtools/cmake/cmake-native_3.30.2.bb
    => ./recipes-devtools/cmake/cmake_3.30.2.bb

vi ~/yocto/poky/BUILD/conf/local.conf
Saisir (en fin de fichier) :
  IMAGE_INSTALL:append = " cmake git openssl libssh2 openssh ssh-pregen-hostkeys packagegroup-core-ssh-openssh kbd"

cd /home/rescue/yocto/poky/BUILD
bitbake core-image-minimal


>>>> COMPILATION INCREMENTALE avec cmake_3.30.2 (et dépendances) en complément


cd /home/rescue/yocto/poky/BUILD/tmp/deploy/images/qemux86-64
su
runqemu

=> Une console terminal s'ouvre, boot, login
login : root (pas de mot de passe)

cd /home/rescue/vsomeip/build
cmake ..

Pb : la librairie boost n'existe pas sur la VM Qemu


### Installer la librairie boost sur la VM Qemu

cd ~/yocto/poky/meta
find . -name '*boost*.bb'
  => ./recipes-support/boost/boost-build-native_1.86.0.bb
  => ./recipes-support/boost/boost_1.86.0.bb

vi ~/yocto/poky/BUILD/conf/local.conf
Saisir (en fin de fichier) :
  IMAGE_INSTALL:append = " boost cmake git openssl libssh2 openssh ssh-pregen-hostkeys packagegroup-core-ssh-openssh kbd"

cd /home/rescue/yocto/poky/BUILD
bitbake core-image-minimal


>>>> COMPILATION INCREMENTALE avec boost_1.86.0 (et dépendances) en complément


cd /home/rescue/yocto/poky/BUILD/tmp/deploy/images/qemux86-64
su
runqemu

=> Une console terminal s'ouvre, boot, login
login : root (pas de mot de passe)

cd /home/rescue/vsomeip/build
cmake ..

Pb : la librairie boost n'existe pas sur la VM Qemu









  make
  make install


### Cas de la compilation de vsomeip sur yocto (ou avec une version du compilateur gcc ultérieur à la version 11.4)

Sur la version actuelle du système Linux yocto, la version de gcc est 13.3. A partir de gcc 12, une erreur  `stringop-overflow` se produit à 54% de la compilation de la librairie vsomeip. Cette erreur peut être corrigée en la transformant en warning et en l'ignorant, sans impact sur le code généré.

Pour cela, modifier le fichier ~/vsomeip/CMakeLists.txt à la ligne 81. Ajouter ` -Wnoerror=stringop-overflow` à la fin de la ligne afin de convertir les erreurs `stringop-overflow` en warnings.  

Voici la ligne corrigée :
  set(OS_CXX_FLAGS "${OS_CXX_FLAGS} -D_GLIBCXX_USE_NANOSLEEP -pthread -O -Wall -Wextra -Wformat -Wformat-security -Wconversion -fexceptions -fstrict-aliasing -fstack-protector-strong -fasynchronous-unwind-tables -fno-omit-frame-pointer -D_FORTIFY_SOURCE=${_FORTIFY_SOURCE} -Wformat -Wformat-security -Wpedantic -Werror -fPIE -Wnoerror=stringop-overflow")

Ré-exécuter ensuite les commandes précédentes :

Executer :
  cmake ..
  make
  make install





