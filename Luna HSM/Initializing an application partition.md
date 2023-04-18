# Initializing a New Partition

## To initialize a new application partition in LunaSH on the Luna Network HSM appliance

The following steps assume that the Luna Network HSM admin has created the partition

In Lunash, log in to the HSM as SO if you are not already logged in.
```
lunash:> hsm login
```

Create the partition, if it has not already been created
```
lunash:> partition create -partition <partition name>
```

Initialize the partition by specifying its partition name. To initialize the partition using a policy template, specify the path to the template file.
```
lunash:> partition init -partition <name> [-applytemplate <template_file>] [-password <password>] [-domain <domain_string>]
```

# Initializing the Crypto Officer and Crypto User Roles

## To initialize the Crypto Officer role from the Client via lunacm

In LunaCM, log in to the partition as Partition SO
```
lunacm:> role login -name po
```

Initialize the Crypto Officer role. If you are using a password-authenticated partition, specify a CO password.
```
lunacm:> role init -name co
```

## To initialize the Crypto Officer role from the Network appliance via lunash

In LunaSH, log in to the HSM as SO if you are not already logged in.
```
lunash:> hsm login
```

Initialize the Crypto Officer role, providing the partition name, the PSO credential (already created) for that partition, and the credential for the CO that is being created. If you are using a password-authenticated partition, specify a CO password.
```
lunash:> partition init co -partition <partition name> -psopin <PSO'spassword> -copin <CO's password>
```
