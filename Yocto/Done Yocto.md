NB : on ne peut pas renommer le rep BUILD en build-qemuarm64 : sinon il faut regénérer l'image yocto from scratch

su
cd /home/rescue/yocto/poky/BUILD
mv downloads ../..
mv sstate-cache ../..

vi conf/local.conf
  Saisir :
    DL_DIRECTORY ?= "$(TOPDIR)/../../downloads
    SSTATE_DIR ?= "$(TOPDIR)/../../sstate-cache"

mkdir -p /home/rescue/yocto/poky/build-qemuarm64
chown -R rescue /home/rescue/yocto/poky/build-qemuarm64

cd /home/rescue/yocto/poky
./oe-init-build-env build-qemuarm64
cd build-qemuarm64

vi conf/local.conf
  Saisir :
    MACHINE ?= "qemuarm64"                               //décommenter cette ligne
    DL_DIRECTORY ?= "$(TOPDIR)/../../downloads
    SSTATE_DIR ?= "$(TOPDIR)/../../sstate-cache"

su -l rescue
bitbake core-image-minimal

