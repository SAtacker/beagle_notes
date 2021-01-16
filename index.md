# Beagle_Notes

## Boot

Power On

1. (First Stage bootloader) TI Boot ROM performs minimal congfig, finds boot image and loads x-loader

   - This bootloader initializes a minimal amount of CPU and board hardware, then accesses the first partition of the SD card (which must be in FAT format), and loads a file called "MLO", and executes it.

2. (Second Stage bootloader) MLO "Mmc LOader" on the FAT partition is the second stage bootloader

   - Sets up pin muxing initializes clocks and memory and loads U boot

3. (Third Stage ) U-Boot is Das-Universal Boot , u-boot.img on FAT

   - Specifies root file system and uses uEnv.txt config , performs additional inits, loads Linux Kernel and passes control
   - The uenvcmd from uEnv.txt file is executed.
   - The file /dtbs/am335x‐bone*device_name*.dtb is read in. This file contains the BBB’s compiled device tree description, which is discussed shortly. After this description is read in, the flattened device tree blob is placed in memory at the address 0x815f0000

4. Linux Kernel (EXT4 partition on SD card)
   - Decompresses kernel into memory and sets up I2C , USB, etc. and mounts file system containing linux applications i.e mounts the root file system ( mmcroot and mmcrootfstype are defined in uEnv.txt )
   - Calls userspace process init

## Kernel Space and User Space

- Kernel space is the area of system memory where Linux kernel runs and is separated from user space to provide better security and helps to avoid crashing due to badly written user code.

```
    ---------------------------------------------
    |               User space                  |
    |  /sbin/init UserCode LinuxConsole         |
    |            GNU C lib (glib)               |
    ---------------------------------------------
                        ||
                        \/
                    Kernel space
                        ||
                        \/
                    Hardware & Devices
```

- A kernel module is an object file that contains code, which can be loaded and unloaded from the kernel on demand. In many cases the kernel can even load and unload modules while it is executing, without needing to reboot the BBB.
- Example : When we add wifi module it uses LKM (loadable kernel module)
- Further kernel services are made available through system calls
- System V init or systemd manages these systems and services, can be used to start and stop them

### Init process

- Begins by reading config from /etc/inittab which defines runlevel
- Runlevel defines the state of device and controls which process and services are started by init
- In debian there are several runlevels from 0-6 

```
satacker@ubuntu:~/Desktop/beagle_notes$ who -r
         run-level 5  2021-01-14 22:00
satacker@ubuntu:~/Desktop/beagle_notes$ runlevel
N 5
satacker@ubuntu:/etc$ ls -d rc*
rc0.d  rc1.d  rc2.d  rc3.d  rc4.d  rc5.d  rc6.d  rcS.d
```

