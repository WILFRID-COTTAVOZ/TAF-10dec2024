créer un workspace 
dans ce workspace créer rep layers et builds au même niveau que downloads et sstate-cache
dans rep layers télécharger yocto par git
  git clone git://git.yoctoproject.org/poky -b scarthgap
cd poky
git tag
git checkout <poky_derniere_version>
cd ../..     //on remonte au niveau workspace

source layers/poky/oe-init-build-env builds/build-qemu
runqemu
login : root (pas de mdp)

NB : Alt-Ctrl-G pour sortir de la console Qemu

NB : runqemu nographic => console texte + même config clavier (??)
=> Ctrl-a pour arrêter Qemu

PC recommandé : 12 à 16 cores, 32 Go RAM, 100 Go HDD
yocto sur VM : temps de compil très longs !

MACHINE=qemuarm bitbake core-image-minimal

runqemu qemuarm

# générer une image pour qemuarm64

créer rep build-qemuarm64 dans builds au même niveau que build-qemu

cd build-qemu
mv downloads ../../
mv sstate-cache ../../
cd ../../
source layers/poky/oe-init-build-env build/build-qemuarm64

vi builds/build-qemuarm64/conf/local.conf
  Saisir :
    MACHINE ?= "qemuarm64"                     // vérifier existence du fichier layers/poky/meta/conf/machine/qemuarm64.conf
    DL_DIR ?= "$(TOPDIR)/../../downloads"
    SSTATE_DIR ?= "$(TOPDIR)/../../sstate-cache"

cd builds/build-qemuarm64

bitbake core-image-minimal

runqemu


# Configurer les users et hostname sur l'image 

openssl passwd -6 yocto     // hash du mdp pour root
=> mdp hashé en SHA512

openssl passwd -6 yocto     // hash du mdp pour rescue
=> mdp hashé en SHA512

NB : préfixer les 3 $ de chaque mdp hashé par \


vi builds/build-qemuarm64/conf/local.conf
  Saisir :
    INHERIT += "extrausers"
    EXTRA_USERS_PARAMS += "useradd -p '<mdp en SHA512>' rescue;"
    EXTRA_USERS_PARAMS += "usermod -p '<mdp en SHA512>' root;"

    hostname:pn-base-files = "SDVPi5-1"


# ajouter vsomeip dans l'image

recherche d'une recette vsomeip dans le site https://layers.openembedded.org/
=> vsomeip v3.4.10 "The implementation of SOME/IP" => layer meta-networking

mkdir -p /home/rescue/yocto/poky/other-layers
cd /home/rescue/yocto/poky/other-layers
git clone git://git.openembedded.org/meta-openembedded 

cd meta-openembedded
find meta-networking -name '*vsomeip*.bb'
  => meta-openembedded/meta-networking/recipes-protocols/vsomeip/vsomeip_3.4.10.bb

cd /home/rescue/yocto/poky/build-qemuarm64
bitbake-layers add-layer ../other-layers/meta-openembedded/meta-oe                    //pour ajouter le layer meta-oe à la config de la VM qemuarm64 (pérequis de 
                                                                                        meta-python) 
bitbake-layers add-layer ../other-layers/meta-openembedded/meta-python                //pour ajouter le layer meta-python à la config de la VM qemuarm64 (prérequis de  
                                                                                        meta-networking) 
bitbake-layers add-layer ../other-layers/meta-openembedded/meta-networking            //pour ajouter le layer meta-networking à la config de la VM qemuarm64 


bitbake-layers show-layers                                                            //pour vérifier l'ajout des 3 layers à la config de la VM qemuarm64 

NB : La priorité des layers ajouté est de 5, comme les layers natifs


Executer :
  vi ~/yocto/poky/build-qemuarm64/conf/local.conf
  Saisir (en fin de fichier) :
    IMAGE_INSTALL:append = " vsomeip openssl libssh2 openssh ssh-pregen-hostkeys kbd"

NB : packagegroup-core-ssh-openssh non supporté pour VM qemuarm64     ??                (supporté pour VM Qemu x86 64 bits)


Executer :
  cd /home/rescue/yocto/poky/build-qemuarm64
  bitbake core-image-minimal 


NB : vsomeip est installé 
       libs sous /usr/lib : libvsomeip*.so.*
       fichiers de config json sous /etc/vsomeip : *.json
       Pas d'appli fournie










# générer une image pour la carte Raspberry Pi 4

cd ~/layers
git clone git://git.yoctoproject.org/meta-raspberrypi -b scarthgap
cd ..
source layers/poky/oe-init-build-env builds/build-rpi

cd build-rpi
bitbake-layers show-layers
bitbake-layers add-layer ../../layers/meta-raspberrypi/
bitbake-layers show-layers
vi build-rpi/conf/bblayers.conf    // Pour voir le layer ajouté à la variable BBLAYERS

ls ../../layers/meta-raspberrypi/conf/machine   // Pour voir les cartes Raspi supportées

vi conf/local.conf
  Saisir :
    MACHINE = "raspberrypi4"
    ENABLE_UART = "1"     // Pour activer une console sur le port série de la Raspi
    DL_DIR = "$(TOPDIR)/../../downloads"
    SSTATE_DIR = "$(TOPDIR)/../../sstate-cache"
    LICENSE_FLAGS_ACCEPTED += 'synaptics-killswitch'   // Pour le Bluetooth

bitbake core-image-base

NB : pour le lab R&D SDV générer plutôt pour Raspi 5 !!


# Installer image sur carte Raspi 4

Cette commande produit 2 fichiers .wic.bz2 et .wic.bmap dans tmp/deploy/images/raspberrypi4

Insérer une carte micro-SD dans un adaptateur USB
lsblk 
  => sdd avec 2 partitions

umount /dev/sdd?
sudo bmaptool copy tmp/deploy/images/raspberrypi4/core-image-base-raspberrypi4.rootfs.wic.bz2 /dev/sdd

Insérer la carte micro-SD dans la carte Raspi 4
Booter la carte Raspi 4
login : root   (no mdp)
uname -a
cat /proc/cpuinfo
free
cat /sys/firmware/devicetree/base/model

NB : Avec minicom se connecter au port série de la Raspi


**** Tuto II.1 *****

