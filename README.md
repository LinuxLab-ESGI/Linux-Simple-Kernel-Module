# Linux-Simple-Kernel-Module

Short repo about Linux kernel modules.
We are going to see the main commands to manage kernel modules and develop a simple module for our system.

## Kernel modules

Kernel modules are separated codes that can be integrated (loaded or unloaded) into the Linux kernel on the demand without the need to reboot the system. The main advantage is the capacity to add features to the Kernel without recompiling all the Kernel.
It is usually called LKM (**L**inux **K**ernel **M**odule)

## Main commands

### Getting information

| Command                             | Description                   |
| ----------------------------------- | ----------------------------- |
| `lsmod`                               | Current loaded kernel modules |
| `modinfo module_name`                 | Current loaded kernel modules |
| `modprobe --show-depends module_name` | Current loaded kernel modules |

### Manually load a module

To load a module located in `/usr/lib/modules/$(uname -r)/` :  
`modprobe module_name`

If the module is not located in the specific folder, we can use this base command but it won't find automatically dependencies :  
`insmod /path/module_name`

### Manually unload a module

`modprobe -rf module_name`  
*-r remove*  
*-f force*

Or :  
`rmmod module_name`

### Automatically load a module

If we want to add or blacklist an module to the Kernel, we can specify a list of kernel modules to load in a conf file. Each module name are separated by a newline.  
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

```C
int var =1;
```
