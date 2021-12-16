---
marp: true
paginate: true
color: #ffff
backgroundColor: #2A2A2A
header: '![width:100px height:100px](./img/logo.png)'
footer: "**01/01/1970 - Author**"
author: AnthonyF
---
<style>
section {
  font-family: 'Century Gothic', serif !important;
}
</style>
<!-- _class: invert -->

# Simple module pour Kernel Linux <!-- fit -->

---
<!-- _class: invert -->

# Sommaire
- Modules Kernel
- Principales commandes
- Notre premier Hello-Bye World module

---
<!-- _class: invert -->

# Modules Kernel ou **LKM (Loadable Linux Kernel)**

- Code compile faisant partie du noyau Linux (souvant des drivers)
- Acces au controle total de la machine
- Acces aux appels systemes (Syscall : open, read, brk=>malloc)

==> Kernel space != User space

## Avantages
- Peut etre charge ou decharge (loaded/unloaded) au noyau
- Pas besoin de redemarrer le systeme ni de recompiler le noyau

---
<!-- _class: invert -->

## Modules de bases
- Systeme de fichier, protocoles reseaux, etc.

## Modules peripheriques
- Carte son, video, reseaux, etc.
  
## Modules supplementaires
- Carte video non-libre (Nvidia), Wi-fi, etc.

---
<!-- _class: invert -->

# Principales commandes

| Command                               | Description                            |
| ------------------------------------- | -------------------------------------- |
| `lsmod`                               | Show all loaded kernel modules         |
| `modinfo module_name`                 | Show information about a kernel module |
| `modprobe --show-depends module_name` | Show dependencies of a module          |


---
<!-- _class: invert -->

# Charger manuellement un module

- les modules sont localises dans  `/usr/lib/modules/$(uname -r)/` 
  
Pour les charger : `sudo modprobe module_name`

Si le module n'est dans le repertoire : `sudo insmod /path/module_name`



---
<!-- _class: invert -->

# Decharger manuellement un module

`sudo modprobe -rf module_name`  
*-r remove*  
*-f force*

ou :  
`sudo rmmod module_name`


---
<!-- _class: invert -->

# Charger ou "blacklister" un module avec un fichier de conf
- fichier de conf : `/etc/module-load.d/name_of_file.conf`

Exemple :  

``` Bash
cat /etc/modules-load.d/modules.conf 

# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
module_name1
module_name1
```


---
<!-- _class: invert -->

# "Blacklister" un module

- Eviter le chargement d'un module
- Ajout de l'option `blacklist` dans `/etc/modprobe.d/`

Exemple :  

``` Bash
cat /etc/modules-load.d/blacklist.conf 

# Not loaded modules
blacklist module_name1
blacklist module_name1
```

---
<!-- _class: invert -->

# Developpement de notre premier module <!-- fit -->

Les packets a installer :

- gcc
- make
- your kernel headers
- build-essential

Pour build-essential :  

``` Bash
sudo apt install gcc build-essential make linux-headers-`uname -r`
```
