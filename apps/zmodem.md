# File transfer through Zmodem

## Prerequisites

### Minicom

```
sudo apt-get update
sudo apt-get install minicom
```

Use ```minicom -s``` to configure minicom parameters. Select:

- Serial port setup: change the serial device from ```/dev/modem``` to ```/dev/ttyUSB0```.
- Disable hardware flow control.
- Save using ```Save setup as dfl```, so that we overwrite the default configuration file.

### Zmodem

Use the following configs:
- CONFIG_SYSTEM_ZMODEM=y
- CONFIG_SYSTEM_ZMODEM_RCVBUFSIZE=1024
- CONFIG_SYSTEM_ZMODEM_PKTBUFSIZE=1024
- CONFIG_SYSTEM_ZMODEM_SNDBUFSIZE=1024
- CONFIG_FS_TMPFS=y

We need to increase the buffer size from 512 (default) to 1024 bytes because the Linux implementation of zmodem uses buffers of 1024 bytes.

By default, zmodem uses ```/tmp``` to store the received files so we need to enable support for that as well.

## ESP32 -> Linux transfer

Please use ```sz <path>``` when trying to transfer data from the ESP32 to the Linux machine. Once an ``sz`` command has been detected, Minicom will open an ```rz``` instance and the transfer will begin automatically.

Once the transfer is complete, you can find your file in the current directory of the minicom process.

```Note:``` it seems that the ```sz``` binary needs the full path of the file you want to send.

## ESP32 <- Linux transfer

Minicom has the special key ```CTRL+A S``` which triggers Linux automatically to send a file to the ESP32. In the background, minicom will open an ```sz``` and ```rz``` process on Linux and ESP32, respectively.

```Note:``` there is also ```CTRL+A R``` which technically should allow ESP32 to send files to the Linux machine, but I never managed to get it to work.
