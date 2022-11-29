# MacOS set-up tutorial

## Install necessary packages:

```
pip install esptool
brew install x86_64-elf-gcc
brew install u-boot-tools
brew update && brew install bash
brew install picocom
```

## Clone repos:

```
git clone https://bitbucket.org/nuttx/tools.git
git clone https://github.com/apache/incubator-nuttx.git nuttx
git clone https://github.com/apache/incubator-nuttx-apps.git apps
```

## Install kconfig-frontends:

```
cd tools/kconfig-frontends
patch < ../kconfig-macos.diff -p 1
./configure --enable-mconf --disable-shared --enable-static --disable-gconf --disable-qconf --disable-nconf
make
make install
```

## Download crosscompiler:

Download [xtensa-esp32-elf](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/tools/idf-tools.html#xtensa-esp32-elf). Afterwards, please add the binaries to your PATH variable: `export PATH=$PATH:/Users/laura/Documents/NuttX/xtensa-esp32-elf/bin`

## Compile NuttX:

```
cd nuttx
./tools/configure.sh -m esp32-devkitc:nsh
make menuconfig
make
```

## Download bootloader and partition table:

Download the [bootloader](https://github.com/espressif/esp-nuttx-bootloader/releases/download/latest/bootloader-esp32.bin) and the [partition table](https://github.com/espressif/esp-nuttx-bootloader/releases/download/latest/partition-table-esp32.bin).

## Flash NuttX on ESP32

```
make flash ESPTOOL_PORT=/dev/cu.usbserial-0001  ESPTOOL_BINDIR=../bootloader/
picocom -b 115200 /dev/cu.usbserial-0001
```
