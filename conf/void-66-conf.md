﻿# 66

## 1. Overview

> Sixty-six is a collection of system tools built around s6 and s6-rc created to make the implementation and manipulation of service files on your machine easier. It is meant to be a toolbox for the declaration, implementation and administration of services where separate programs can be joined to achieve powerful functionality with small amounts of code.

[project page](https://web.obarun.org/software/66/v0.6.0.0/)

66 manages services in trees, which are collections of services, analogous to runlevels or targets in other init systems/service managers.


## 2. Installation

- Install the `boot-66serv` and `void-66-services` packages. Currently these packages are in a [PR](https://github.com/void-linux/void-packages/pull/25743), so one needs to build them first.

`boot-66serv` contains the `boot@` module service, which, along with other frontend files and scripts, are used for the first stage of booting a system. It is rougly analogous to the runit-void package for runit as it initialises the system (setting hostname, timezone, open luks devices, etc) and starts agetty on tty1-4 by default. The package was created as a portable stage1 for 66 and is used in the [Obarun](http://obarun.org/) distribution.
The package also contains some scripts written in order to make the services work for Void Linux.

`void-66-services` contains service frontend files for Void Linux.
 

## 3. Configuration

There are two ways to create the recommended basic trees for 66, an automatic and a manual one. The automatic configuration is **strongly** recommended.

### 3.1. Automatic configuration

- Run the `66boot-initial-setup` script:

_(commands prefixed by # are given with elevated privileges - as root)_
```
# 66boot-initial-setup
```

The script creates the necessary trees, enables in them some services and created the target for the basic configuration file symlink.


### 3.2 Manual configuration

#### 3.2.1 The `boot` tree

- Create a mandatory **n**ew tree `boot` and enable the boot@system service in it:
_(commands prefixed by #  are given with elevated privileges - as root)_
```
# 66-tree -n boot  
# 66-enable -F -t boot boot@system
```
- Create a permanent boot@system configuration file:
```
# cp /etc/66/conf/boot@system/version/.boot@system /etc/66/conf/boot@system/version/boot@system
```

#### 3.2.2 The `default` tree
More services can be enabled in a different tree, that starts after the boot tree. **default** is a nice name for it, as it is used for the... default collection of services in void.

- Create the **n**ew tree, **E**nable it and make it **c**urrent:

```
# 66-tree -nEc default
```

- Enable services in the new tree -the switch-initutils services is recommended:
```
# 66-enable switch-initutils
```

#### 3.2.3 Using runit services

66 can work with the existing runit services. That is useful as there are not yet frontend service files for all the packages that have a runit service directory.
To use runit services, a separate runit tree can be created, the runit service enabled and started in it and make the tree start after default:

```
# 66-tree -nE runit
# 66-enable -t runit runit
# 66-tree -S default runit
```

The runit services are started the normal way, by symlinking the service directories under /var/service/.


### 3.3 Finalising configuration

Both methods lead to the same basic trees created and services enabled. But before changing the init system, some more configuration must happen.

- Edit the **/etc/66rc.conf** with a text editor, save it and re-enable the `boot@system` service file in the `boot` tree:

```
# 66-enable -t boot -F boot@system
```

Please consult the `boot@` man page and the comments of the configuration file. Wrong configuration can result in an unbootable or problematic system!

### 3.4 Switching to 66 from runit

To boot the system with 66 instead of runit after the configuration, you just add `init=/usr/bin/66` to the kernel commandline. To switch back, remove the line.