# MacOS set-up tutorial

In this tutorial you will learn how to build and upload the NuttX OS on your ESP32 Sparrow boards. The Sparrow board is custom-made, built on the ESP32 Wrover module by adding:

- LTR308 light sensor
- BME680 temperature, humidity and pressure sensor
- SSD1306 128x32 OLED display
- I2S microphone
- microSD card reader

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

Please not that if you are compiling for another target hardware, such as the popular WROOM module, you need to use `./tools/configure.sh esp32-devkitc:nsh`. Alternatively, since Sparrow boards are built upon the WROVER module, you can also use `./tools/configure.sh esp32-wrover-kit:nsh`. However, in this case you would have to manually enable the support for various peripherals using NuttX's `make menuconfig`.

## Download bootloader and partition table:

Download the [bootloader](https://github.com/espressif/esp-nuttx-bootloader/releases/download/latest/bootloader-esp32.bin) and the [partition table](https://github.com/espressif/esp-nuttx-bootloader/releases/download/latest/partition-table-esp32.bin).

## Flash NuttX on ESP32

```
make flash ESPTOOL_PORT=/dev/cu.usbserial-0001  ESPTOOL_BINDIR=../bootloader/
picocom -b 115200 /dev/cu.usbserial-0001
```
