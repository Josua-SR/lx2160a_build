# SolidRun's LX2160A COM express type 7 build scripts

## Introduction
Main intention of this repository is to provide build scripts that are easy to handle than NXP's flexbuild build environment.

They are used in SolidRun to quickly build images for development where those images can be SD or SPI booted or network TFTP (kernel) or used for root NFS

The sources are pulled from NXP's codeaurora repository and patched after being clone using the patches in the patches/ directory

## Build with Docker
A docker image providing a consistent build environment can be used as below. Since some steps require mounting a loopback device, you need to grant permission for the container to do so when launching:

1. build container image (first time only)
   ```
   docker build -t lx2160a_build docker
   ```
2. invoke build script in working directory
   ```
   docker run --cap-add SYS_ADMIN --device /dev/loop0 --device /dev/loop-control -i -t -v "$PWD":/work lx2160a_build $(id -u) $(id -g)
   ```

## Build with host tools
Simply running ./runme.sh will check for required tools, clone and build images and place results in images/ directory.

We enhanced the NXP PBI scripting tools to accomodate auto detection of boot device, due to that the boot images are now unified for SD, eMMC and SPI.

## Customize
By default the script will create an image bootable from SD (ready to use .img file) with DDR4 SO-DIMMs 3200Mtps, SERDES configuration 8/5/2 (SERDES1 = 8, SERDES2 = 5, SERDES = 2) and u-boot as boot loader.

Following are the environment variables that can be modified -

Selecting DDR4 SO-DIMM speed - *DDR_SPEED=2400,2600,2900,3200*

Selecting SERDES configuration - *SERDES=SD1_SD2_SD3*

Selecting boot loader - *BOOT_LOADER=u-boot,uefi*


### Examples:
- `./runme.sh` **or**
- `docker run --cap-add SYS_ADMIN --device /dev/loop0 --device /dev/loop-control -i -t -v "$PWD":/work lx2160a_build $(id -u) $(id -g)`

generates *images/lx2160acex7_2000_700_3200_8_5_2.img* which is an image ready to be deployed on micro SD card and *images/lx2160acex7_xspi_2000_700_3200_8_5_2.img* which is an image ready to be deployed on the COM SPI flash.


## Deploying
For SD card bootable images, plug in a micro SD into your machine and run the following, where sdX is the location of the SD card got probed into your machine -

`sudo dd if=images/lx2160acex7_2000_700_3200_8_5_2_sd.img of=/dev/sdX`

For SPI boot, boot thru SD card and then load the _xspi_ images to system memory and flash it using the `sf probe` and `sf update` commands. An example below loads the image through TFTP prototocl, flashes and then verifies the image -

`sf probe; setenv ipaddr 192.168.15.223; setenv serverip 192.168.15.3; tftp 0xa0000000 lx2160acex7_xspi_2000_700_2600_8_5_2_xspi.img;sf update 0xa0000000 0 $filesize; sf read 0xa4000000 0 $filesize; cmp 0xa0000000 0xa4000000 $filesize`
