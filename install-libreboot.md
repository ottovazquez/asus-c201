# Install Asus/Libreboot UEFI

## Flash UEFI
All documentaiton from [Libreboot](https://libreboot.org/docs/install/c201.html)
It is a good idea to use `flashrom` bin from CrOS

```
# flash original Asus UEFI
flashrom -p host -w libreboot/roms/asus-c201.img.original

# flash Libreboot UEFI
flashrom -p host -w libreboot/roms/asus-c201.img.libre-r20160907
```


## Unbrick using RPI or BB

```
apt-get update
apt-get install -y 	\
  build-essential 	\
  pciutils 			\
  usbutils          \
  libpci-dev        \
  libusb-dev        \
  libftdi1          \
  libftdi-dev 		\
  zlib1g-dev        \
  subversion        \
  libusb-1.0-0-dev

# build flashrom
git clone https://github.com/flashrom/flashrom.git
cd flashrom
make

# check flashrom
./flashrom -VVV -p linux_spi:dev=/dev/spidev0.0

# connect to bios and check spidev
cc -o spidev_test spidev_test.c
./spidev_test -D /dev/spidev0.0 #DEADBEEFBAADFOOD 

```

## Resources
- [Libreboot RaspberryPi setup](https://libreboot.org/docs/install/rpi_setup.html)
- [Libreboot Asus C201](https://libreboot.org/docs/install/c201.html)
- [ Disabling SPI write protection, reflashing, and unbricking an Asus Chromebook C201
](https://gist.github.com/jcs/4bf59314d604538a5098)
- [Flashrom for RaspberryPi](https://www.flashrom.org/RaspberryPi)
- [Asus C201 take down](http://selinuxproject.org/%7Ejmorris/lss2013_slides/
safford_chromebook_takeown.pdf)
- [Unbricking the C720 Chromebook with the BeagleBone Black or Raspberry Pi](https://www.tnhh.net/posts/unbricking-chromebook-with-beaglebone.html)
