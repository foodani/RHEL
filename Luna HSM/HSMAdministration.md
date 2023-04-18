# Recover from STM

Ensure that you have the two strings that were presented when the HSM was placed into STM, or that were emailed to you if this is a new HSM.

If the HSM is initialized, log in as the HSM SO. If this is a new or zeroized HSM, skip to the next step.
```
lunash:> hsm login
```

Recover from STM, specifying the random user string that was displayed when the HSM was placed in STM, or that was emailed to you if this is a new HSM:

```
lunash:> hsm stm recover -randomuserstring <XXXX-XXXX-XXXX-XXXX>
```

You are presented with a verification string. Visually compare the string with the original verification string that was sent via e-mail (or other means).
If the string matches the original verification string, the HSM has not been used or otherwise altered since STM was enabled, and can be safely re-deployed.
Enter proceed to recover from STM.

# Initializing a New or Factory-reset HSM

Log into LunaSH as admin. You can use a serial terminal window or SSH connection.

Run the hsm init command, specifying a label for your Luna Network HSM:
```
lunash:> hsm init -label <label>
```
Respond to the prompts to complete the initialization process:

•on a password-authenticated HSM, you are prompted for the HSM password and for the HSM Admin partition cloning domain string (cloning domains for application partitions are set when the application partitions are initialized).

•on a multifactor quorum-authenticated HSM, you are prompted to attend to the Luna PED to create a new HSM SO (blue) PED key for this HSM, re-use an HSM SO PED key from an existing HSM so that you can also use it to log in to this HSM, or overwrite an existing key with a new authentication secret for use with this HSM. You are also prompted to create, re-use, or overwrite the Domain (red) PED key. You can create MofN quorum keysets and duplicate keys as required. See Multifactor Quorum Authentication for more information.

# Creating an Application Partition

Log in as HSM SO

```
lunash:> hsm login
```

Create the application partition, specifying a partition name. This name is distinct from the partition label assigned during initialization and can be changed later. You can also specify the desired partition size in bytes
Partition names created in LunaSH must be 1-32 characters in length. The following characters are allowed:
abcdefghijklmnopqurstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ 0123456789!@#$%^*()-_=+{}[]:,./?~
Spaces are allowed; enclose the partition name in double quotes if it includes spaces.
The following characters are not allowed: &\|;<>`'"?
No two partitions can have the same name.
```
lunash:> hsm show 
lunash:> partition create -partition <name> [-size <size> | -allfreestorage] [-version <1/0>]
lunash:> partition list
```
