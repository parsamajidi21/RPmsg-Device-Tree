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

#### step 2:
All the target machine should be specified and as the image is being built for the Verdin i.MX8 Mini, some changes are needed. The following line in local.conf should be uncommented:
```bash
MACHINE ?= "verdin-imx8mm"
```
#### step 3:
Since the image is being constructed for a device utilizing an NXP-based System on Module (SoM), certain downloads necessitate your acknowledgment and agreement with the NXP®/Freescale End User License Agreement (EULA), which can be found in the layers/meta-freescale/EULA directory. To indicate your acceptance, you must include the provided line in your local.conf file.
```bash
ACCEPT_FSL_EULA = "1"
```
#### step 4:
The target reference image in this case is [**tdx-reference-minimal-image**](https://developer.toradex.com/linux-bsp/os-development/build-yocto/build-a-reference-image-with-yocto-projectopenembedded/?gclid=CjwKCAjwsvujBhAXEiwA_UXnAEbxzwboBRawAccQ5irI6bVBE26zMIeLDcsvv2Nfc891noOnpLSevhoCEHMQAvD_BwE#build-an-image) which is a non-graphical-interface image, and it includes the base command-line package ‘**packagegroup-base-tdx-cli**’ included in '**[packagegroup-tdx-cli.bb](https://git.toradex.com/cgit/meta-toradex-demos.git/tree/recipes-images/images/packagegroup-tdx-cli.bb?h=dunfell-5.x.y)**'.
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
  
![Image](https://github.com/parsamajidi21/RPmsg-Device-Tree/blob/main/images/builtYocto.png)

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

![dd](https://github.com/parsamajidi21/RPmsg-Device-Tree/blob/main/images/photo1.png)

In the image, the above terminal is for communicating with Linux/U-Boot and the below terminal is for communicating with the cortex-M4. When we run the above command, it starts communicating with Linux through the UART and by powering up the board, Linux boots up. Meanwhile booting up the OS by pressing any key in the keyboard and interrupting the boot process we will have a U-Boot shell, as follows:

![dd](https://github.com/parsamajidi21/RPmsg-Device-Tree/blob/main/images/photo2.png)

following commands load the executbale file in memory accessing by M4 processor:
```bash
Verdin iMX8MM # setenv m4addr 0x7e0000
Verdin iMX8MM # saveenv
Verdin iMX8MM # fatload mmc 1 0x48000000 hello_world.bin && dcache flush && cp.b 0x48000000 ${m4addr} 0x20000
Verdin iMX8MM # bootaux ${m4addr}
```
![dd](https://github.com/parsamajidi21/RPmsg-Device-Tree/blob/main/images/photo4.png)

The file "hello_world.bin" is 6268 bytes in size. After configuring an environment variable using the 'setenv' command, the address 0x007e0000 is designated as 'm4addr'. This address corresponds to the Cortex-M4's Tightly Coupled Memory (TCM) accessible by Cortex-A53.

The `fatload mmc 1 0x48000000 hello_world.bin` command fetches the file from the second SD card partition (mmc 1) and stores it at memory address 0x48000000, which the M4 can access.

The `dcache flush` command ensures data integrity, synchronizing memory modifications with the shared main memory of Cortex A53 and Cortex M4.

Copying a data block from 0x48000000 to 0x007e0000, the block's size is specified as 0x20000.

Executing `bootaux ${m4addr}` activates the Cortex-M4 core, displaying `hello world` on the M4 UART interface. This establishes communication between both cores.

### Interprocess Communication (IPC) - RPmsg on Verdin i. MX8M Mini:

Modern System-on-Chips (SoCs) often employ heterogeneous remote processor devices in asymmetric multiprocessing (AMP) configurations, which utilize different instances of operating systems such as Linux and real-time OS. RPmsg serves as a communication bridge between kernel drivers on different processors, allowing the exchange of messages and data. This note describes the process of configuring and utilizing RPmsg for communication between a Cortex A53 processor (Linux) and a Cortex M4 processor (RTOS). You can find more information about RPMsg in AMP configurations [here](https://technotes.kynetics.com/2018/Linux_rpmsg_char_driver/). 

#### Procedure

1. Building RPmsg Executables:
    * Download the FreeRTOS source files, [here](https://developer.toradex.com/software/real-time/freertos/freertos-on-the-cortex-m4-of-a-verdin-imx8m-mini/#get-the-freertos-source-code)
    * Navigate to `/boards/evkmimx8mm/multicore_examples`
    * Two folders exist in this directory: `rpmsg_lite_pingpong_rtos` and `rpmsg_lite_str_echo_rtos`.
    * For example, navigate to `/boards/evkmimx8mm/multicore_examples/rpmsg_lite_str_echo_rtos/armgcc`.
    * Run the `build_all.sh` shell script to generate executables.
    * This results in the creation of two files: `rpmsg_lite_str_echo_rtos.bin` and `rpmsg_lite_str_echo_rtos_imxcm4.elf`.
    * Copy the `rpmsg_lite_str_echo_rtos.bin` on a SD card.
2. Device Tree Modifications:
    * Default device tree may lack the RPmsg feature definition.
    * Combine existing device tree files like `imx8mm-evk-rpmsg.dts`, `imx8mm-verdin.dtsi`, `imx8mm-verdin-wifi.dtsi`, and `imx8mm-verdin-dev.dtsi` to create **`imx8mm-verdin-wifi-dev-rpmsg.dts`** in the `{HOME}/path/oe-core/build/tmp/work-shared/verdin-imx8mm/kernel-source/arch/arm64/boot/dts/freescale `
    * Include the necessary .dtsi files within `imx8mm-evk-rpmsg.dts`, then rename it to `imx8mm-verdin-wifi-dev-rpmsg.dts`.    
       **Find the custom device tree [here](https://github.com/parsamajidi21/RPmsg-Device-Tree/blob/main/imx8mm-verdin-wifi-dev-rpmsg.dts)** 
    * Add the following command in the corresponding Makefile:
      ```bash
      dtb-$(CONFIG_ARCH_MXC) += imx8mm-verdin-wifi-dev-rpmsg.dtb
      ```
    * Modify the `KERNEL_DEVICETREE` and `TORADEX_PRODUCT_IDS` in the  machine configuration `verdin-imx8mm.conf` in the `/oe-core/layers/meta-toradex-nxp/conf/machine` directory.
      ```bash
      KERNEL_DEVICETREE = " \
      freescale/imx8mm-verdin-nonwifi-dahlia.dtb \
      freescale/imx8mm-verdin-nonwifi-dev.dtb \
      freescale/imx8mm-verdin-nonwifi-yavia.dtb \
      freescale/imx8mm-verdin-wifi-dahlia.dtb \
      freescale/imx8mm-verdin-wifi-dev.dtb \
      freescale/imx8mm-verdin-wifi-yavia.dtb \
      freescale/imx8mm-verdin-wifi-dev-rpmsg.dtb \
      "
      ```  
      ```bash
      TORADEX_PRODUCT_IDS[0055] = "imx8mm-verdin-wifi-v1.1-dev-rpmsg.dtb"
      ```
      **Note**: For TORADEX_PRODUCT_IDS[0055], first check the Product id of your system, in this case it is **0055**
3. Building and Installing the Image:
    * Use bitbake to build the new image incorporating the custom device tree.
    * Install the new image on the board.
4. U-Boot Configuration:
    * Within U-Boot, execute the following commands to load the RPmsg file into memory and boot the M4 core:
      ```bash
      Verdin iMX8MM # setenv m4addr 0x7e0000
      Verdin iMX8MM # saveenv
      Verdin iMX8MM # fatload mmc 1 0x48000000 rpmsg_lite_str_echo_rtos.bin && dcache flush && cp.b 0x48000000 ${m4addr} 0x20000
      Verdin iMX8MM # bootaux ${m4addr}
      ```
Upon executing the provided U-Boot commands, the Cortex M4 processor successfully loads the RPmsg executable from memory. Subsequently, booting the system in the M4 console confirms the readiness of the M4 processor to receive messages from the Linux side.
![](https://github.com/parsamajidi21/RPmsg-Device-Tree/blob/main/images/photo5.png)

#### Loading RPmsg Module and Device Creation:
After following the previously mentioned steps and booting the system in the Linux console, the next phase involves loading the RPmsg module and creating the necessary device for communication.

In the Linux console, execute the command to load the RPmsg module:
```bash
modprobe imx_rpmsg_tty
```
This command initializes the RPmsg module, enabling communication between the Linux and RTOS processors.
After device creation, the RPmsg device is accessible as /dev/ttyRPMSG30.
Messages sent from the Linux side to /dev/ttyRPMSG30 will be received and displayed in the M4 console, demonstrating the successful data exchange through RPmsg.

![](https://github.com/parsamajidi21/RPmsg-Device-Tree/blob/main/images/photo6.png)

#### summary
The completion of these steps solidifies the RPmsg communication framework's functionality. By loading the RPmsg module, creating the RPmsg device, and engaging in bidirectional communication, the system successfully demonstrates interprocessor communication between the Cortex A53 (Linux) and Cortex M4 (RTOS) processors. This comprehensive implementation underscores the efficiency and versatility of RPmsg in enabling seamless communication in heterogeneous systems.

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



