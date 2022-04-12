# Using ESP8266-RTOS-SDK on Debian

## Motivation

ESP8266-RTOS-SDK is not well supported on PlatformIO. <https://community.platformio.org/t/vs-code-esp8266-non-rtos-nonos-sdk-project-creation/27091> (PlatformIO provides a very low friction path tp programming a variety of microcontrollers.) Espressif themselves deprecate ESP8266-NONOS-SDK and recommend ESP8266-RTOS-SDK <https://github.com/espressif/ESP8266_NONOS_SDK#support-policy-for-esp8266-nonos> It seems if one wishes to use Espressif's SDK and tools directly, it should be via the instructions found at <https://github.com/espressif/ESP8266_RTOS_SDK#esp8266-rtos-software-development-kit>. I found these instructions a bit challenging so I thought it would be useful to work my way through them (blunder my way through) and document what I needed to do to get this working.

## Overview

1. Install required OS packages and some config.
1. Clone the Espressif Github repo.
1. Run the repo provided `install.sh` script to install the tools.
1. Build a trial project.

Note: The instructions on the web page list downloading build tools and then provide no hint as to where to install them. The `install.sh` script will do that so there is no need for the user to download build tools separately.

## Environment

This install was performed on a new install of Debian Bullseye, including the Gnome and KDE desktops as well as the SSH server using a non-free netinst image. Commands that require root are preceeded with `sudo` rather than using `#` or `$` to indicate whether they are user or root commands to make it poossible to copy the entire line and paste it into a terminal window.


The SDK requires the following environment variable to get off the ground.

```text
export IDF_PATH=~/esp/ESP8266_RTOS_SDK
```

The SDK requires that `python` run Python 3. On Debian Bullseye this can be accomplished using `update-alternatives`. (Debian Bullseye does not have an unversioned `python` program by default.)

```text
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.9 1
python --version
```

```test
sudo apt install wget python3-virtualenv python3-pip \
    libncurses-dev flex bison gperf minicom git
```

I configure `git` with the following commands. (Substitute your preferred characteristics.)

```text
git config --global user.name "<Yourt Name>"
git config --global user.email <your email>
git config --global pull.ff only 
```


You will need read/write access to the USB device. This can be done by adding the user to the `dialout` group.

```text
sudo usermod -a -G dialout $USER
su - $USER
```

(The second command adds the group to the users environment without having to logout.)


## Cloning ESP8266-RTOS-SDK

Espressif's instructions indicate to clone into `~/esp/`

```text
mkdir esp
cd esp
git clone https://github.com/espressif/ESP8266_RTOS_SDK.git
```

This will create `~/esp/ESP8266_RTOS_SDK/`.

## Install the tool chain

```text
export IDF_PATH=~/esp/ESP8266_RTOS_SDK
cd ~/esp/ESP8266_RTOS_SDK/
./install.sh
```

If all goes well, you will see

```text
.
.
.
All done! You can now run:

  . ./export.sh

```

This will have installed the tool chain in `~/.espressif`. Sourcing `export.sh` will report

```text
hbarta@yggdrasil:~/esp/ESP8266_RTOS_SDK$ . ./export.sh
Adding ESP-IDF tools to PATH...
Checking if Python packages are up to date...
Python requirements from /home/hbarta/esp/ESP8266_RTOS_SDK/requirements.txt are satisfied.
Added the following directories to PATH:
  /home/hbarta/esp/ESP8266_RTOS_SDK/components/esptool_py/esptool
  /home/hbarta/esp/ESP8266_RTOS_SDK/components/partition_table/
  /home/hbarta/.espressif/tools/xtensa-lx106-elf/esp-2020r3-49-gd5524c1-8.4.0/xtensa-lx106-elf/bin
  /home/hbarta/.espressif/python_env/rtos3.4_py3.9_env/bin
  /home/hbarta/esp/ESP8266_RTOS_SDK/tools
Done! You can now compile ESP8266-RTOS-SDK projects.
Go to the project directory and run:

  make

hbarta@yggdrasil:~/esp/ESP8266_RTOS_SDK$
```

The hello_world project can then be built

```text
cd ./examples/get-started/hello_world
make
make flash
```

I was unable to get `minicom` to read the output from `/dev/ttyUSB0` at the default baud rate of 74880 and also tried 9600, 19200 and 115200.

## Contributing

Contributions welcome but not limited to

* Wrong or confusing wording.
* Additional Linux distros and/or operating systems.
* Instructions for integrating this with VS Code or other popular IDEs but without PlatformIO. (Or with PlatformIO if that can be managed.)

The repo for this page is at <https://github.com/HankB/MkDocs-blog/blob/main/tech/ESP8266-RTOS-SDK_on_Debian.md>

