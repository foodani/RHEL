## Preparing the HSM/Partition to Use STC

To establish an STC connection between partition and client, you must first enable STC on the HSM (depending on your HSM firmware version), create one or more partitions and export their partition identities. These operations are performed by the HSM SO.

Connect to the appliance via SSH or a serial connection, and log in to LunaSH as admin

Log in as HSM SO
```
lunash:> hsm login
```

Enable HSM Policy 39: Allow Secure Trusted Channel. If you are using Luna HSM Firmware 7.7.0 or newer, this policy has been removed; skip this step.
```
lunash:> hsm changepolicy -policy 39 -value 1
```

Create one or more new partitions for the client
```
lunash:> partition create -partition <partition_name> [-size <bytes>]
```

For each partition, export the partition identity public key to the Luna Network HSM file system. The file will be named with the partition's serial number. The command syntax is different depending on the Luna software/firmware version:

Luna 7.7.0 or newer:
```
lunash:> partition stcidentity export -partition <partition_name>

lunash:>partition stcidentity export -partition app_par1
Successfully exported partition identity for partition app_par1 to file: 154438865304.pid
```

[Optional] View the partition identity public key hash. If you are not the client administrator, it is recommended that you provide it (via separate channel) so that the client administrator can verify the key's integrity as described in Creating a Client-Partition STC Connection. The command syntax is different depending on the Luna software/firmware version:

Luna 7.7.0 or newer:
```
lunash:> partition stcidentity show -partition <partition_name>
```

If the client administrator does not have admin access to the appliance, or a firewall prevents you from using pscp or scp, you must transfer these files from the HSM and provide them to the client administrator by other secure means:

The HSM Server Certificate (server.pem) from the Luna Network HSM.

The partition identity public key for each partition the client will access (154438865304.pid in the example above).

[Optional] The partition identity public key hash for each partition the client will access. This is recommended so that the client can verify the key's integrity before using the partition. Do not send the hash by the same means as the certificates.

## Preparing the Client to Use STC

To access partitions on the HSM using STC, you must first create an STC token and identity on the client. These operations are performed by the client administrator.

>CAUTION!   If you already have STC connections to partitions on other HSMs, skip this procedure and use the existing client token/identity. If you re-initialize an existing client token/identity, active STC connections to this client will be broken.

Open a command prompt or terminal and navigate to the Luna HSM Client directory.
```
Windows: C:\Program Files\SafeNet\LunaClient
Linux/AIX: /usr/safenet/lunaclient/bin
Solaris: /opt/safenet/lunaclient/bin
```

[Optional] Launch LunaCM and verify that the STC client token is uninitialized.
```
lunacm:> stc tokenlist
```

Initialize the STC client token, specifying a token label.
```
lunacm:> stc tokeninit -label <token_label>
```

Create a client identity on the token.
```
lunacm:> stc identitycreate -label <client_identity>
```
The STC client identity public key is automatically exported to:
```
<client_install_directory>/data/client_identities/
```

## Creating a Client-Partition STC Connection

On the client workstation, use pscp or scp to import the HSM Appliance Server Certificate (server.pem) from the appliance. You require the appliance's admin password to complete this step.
```
pscp admin@<host/IP>:server.pem <target_filename>
```

Register the HSM Server Certificate with the client, using the vtl utility from the command line or shell prompt. If using a host name, ensure the name is reachable over the network (ping <hostname>). Thales recommends specifying an IP address to avoid network issues.
  ```
  vtl addServer -n <Network_HSM_hostname/IP> -c <server_certificate>
  ```

  [Optional] To check that you have successfully registered the appliance with the client, display the list of registered servers.
  ```
  vtl listServers
  ```
  
  Use pscp or scp to import the partition identity public keys for all partitions you will access with STC. The files are named with the partition serial number (<partitionSN>.pid). You require the appliance's admin password to complete this step.
  
  Register the partition identity public key to the client. Specify the path to the key file and, optionally, a label for the partition identity.
  ```
  lunacm:> stc partitionregister -file <partition_identity> [-label <partition_label>]

lunacm:> stc partitionregister -file /usr/safenet/lunaclient/data/partition_identities/154438865304.pid -label app_par1

Partition identity 154438865305 successfully registered. 
  ```
  Repeat this step for each partition identity public key you wish to register to this client.
  
  [Optional] If the HSM SO provided the partition identity public key hash, verify that the hashes match.
  ```
  lunacm:> stc identityshow
  ```
  
  Display the list of registered Luna Network HSM servers to find the server ID of the appliance that hosts the partition(s).
  ```
  lunacm:> clientconfig listservers
  ```
  
  Enable the STC connection.
  ```
  lunacm:> stc enable -id <server_ID>
  ```
  LunaCM restarts. If successful, the partition appears in the list of available slots.
  
  [Optional] Set the active slot to the new partition and verify the STC link.
  ```
  lunacm:> slot set -slot <slot>
lunacm:> stc status
  ```
