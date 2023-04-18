# Windows Client installation

>NOTE   
CSP or KSP registration includes a step that verifies the DLLs are signed by our certificate that chains back to the DigiCert root of trust G4 (in compliance with industry security standards).
This step can fail if your Windows operating system does not have the required certificate. If you have been keeping your Windows OS updated, you should already have that certificate.
If your Luna HSM Client host is connected to the internet, use the following commands to update the certificate manually:
```
certutil -urlcache -f http://cacerts.digicert.com/DigiCertTrustedRootG4.crt DigiCertTrustedRootG4.crt
certutil -addstore -f root DigiCertTrustedRootG4.crt
```
>To manually update a non-connected host
1.Download the DigiCert Trusted Root G4 ( http://cacerts.digicert.com/DigiCertTrustedRootG4.crt DigiCertTrustedRootG4.crt ) to a separate internet-connected computer.
2.Transport the certificate , using your approved means, to the Luna Client host into a <downloaded cert path> location of your choice
3.Add the certificate to the certificate store using the command:
```
certutil -addstore -f root <downloaded cert path>
```
  
## To install the Luna HSM Client software and drivers for all Luna devices and all features
  
From the location of LunaHSMClient.exe run the following command:

Install the full Luna HSM Client software with drivers for all Luna HSMs (Luna Network HSM, Luna PCIe HSM, Luna Backup HSM, Remote PED), as well as all the features (CSP/KSP, JSP, JCProv, C++ SDK, SNMP Subagent)
```
LunaHSMClient.exe /install /quiet ADDLOCAL=all
```
  
>NOTE   
You can omit the /quiet option to see all options in the GUI dialog.
  
[Optional logging] Install the full Luna HSM Client software with drivers for all Luna HSMs (Luna Network HSM, Luna PCIe HSM, Luna Backup HSM, Remote PED, as well as all the features (CSP/KSP, JSP, JCProv, C++ SDK, SNMP Subagent), and log the process.
```
LunaHSMClient.exe /install /log install.log /quiet ADDLOCAL=all
```
  
>NOTE     
The setting /log is optional and saves the installation logs to the file named install.log in the example. The install.log file (whatever name you give it) is required only if troubleshooting an issue with Thales GroupTechnical Support.

## Installing the Luna HSM Client for the Luna Network HSM

Use the ADDLOCAL=NETWORK option to install the base client software for the Luna Network HSM. Include the values for any optional, individual software components you desire. The base software must be installed first.
  
From the location of LunaHSMClient.exe run one of the following commands:

Install the base Luna HSM Client software necessary to communicate with Luna Network HSM
```
LunaHSMClient.exe /install /quiet ADDLOCAL=NETWORK
```

[Optional] Install the base Luna HSM Client software and any of the optional components for the Luna Network HSM that you desire:

For example, the following command installs the base software and all of the optional components:
```
LunaHSMClient.exe /install /quiet ADDLOCAL=NETWORK,CSP_KSP,JSP,SDK,JCProv
```
LunaHSMClient.exe /install /quiet ADDLOCAL=NETWORK,CSP_KSP,JSP,SDK,JCProv

If you wish to install only some of the components, just specify the ones you want after the product name (NETWORK in this example).
  
## Installing the Luna HSM Client for the Luna Backup HSM
  
Use the ADDLOCAL=BACKUP option to install the base client software for the Luna Backup HSM, and the optional feature, if desired. For the Backup HSM, which performs backup and restore operations and is not enabled for use with cryptographic applications, the feature you might add is SNMP, if applicable in your environment.
  
From the location of LunaHSMClient.exe run one of the following commands:

Install the base Luna HSM Client software for Luna Backup HSM
```
LunaHSMClient.exe /install /quiet /norestart ADDLOCAL=BACKUP
```
  
Install the base Luna HSM Client software and an optional component for the Luna Backup HSM:

For example, the following command installs the base software and the optional component:
````
LunaHSMClient.exe /install /quiet /norestart ADDLOCAL=backup
```

## Installation Location
  
Specify the installation location, if the default location is not suitable for your situation.

This applies to installation of any Luna Device. Provide the INSTALLDIR= option, along with a fully qualified path to the desired target location. For example:
```
LunaHSMClient.exe /install /quiet addlocal=all installdir=c:\lunaclient
```
  
That command silently installs all of the Luna device software and features to the folder c:\lunaclient (in this example). The software is installed into the same subdirectories per component and feature, under that named folder, as would be the case if INSTALLDIR was not provided. That is, INSTALLDIR changes the prefix or primary client installation folder to the one you specify, and the libraries, devices, tools, certificate folders, etc. are installed in their predetermined relationship, but under the new main folder location.
 
## Logging
  
If problems are encountered during installation or uninstallation of the software and you wish to determine the reason, or if Thales Technical Support has requested you to do so, detailed logs can be generated and captured by specifying the /log option and providing a filename to capture the log output. Two logs are generated – one according to the name given and the other similarly named, with a number appended. Both log files must be sent to Thales support if assistance is required.

Example commands that include logging are:
```
LunaHSMClient.exe /install /quiet /log install.log /norestart ADDLOCAL=backup,snmp
LunaHSMClient.exe /uninstall /quiet /log uninstall.log
```
