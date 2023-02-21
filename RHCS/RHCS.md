# RHCS

## Prerequisites

```
sysctl crypto.fips_enabled
```
If the returned value is 1, FIPS mode is enabled. If the value is 0:
```
fips-mode-setup --enable
reboot
```
Verify if SELinux is Running in Enforcing Mode
```
getenforce
```
Opening the Required Ports in the Firewall
```
firewall-cmd --permanent --add-port={8080/tcp,8443/tcp,8009/tcp,8005/tcp}
firewall-cmd --reload
```
### Installing Red Hat Directory Server
```
subscription-manager repos --enable=dirsrv-11-for-rhel-8-x86_64-rpms
dnf module install redhat-ds
dnf install openldap-clients
dscreate create-template ds.inf
```
Create the setup configuration with the .inf file
```
dscreate from-file ds.inf
```
#### Enabling TLS Support in Directory Server

**Installing a CA Certificate**

For a certificate issued by a Certificate Authority (CA):
```
dsconf -D "cn=Directory Manager" ldap://server.example.com security ca-certificate add --file /root/ca.crt --name "Example CA"
dsconf -D "cn=Directory Manager" ldap://server.example.com security ca-certificate set-trust-flags "Example CA" --flags "CT,,"
```
Import the server certificate
```
dsconf -D "cn=Directory Manager" ldap://server.example.com security certificate add --file /root/instance_name.crt --name "Server-Cert" --primary-cert
```
For a self-signed certificate:
```
openssl rand -out /tmp/noise.bin 4096
certutil -S -x -d /etc/dirsrv/slapd-instance_name/ -z /tmp/noise.bin -n "Server-Cert" -s "CN=$HOSTNAME" -t "CT,C,C" -m $RANDOM --keyUsage digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment
certutil -L -d /etc/dirsrv/slapd-instance_name/ -n "Server-Cert" | egrep "Issuer|Subject"
```
Enable TLS and set the LDAPS port:
```
dsconf -D "cn=Directory Manager" ldap://server.example.com config replace nsslapd-securePort=636 nsslapd-security=on
```
Display the name of the server certificate in the NSS database:
```
dsconf -D "cn=Directory Manager" ldap://server.example.com security certificate list
```
To enable the RSA cipher family, setting the NSS database security device, and the server certificate name:
```
dsconf -D "cn=Directory Manager" ldap://server.example.com security rsa set --tls-allow-rsa-certificates on --nss-token "internal (software)" --nss-cert-name Server-Cert
```
```
dsctl instance_name restart
```
If encryption is enabled and a password set on the NSS database, Directory Server prompts for this password when the service starts. To bypass this prompt, you can store the NSS database password in the /etc/dirsrv/slapd-instance_name/pin.txt file. This enables Directory Server to start automatically without prompting for this password.
**The password is stored in clear text. Do not use a password file if the server is running in an unsecured environment.**
```
systemctl stop dirsrv@instance_name.service
```
Store the Directory Manager's password in the /etc/dirsrv/instance_name/password.txt file. For example:
```
echo password > /etc/dirsrv/slapd-instance_name/password.txt
chown dirsrv.dirsrv /etc/dirsrv/slapd-instance_name/password.txt
chmod 400 /etc/dirsrv/slapd-instance_name/password.txt
```
Store the Directory Manager's password in the /etc/dirsrv/instance_name/pin.txt file. For example:
```
echo "Internal (Software) Token:password" > /etc/dirsrv/slapd-instance_name/pin.txt
chown dirsrv:dirsrv /etc/dirsrv/slapd-instance_name/pin.txt
chmod 400 /etc/dirsrv/slapd-instance_name/pin.txt
```
To change the NSSDB password:
```
certutil -W -d /etc/dirsrv/slapd-instance_name/ -f /etc/dirsrv/slapd-instance_name/pwdfile.txt
```
```
systemctl start dirsrv@instance_name.service
```
Verify the TLS connection using the openldap-clients and NSS database:
```
LDAPTLS_CACERTDIR=/etc/dirsrv/slapd-instance_name \
	ldapsearch -H ldaps://$HOSTNAME:11636 \
	-x -D "cn=Directory Manager" -w Secret.123 \
	-b "dc=example,dc=org" -s base "(objectClass=*)"
```
If the above command doesn't work add this to the file /etc/openldap/ldap.conf:
```
TLS_REQCERT allow
```
