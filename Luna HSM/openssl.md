## OpenSSL integration

Download libp11 (https://github.com/OpenSC/libp11/releases)

Install 
```
yum groupinstall "Development Tools"
yum install openssl-devel
yum install pkg-config
```

Extract libp11

Launch the script
```
./configure --prefix=/usr
make
make check
make install
```

Edit the OpenSSL configuration file (/etc/ssl/openssl.cnf) to add the following lines:

At the top:
```
openssl_conf = openssl_init
```

At the bottom:
```
[openssl_init]
engines = engine_section

[engine_section]
pkcs11 = pkcs11_section

[pkcs11_section]
engine_id = pkcs11
dynamic_path = /usr/lib64/engines-1.1/pkcs11.so
MODULE_PATH = /usr/safenet/lunaclient/lib/libCryptoki2_64.so
init = 1
slot = 0

#dynamic_path = /opt/Thales/LunaClient/lib/libCryptoki2_64.so
#MODULE_PATH = /opt/Thales/LunaClient/lib/libLunaAPI.so
```
To find the pkcs11.so path run the command:
```
find /usr/lib* -name pkcs11.so
```

Run the following command to initialize the HSM token:
```
pkcs11-tool --module /usr/safenet/lunaclient/lib/libCryptoki2_64.so --init-token --label "My Token" --so-pin <SO_PIN> --pin <USER_PIN> -l
```
Replace <SO_PIN> with the Security Officer (SO) PIN for the HSM token and <USER_PIN> with the user PIN for the token. This command initializes the HSM token and sets the label to "My Token".

Run the following command to generate a key pair and save it inside the HSM token:
```
pkcs11-tool --module /usr/safenet/lunaclient/lib/libCryptoki2_64.so --keypairgen --key-type rsa:2048 --id <OBJECT_ID> --pin <USER_PIN> --label "My Key"

pkcs11-tool --module /usr/safenet/lunaclient/lib/libCryptoki2_64.so --keypairgen --slot 6 --key-type rsa:1024 --id 1 --pin zaq12wsx --label PECTITT_KEY -m SHA1-RSA-PKCS -l
```
Run the following combat to show the objects in the slot:
```
pkcs11-tool --module /usr/safenet/lunaclient/lib/libCryptoki2_64.so --slot <id> --login --pin <pin> --list-objects

or

/usr/safenet/lunaclient/bin/cmu list
```
Replace <OBJECT_ID> with the desired ID for the key object (such as "My Key") and <USER_PIN> with the user PIN for the token. This command generates a 2048-bit RSA key pair and saves it inside the HSM token with the specified ID and label.

Run the following command to generate a CSR using the key pair stored in the HSM:
```
openssl req -engine pkcs11 -keyform engine -key <OBJECT_ID> -new -out csr.csr
```
Replace <OBJECT_ID> with the ID of the key pair stored in the HSM token. This command generates a CSR using the key pair stored inside the HSM.

Run the following command to sign the CSR with the key in the HSM:
```
pkcs11-tool --module /usr/safenet/lunaclient/lib/libCryptoki2_64.so --sign --slot <id> --login --pin <pin> --key --input-file <path-csr> --output-file <path> --id <key-id>
```
