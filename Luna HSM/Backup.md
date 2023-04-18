## Recovering the Luna Backup HSM 7 from Secure Transport Mode

Connect the Luna Backup HSM 7 to a USB port on a client workstation running Luna HSM Client 10.1.0 or newer, with the Backup option installed (refer to Luna HSM Client Software Installation for your client operating system).

Launch LunaCM on the client workstation.

Select the slot assigned to the Luna Backup HSM 7 Admin partition.
```
lunacm:> slot set -slot <slot_id>
```

Recover the HSM from Secure Transport Mode. See Secure Transport Mode for more information about the Random User String:
```
lunacm:> stm recover -randomuserstring <string>
```

## Initializing a Luna Backup HSM 7 for Password Authentication

Open a network (SSH) or serial connection to the appliance and log in as admin, or other admin-level user, to start a LunaSH session.

Connect the backup HSM directly to the Luna Network HSM using the included USB cable.

Get the serial number of the backup HSM, or read the serial number from the Backup HSM display screen.
```
lunash:> token backup list
```

Initialize the backup HSM:
```
lunash:> token backup init -label <backup_hsm_label> -serial <backup_hsm_serial_number>
```

## Configuring the Luna Backup HSM 7 for FIPS Compliance

On the Luna HSM Client computer, run LunaCM.

Set the active slot to the Luna Backup HSM 7.
```
lunacm:> slot set -slot <slot_id>
```

Log in as Backup HSM SO
```
lunacm:> role login -name so
```

Set HSM policy 55: Enable Restricted Restore to 1.
```
lunacm:> hsm changehsmpolicy -policy 55 -value 1
```

[Optional] Check that the Luna Backup HSM 7 is now in FIPS 140-2 approved operation mode.
```
lunacm:> hsm showinfo
```

## Backing Up a Password-Authenticated Partition

Open a network (SSH) or serial connection to the appliance and log in as admin, or other admin-level user, to start a LunaSH session.

Get the serial number of the backup HSM, or read the serial number from the Backup HSM display screen.
```
lunash:> token backup list
```

Display a list of application partitions; you require the label for the partition you are backing up.
```
lunash:> partition list
```

If you plan to back up to an existing partition on the Backup HSM, display a list of the existing backups.
```
lunash:> token backup partition list -serial <backup_hsm_serial_number>
```

Initiate the backup operation:
```
lunash:> partition backup -partition <source_partition_label> -serial <backup_hsm_serial_number> [-tokenpar <target_backup_partition_label>] [-add | -replace]
```

>NOTE   You must specify -add or -replace when backing up to an existing backup partition. Use -add to add only new objects. Use -replace to add new objects and overwrite existing objects. You do not need to specify these options when backing up a V1 partition, as only the SMK is backed up.
If you omit the -tokenpar option when creating a new backup, the partition is assigned a default name (<source_partition_name>_<YYYYMMDD>) based on the source HSM's internally-set time and date.
If the backup operation is interrupted (if the Backup HSM is unplugged, for example), the Backup HSM's full available space can become occupied with a single backup partition. If this occurs, delete the backup partition with lunash:> token backup partition delete before reattempting the backup operation.
 
 Respond to the prompts for the following passwords:
 
 The Crypto Officer password for the source partition
 
 The HSM SO password for the backup HSM
 
 If you are creating a new backup, you must provide the domain string for the source partition -- it is used to initialize the new backup partition so that objects can be cloned. If your target is an existing backup partition, the operation will proceed only if the domains match.
 
 The backup begins once you have completed the authentication process. Objects are backed up one at a time.
