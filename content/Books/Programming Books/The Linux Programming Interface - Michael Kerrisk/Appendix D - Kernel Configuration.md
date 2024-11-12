---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Appendix D - Kernel Configuration
modified: 2024-11-11T21:37:49-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **D**  
**KERNEL CONFIGURATION**

Many features of the Linux kernel are components that can be optionally configured. Before compiling the kernel, these components can be disabled, enabled, or, in many cases, enabled as loadable kernel modules. One reason to disable an unneeded component is to reduce the size of the kernel binary, and thus save memory, if the component is not required. Enabling a component as a loadable module means that it will be loaded into memory only if it is required at run time. This likewise can save memory.

Kernel configuration is done by executing one of a few different _make_ commands in the root directory of the kernel source tree—for example, _make menuconfig_, which provides a _curses_-style configuration menu, or, more comfortably, _make xconfig_, which provides a graphical configuration menu. These commands produce a .config file in the root directory of the kernel source tree that is then used during kernel compilation. This file contains the settings of all configuration options.

The value of each option that is enabled is shown in the .config file in a line of the following form:

CONFIG_NAME=value

If an option is not set, then the file contains a line of this form:

# CONFIG_NAME is not set

In the .config file, lines beginning with a # character are comments.

Throughout this book, when we describe kernel options, we won’t describe precisely where in the _menuconfig_ or _xconfig_ menu the option can be found. There are a few reasons for this:

• The location can often be determined fairly intuitively by navigating through the menu hierarchy.

• The location of configuration options does change over time, as the menu hierarchy is restructured across kernel versions.

• If we can’t find the location of a particular option within the menu hierarchy, then both _make menuconfig_ and _make xconfig_ provide search facilities. For example, we can search for the string CONFIG_INOTIFY to find the option for configuring support for the _inotify_ API.

The configuration options that were used to build the currently running kernel are viewable via the /proc/config.gz virtual file, a compressed file whose contents are the same as the .config file that was used to build the kernel. This file can be viewed using _zcat(1)_ and searched using _zgrep(1)_. The /proc/config.gz file is itself only available if the kernel was configured with the CONFIG_IKCONFIG and CONFIG_IKCONFIG_PROC configuration options enabled.