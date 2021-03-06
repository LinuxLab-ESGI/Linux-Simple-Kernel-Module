# Linux-Simple-Kernel-Module

Short repo about Linux kernel modules.
We are going to see the main commands to manage kernel modules and develop a simple module for our system.

- [Linux-Simple-Kernel-Module](#linux-simple-kernel-module)
  - [Kernel modules](#kernel-modules)
  - [Main commands](#main-commands)
    - [Getting information](#getting-information)
    - [Manually load a module](#manually-load-a-module)
    - [Manually unload a module](#manually-unload-a-module)
    - [Automatically load a module](#automatically-load-a-module)
    - [Blacklist a module](#blacklist-a-module)
  - [Our first Hello-Bye World module](#our-first-hello-bye-world-module)
    - [Explanation](#explanation)
  - [Sources](#sources)
## Kernel modules

Kernel modules are separated compiled codes that can be integrated (loaded or unloaded) into the Linux kernel on the demand without the need to reboot the system. The main advantage is the capacity to add features to the Kernel without recompiling all the Kernel.
It is usually called LKM (**L**inux **K**ernel **M**odule)

## Main commands

### Getting information

| Command                               | Description                            |
| ------------------------------------- | -------------------------------------- |
| `lsmod`                               | Show all loaded kernel modules         |
| `modinfo module_name`                 | Show information about a kernel module |
| `modprobe --show-depends module_name` | Show dependencies of a module          |

### Manually load a module

To load a module located in `/usr/lib/modules/$(uname -r)/` :  
`sudo modprobe module_name`

If the module is not located in the specific folder, we can use this base command but it won't find automatically dependencies :  
`sudo insmod /path/module_name`

### Manually unload a module

`sudo modprobe -rf module_name`  
*-r remove*  
*-f force*

Or :  
`sudo rmmod module_name`

### Automatically load a module

If we want to add or blacklist an module to the Kernel, we can specify a list of kernel modules to load in a conf file. Each module name is separated by a newline.  
The conf file has to be in `/etc/module-load.d/name_of_file.conf`.
Example :  

``` Bash
cat /etc/modules-load.d/modules.conf 

# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
module_name1
module_name1
```

### Blacklist a module

If we want to prevent a module to be loaded for any reasons, we can add it in a conf file with the option `blacklist` in `/etc/modprobe.d/`.  
Example :  

``` Bash
cat /etc/modules-load.d/blacklist.conf 

# Not loaded modules
blacklist module_name1
blacklist module_name1
```

## Our first Hello-Bye World module

> :warning: 
> It's recommended to develop kernel modules in a virtual machine.

First we need some packages :

- gcc
- make
- your kernel headers
- build-essential

We can install it :  

``` Bash
sudo apt install gcc build-essential make linux-headers-`uname -r`
```

The main C file named `lkm_hello.c` :  

```C
/*
 * File : lkm_hello.c 
 * Simple Hello-World LKM 
 * Linux-Lab ESGI 2021
 */

#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Linux-Lab");
MODULE_DESCRIPTION("Simple LKM");
MODULE_VERSION("0.1");

static int __init lkm_hello_init(void)
{   
    char string[] = "World";
    printk(KERN_INFO "Hello, %s !\n", string);
    return 0;
}
static void __exit lkm_hello_exit(void)
{
    printk(KERN_NOTICE "Bye, World !\n");
}
module_init(lkm_hello_init);
module_exit(lkm_hello_exit);
```

The Makefile named `Makefile` next to the C file :  

```Makefile
BUILD=/lib/modules/$(shell uname -r)/build

obj-m += lkm_hello.o

all:
    make -C $(BUILD) M=$(PWD) modules
    echo "Compilation done"
clean:
    make -C $(BUILD) M=$(PWD) clean
    echo "Cleaning done"
test:
    sudo dmesg -C
    sudo insmod lkm_hello.ko
    sudo rmmod lkm_hello.ko
    dmesg
```

To compile our module :

`make all`  
or `make`

To test :  
`sudo make test`

To clean :  
`make clean`

### Explanation

**lkm_hello.c**
Includes for all the kernel and modules functions

```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
```

Module license and descriptions (for `modinfo`)

```C
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Linux-Lab");
MODULE_DESCRIPTION("Simple LKM");
MODULE_VERSION("0.1");
```

Two functions to initialize (during the load) and exit (during th unload) of the module.  
`static int __init functionName(void)` return an int  
`static void __exit lkm_hello_exit(void)` return a void

`__init` and `__exit` are macros to help the detection of the init and exit functions by the Kernel.

`module_init()` and `module_exit()` are need along the macros.

```C
static int __init lkm_hello_init(void)
{...}
static void __exit lkm_hello_exit(void)
{...}
module_init(lkm_hello_init);
module_exit(lkm_hello_exit);   

```

In kernel development we don't have standards C library.
For example instaed of `printf()` we have `prink()` function that takes a FLAG_INFO, a string and optionally arguments. It prints the message in the ring buffer, a sort of log for the kernel.
> We can also see the log of the kernel in /var/log/syslog

FLAG KERNEL_INFO indicates the priority of the message (error, warning, alert, info,...)

```C
printk(FLAG KERNEL_INFO "Hello, %s !\n", arg);
```

**Makefile**
Compiling with make is necessary.
`obj-m:...` specifies that the result of the compilation of our file is for building a module.

By default make will execute the `all` section.
`-C` change the directory.
This line is to compile our module :  

``` Makefile
# Compile our module
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
```

To clean all the temp files created during the compilation :

```Makefile
# Clean all the temp files
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

To test our module we can run these commands :

```Bash
# Clean the ring buffer
sudo dmesg -C
# Load the module
sudo insmod lkm_hello.ko
# Unload the module
sudo rmmod lkm_hello.ko
# Print the ring buffer
dmesg
```

## Sources

[Kernel Module - Archlinux](https://wiki.archlinux.org/index.php/Kernel_module)

[Les modules linux - Ubuntu](https://doc.ubuntu-fr.org/tutoriel/tout_savoir_sur_les_modules_linux)

[TP kernel linux - Paristech](https://perso.telecom-paristech.fr/duc/cours/)

[The Linux Kernel Module Programming Guide - TLDP](https://tldp.org/LDP/lkmpg/2.6/lkmpg.pdf)

[Modules Kernel Linux - Kali FR](https://www.kali-linux.fr/hacking/modules-kernel-linux)

[Write Linux Kernel Module - Thegeekstuff](https://www.thegeekstuff.com/2013/07/write-linux-kernel-module/)

___
Author : AnthonyF  
Modified : 06/03/2021
