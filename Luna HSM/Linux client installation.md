# Linux Luna HSM Client Installation

The installation script is install.sh and is usually launched with sh install.sh followed by any options or parameters.

interactive:
```
sh install.sh [-install_directory <prefix>]
```

all:
```
sh install.sh all [-install_directory <prefix>]
```

scriptable:
```
sh install.sh -p [network|pci|usb|backup|ped] [-c sdk|jsp|jcprov|snmp]|fmsdk|fm_tools [-install_directory </usr>]
```

The options on the script are:

device(s)

•network is the Luna Network HSM (software only, no drivers)

•pci is the Luna PCIe HSM (software plus PCI driver)

•usb is the Luna USB and Backup HSMs (software plus driver for the USB-connected HSMs)

•backup is software to enable Remote Backup

•ped is software for the Luna Remote PED

components include the optional Software Development kit, Java providers, SNMP instance (not needed for Luna Network HSM which has it built in), Functionality Module tools, and the Functionality Module SDK

### Install.sh syntax and options:

```
[myhost]$ sh install.sh help
 
usage:
install.sh      - Luna HSM Client install through menu
install.sh help - Display scriptable install options
install.sh all  - Complete Luna HSM Client install

install.sh -p [network|pci|usb|backup|ped] [-c sdk|jsp|jcprov|snmp|fmsdk|fm_tools] [-install_directory </usr>]

  -p <list of Luna products>
  -c <list of Luna components|all> - Optional. Default components are installed if not provided
  -install_directory <Defaults to /usr> - Optional. Sets the installation directory prefix.
Non-root install is restricted to installation of Luna Network HSM
product and Luna SDK, Luna JSP (Java) and Luna JCPROV (Java) components.

Luna products options
   network - Luna Network HSM
   pci     - Luna PCIe HSM
   usb     - Luna USB HSM
   backup  - Luna Backup HSM 
   ped     - Luna Remote PED

Luna components options
   sdk     - Luna SDK
   jsp     - Luna JSP (Java) --> Luna Network HSM, Luna PCIe HSM and Luna USB HSM default component
   jcprov  - Luna JCPROV (Java) --> Luna Network HSM, Luna PCIe HSM and Luna USB HSM default component
   snmp    - Luna SNMP subagent
```

By default, the Client programs are installed in the /usr/safenet/lunaclient directory.

## Flexible Install paths

An administrative (root) user, in charge of installing and uninstalling the software, has access wherever the installed material eventually resides. However, the operational, application-level use of Luna HSM Client might be assigned to a non-root user with constrained access and privileges. That non-root user might be a person or a departmental function or an application. By changing the install path to (for example) %home/bigapplication/safenet/luna you allow that non-root user access to tools and files for connecting to the HSM and using HSM partitions.

You can change the installation path for scriptable (non-interactive) installs by changing the prefix with the script option -install_directory <prefix>

The prefix, or major location is your choice, and replaces the /usr default portion. (See mention of SELinux, earlier on this page)
  
The script option -install_directory <prefix> is available for scriptable installation, where either "all" or a list of products and components is specified on the command line. The script option -install_directory <prefix> is not used with interactive installation; instead, you are prompted.

The /safenet/lunaclient portion is appended by the install script, and provides a predictable structure for additional subdirectories to contain certificate files, and optionally STC files.

Regardless of -install_directory <prefix> provided, some files are not affected by that option (for example, the Chrystoki.conf configuration file goes under /etc, service files need to be in the service directory expected by Linux in order to run at boot time, and so on).
  
## Controlling User Access to Your Attached HSMs and Partitions
  
By default, only the root user has access to your attached HSMs and partitions. You can specify a set of non-root users that are permitted to access your attached HSMs and partitions, by adding them to the hsmusers group.
  

  
