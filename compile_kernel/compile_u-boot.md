```
apt-get install build-essential git gcc-arm-linux-gnueabihf bison flex python-all-dev

mkdir -p $HOME/projects/mahalia
git clone git@github.com:mahalia-dependencies/mahalia-u-boot.git $HOME/projects/mahalia/mahalia-u-boot

cd $HOME/projects/mahalia/mahalia-u-boot
git checkout v2017.09

make am335x_boneblack_defconfig
ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make
```
