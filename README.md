# **Yocto, RTOS, and RPMSG on Toradex IMX8MMini**

## Introduction
In the ever-evolving landscape of embedded systems development, creating customized Linux-based systems to suit specific hardware architectures has become a crucial task. The Yocto Project, an open-source collaboration initiative, empowers developers to craft tailor-made Linux systems for diverse applications. This article delves into the intricate journey of building a Yocto Linux distribution for the [Verdin i.MX8M Mini](https://www.toradex.com/computer-on-modules/verdin-arm-family/nxp-imx-8m-mini-nano), a versatile edge processing board and [Dahlia Carrier board](https://www.toradex.com/products/carrier-board/dahlia-carrier-board-kit). The process spans from workspace setup to interprocess communication, encompassing key stages such as Yocto configuration, RTOS installation, and the utilization of RPMSG.
## Realese Notes
Here are some important notes about our build:

* [Toradex BSP Layers and Refrence Images for Yocto Project](https://developer.toradex.com/software/toradex-embedded-software/embedded-linux-release-matrix/#toradex-bsp-layers-and-reference-images-for-yocto-project)
* **BSP**: 6.3.0
* **Kernel, U-Boot**: downstream Toradex 5.15.77 based on NXP BSP L5.15_2.1.0
* **Yocto**: 4.0 (Kirkstone)
* Compatible with
  * SOM
    * Toradex Verdin i.MX8M Mini V1.1A
  * Carrier Board
    * Dahlia Carrier board V1.1A    
* Get the FreeRTOS Source Code, guide: [here](https://developer.toradex.com/software/real-time/freertos/freertos-on-the-cortex-m4-of-a-verdin-imx8m-mini/#get-the-freertos-source-code) 

## Flash Instructions
The following steps will guide you through the installation of the built Yocto Linux on the [Verdin i.MX8M Mini](https://www.toradex.com/computer-on-modules/verdin-arm-family/nxp-imx-8m-mini-nano)
### **Hardware Requirements**
* Toradex Verdin i.MX8MM V1.1A with the Dahlia carrier board 1.0C, 1.1A or 1.1C
* A USB drive formatted in FAT32
* A host computer running GNU/Linux
* A USB-A to USB-C cable - to load the Toradex Easy Installer
* A SD Card 
### **Software Requirements**
* The Verdin board running the [Toradex Easy Installer](https://developer.toradex.com/easy-installer/toradex-easy-installer/)
  

### Installation of Yocto Linux
As an official definition of the Yocto Project, the Yocto Project is an open-source collaboration project that helps developers create custom Linux-based systems regardless of the hardware architecture.
#### step 1:  
The first step is to prepare our workspace, the ‘repo’ utility is used for this purpose. The repo utility is based on Google’s Git-repo tool and provides a way to manage a set of repositories as a single super project. It allows to specify a manifest file that defines the repositories, their locations, and the branches to be used. The manifest file is an XML file that describes the project’s structure.
##### Install the repo bootstrap binary:
```bash
$ mkdir ~/bin
$ export PATH=~/bin:$PATH
$ curl https://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
Create a directory for your OE-Core setup to live in and clone the meta-information:
```bash
$ mkdir ${HOME}/path/oe-core
$ cd ${HOME}/path/oe-core
$ repo init -u git://git.toradex.com/toradex-manifest.git -b kirkstone-6.x.y -m tdxref/default.xml
$ repo sync
```
Source the file `export` to setup the environment. On the first invocation, this also copies a sample configuration to build/conf/*.conf.
```bash
$ . export
```
**Note**: Sourcing export configures the shell environment for the current shell session. You must enter this command whenever you open a new shell session for use with OpenEmbedded.

#### step 2: Adjust the settings in the local.conf file
First of all the target machine should be specified and as the image is being built for the Verdin i.MX8 Mini, some changes are needed. The following line in local.conf should be uncommented:
```bash
MACHINE ?= "verdin-imx8mm"
```
#### step 3:
Since the image is being constructed for a device utilizing an NXP-based System on Module (SoM), certain downloads necessitate your acknowledgment and agreement with the NXP®/Freescale End User License Agreement (EULA), which can be found in the layers/meta-freescale/EULA directory. To indicate your acceptance, you must include the provided line in your local.conf file.
```bash
ACCEPT_FSL_EULA = "1"
```
#### step 4:
The target reference image in this case is [**tdx-reference-minimal-image**](https://developer.toradex.com/linux-bsp/os-development/build-yocto/build-a-reference-image-with-yocto-projectopenembedded/?gclid=CjwKCAjwsvujBhAXEiwA_UXnAEbxzwboBRawAccQ5irI6bVBE26zMIeLDcsvv2Nfc891noOnpLSevhoCEHMQAvD_BwE#build-an-image) which is a non-graphical-interface image, and it includes the base command-line package ‘**packagegroup-base-tdx-cli**’ included in '**packagegroup-tdx-cli.bb**'.
Finally the image is built using the following command:
```bash
$ bitbake tdx-refrence-minimal-image
```
The image will take some hour(s) to be built. After the image is built, the output artifacts such as images, u-boot, rootfs, deployable tarball can be found in the following directory:
```bash
$ {HOME}/path/oe-core/build/deploy/images/verdin-imx8mm
```
#### step 5: Install the built image on the target board
* Download the Toradex Easy Installer archive for the Verdin iMX8MM [here](https://developer.toradex.com/software/toradex-easy-installer#load-toradex-easy-installer)
* Unpack the Toradex Easy Installer archive in a directory on the host PC
* Insert the USB drive in one of the USB port on the Dahlia carrier board

**Note**: Skip the above part if the board is already running the Toradex Easy Installer

##### LOAD THE TORADEX EASY INSTALLER
* Set the board on the recovery mode, to do that the JP3 jumper must be opened, and then press and hold the recovery mode button (SW5)
* Power on the board by pressing the ON/OFF Button (SW3) while keeping the Recovery Mode button for an additional 10 seconds. Only after that, release the Recovery Mode Button (SW5)
* From the host Linux pc, enter in the directory with the unpacked Toradex Easy Installer archive and:
    ```bash
    $ sudo ./recovery-linux.sh
    ```
* Wait for the *Successfully* downloaded Toradex Easy Installer message, that will indicate that the board is now booting the Toradex Easy Installer

##### FLASH THE IMAGE  

* Untar the tarball image file and then copy it into a flash disk drive 
* Plug it into the USB port of the dahlia carrier board
* Connect to the board remotely via a VNC tool 
* Install the image on the board
  
![Image](/home/parsa/custom-dt/builtYocto.png)

### Installing RTOS to be used by the Cortex-M4 core
This stage aims to reload code on Cortex-M4 from the U-Boot. 
The first step is to build and prepare FreeRTOS sources to have an intended executable for running on the target. For this purpose, we use MCUXpresso SDK Builder. Find the complete guide [here](https://developer.toradex.com/software/real-time/freertos/freertos-on-the-cortex-m4-of-a-verdin-imx8m-mini/#get-the-freertos-source-code).

After downloading and building the FreeRTOS sources, download the following arm cross-compilation toolchain [from the official website](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads):
```bash
$ tar xjf ~/Downloads/gcc-arm-none-eabi-9-2020-q2-update-x86_64-linux.tar.bz2
```
It can be downloaded and installed using package manager in some linux distributions like ubuntu as follow:
```bash
$ sudo apt-get install gcc-arm-none-eabi binutils-arm-none-eabi
```
After extracting the downloaded SDK, go to `<sdk-directory>/boards/evkmimx8mm/demo_apps/hello_world/armgcc`, then follow the below steps to set some environment variables:
```bash
$ export ARMGCC_DIR=~/gcc-arm-none-eabi-9-2020-q2-update
```
If you've installed the toolchain using your package manager, then run:
```bash
$ export ARMGCC_DIR=/usr
```
At the end run the below shell script and it will build both the debug and release versions of the application for all possible targets (DDR, FLASH and RAM)
```bash
$ ./build_all.sh
```
The next step is to insert an SD card in the host and format it in FAT32, then copy the `hello_world.bin` into the SD card.
the `hello_world.bin` file can be found in `<sdk-directory>/boards/evkmimx8mm/demo_apps/hello_world/armgcc/release`

#### Hardware setup

For hardware setup, plug in the SD card into the USB port of the carrier board, the Dahlia Carrier board features a built-in USB to serial converter which can be used to access both the main OS debug UART- in this case UART3, as well as the default M4 debug UART (UART4) via a single USB C connector (X18)
After connecting the host pc to debug port of your board, four USB serial interface can be seen as follows:

```bash
$ ls /dev/ttyUSB*
  /dev/ttyUSB0 /dev/ttyUSB1 /dev/ttyUSB2 /dev/ttyUSB3  
```
The highest USB index `/dev/ttyUSB3` is the Linux console and `/dev/ttyUSB2` is for M4 processor

#### Environment setup
For preparing the environment and having two terminals, one for Linux/U-Boot and one for communication with the M4, the ‘tmux’ is used, which is a terminal multiplexer. And for communicating with UART through the console a serial terminal tmulator, `picocom`, is used. 


### Interprocess Communication (IPC) - RPmsg on Verdin i. MX8M Mini:



### Refrences:
* [Build a Reference Image with Yocto Project/OpenEmbedded](https://developer.toradex.com/linux-bsp/os-development/build-yocto/build-a-reference-image-with-yocto-projectopenembedded/?gclid=CjwKCAjwsvujBhAXEiwA_UXnAEbxzwboBRawAccQ5irI6bVBE26zMIeLDcsvv2Nfc891noOnpLSevhoCEHMQAvD_BwE)
* [Android 11 for Toradex Verdin i.MX8MM](https://technotes.kynetics.com/2021/Android_11_1.0.0_Toradex_Verdin_iMX8MM_V1.1A/)
* [FreeRTOS on the Cortex-M4 of a Verdin iMX8M Mini
Introduction
](https://developer.toradex.com/software/real-time/freertos/freertos-on-the-cortex-m4-of-a-verdin-imx8m-mini/)
* [Toradex Device Tree Documentation Overview](https://developer.toradex.com/software/linux-resources/device-tree/)
* [Asymmetric Multiprocessing and Embedded Linux](https://elinux.org/images/3/3b/NOVAK_CERVENKA.pdf)
* [An Introduction to Asymmetric Multiprocessing: When this Architecture can be a Game Changer](https://www.slideshare.net/kynetics/amp-kynetics-elc-2018-portland)
* [FreeRTOS on the Cortex-M4 of a Colibri iMX7, RPMSG TTY Example](https://developer.toradex.com/software/real-time/freertos/freertos-on-the-cortex-m4-of-a-colibri-imx7#rpmsg-tty-example)



