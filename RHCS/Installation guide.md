# Installation guide

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
### Installing Certificate System
Enable the Certificate System repository:
```
subscription-manager repos --enable certsys-10.x-for-rhel-8-x86_64-rpms
dnf module enable redhat-pki
```
**Subsystem Configuration Order**

At least one CA running as a security domain is required before any of the other public key infrastructure (PKI) subsystems can be installed.

Install the OCSP after the CA has been configured.

The KRA, and TKS subsystems can be installed in any order, after the CA and OCSP have been configured.

The TPS subsystem depends on the CA and TKS, and optionally on the KRA and OCSP subsystem.

```
yum install redhat-pki
yum update
```
**Interactive installation:**
```
pkispawn -s
```
### Two-step Installation
Create a text file for the configuration settings, such as /root/config.txt, and fill it with the settings described below.
Set the passwords of the Certificate System admin user, the PKCS #12 file, and Directory Server:
```
[DEFAULT]
pki_admin_password=password
pki_client_pkcs12_password=password
pki_ds_password=password
```
To use an LDAPS connection to Directory Server running on the same host, add the following parameters to the [DEFAULT] section in the configuration file:
```
pki_ds_secure_connection=True
pki_ds_secure_connection_ca_pem_file=path_to_CA_or_self-signed_certificate
```
If you use a self-signed certificate in Directory Server use the following command to export it from the Directory Server's Network Security Services (NSS) database:
```
certutil -L -d /etc/dirsrv/slapd-instance_name/ -n "server-cert" -a -o /root/ds.crt
```
**CA Settings**
To increase security, enable random serial numbers by adding the [CA] section with the following setting to the configuration file:
```
[CA]
pki_random_serial_numbers_enable=true
```
Optionally, set the following parameters in the [CA] section to specify the data of the admin user, which will be automatically created during the installation:
```
pki_admin_nickname=caadmin
pki_admin_name=CA administrator account
pki_admin_password=password
pki_admin_uid=caadmin
pki_admin_email=caadmin@example.com
```
To enable Certificate System to generate unique nicknames, set the following parameters in the [DEFAULT] section:
```
pki_instance_name=instance_name
pki_security_domain_name=example.com Security Domain
pki_host=server.example.com
```
Optionally, to use Elliptic Curve Cryptography (ECC) instead of RSA when generating certificates:

Add the following parameters to the [DEFAULT] section:
```
pki_admin_key_algorithm=SHA256withEC
pki_admin_key_size=nistp256
pki_admin_key_type=ecc
pki_sslserver_key_algorithm=SHA256withEC
pki_sslserver_key_size=nistp256
pki_sslserver_key_type=ecc
pki_subsystem_key_algorithm=SHA256withEC
pki_subsystem_key_size=nistp256
pki_subsystem_key_type=ecc
```
Add the following parameters to the [CA] section:
```
pki_ca_signing_key_algorithm=SHA256withEC
pki_ca_signing_key_size=nistp256
pki_ca_signing_key_type=ecc
pki_ca_signing_signing_algorithm=SHA256withEC
pki_ocsp_signing_key_algorithm=SHA256withEC
pki_ocsp_signing_key_size=nistp256
pki_ocsp_signing_key_type=ecc
pki_ocsp_signing_signing_algorithm=SHA256withEC
```
Add the following parameters to the [CA] section to override the RSA profiles with ECC profiles:
```
pki_source_admincert_profile=/usr/share/pki/ca/conf/eccAdminCert.profile
pki_source_servercert_profile=/usr/share/pki/ca/conf/eccServerCert.profile
pki_source_subsystemcert_profile=/usr/share/pki/ca/conf/eccSubsystemCert.profile
```
**Settings for Other Subsystems**
You need the following settings to install a subordinate CA, KRA, OCSP, TKS, or TPS:
```
pki_client_database_password=password
```
If you are installing a TPS:

The config file is different, read the man pkispawn page for all the configs.

Add the following section with the following section:
```
[TPS]
pki_authdb_basedn=basedn_of_the_TPS_authentication_database
```
Optionally, to configure that the TPS use server-side key generation utilizing a KRA that has already been installed in the shared CA instance, add the following entry to the [TPS] section:
```
pki_enable_server_side_keygen=True
```
**Starting the Installation Step**
```
pkispawn -f /root/config.txt -s subsystem --skip-configuration
```
Replace subsystem with one of the following subsystems: CA, KRA, OCSP, TKS, or TPS.

**Starting the Configuration Step**
After you have customized the configuration files according to https://access.redhat.com/documentation/en-us/red_hat_certificate_system/10/html/planning_installation_and_deployment_guide/two-step-installation#customizing_the_configuration_between_the_installation_steps, start the second step of the installation:
```
pkispawn -f /root/config.txt -s subsystem --skip-installation
```
Replace subsystem with one of the following subsystems: CA, KRA, OCSP, TKS, or TPS.
### Post-Installation

**TLS Cipher Configuration**

https://access.redhat.com/documentation/en-us/red_hat_certificate_system/10/html/planning_installation_and_deployment_guide/web-services-configuration-files#configuring-ciphers

**Enabling Certificate Revocation Checking for Subsystems**

https://access.redhat.com/documentation/en-us/red_hat_certificate_system/10/html/planning_installation_and_deployment_guide/web-services-configuration-files#enabling-ocsp-checking-for-the-tks-and-kra

**Enabling Signed Audit Logging**

https://access.redhat.com/documentation/en-us/red_hat_certificate_system/10/html/planning_installation_and_deployment_guide/Configuring_Logs_in_the_CS.cfg_File#enabling_and_configuring_signed_audit_logs

