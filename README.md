# MULTIPLE LOGINS ACTIVE DIRECTORY

### In this project, we configure the OpenLDAP on Linux and using the Active Directory to manage the the user access on OpenLDAP data.
### We will configure the Kerberos and Native LDAP to store the user and domain information.
### SSSD for the Security Services Daemon on LINUX machine


![1](https://user-images.githubusercontent.com/71564211/139781928-0a1e2685-71ac-43ee-8681-501d80defbd1.PNG)

## CONFIGURE DNS SERVICE ON WINDOW SERVER
### REVERSE LOOKUP ZONES
*	Click into the Server Manager
*	Then click to Roles -> DNS Server (Choose no if the warning message display)
*	Click into the DNS -> Your Domain name -> Right Click Reverse Lookup Zones -> New Zone -> Next -> Primary Zone -> Next -> Choose second option (To all DNS servers running on domain controllers in this domain bigguycorp.com) -> Next -> IPv4 Reverse Lookup Zones
*	Then Fill like the table below

![2](https://user-images.githubusercontent.com/71564211/139782095-61e55534-fb8b-4a31-90ce-4bf2b51ec7d1.PNG)

### FORWARD LOOKUP ZONES
*	Then click to Roles -> DNS Server (Choose no if the warning message display)
*	Click into the DNS -> Your Domain name -> Forward Lookup Zones -> choose domain (bigguycorp.com)
*	Right Click to the Domain name, then choose New Host AAA…
*	Then fill the information of Linux machine (abc-co) with proper IP address 192.168.10.4

![3](https://user-images.githubusercontent.com/71564211/139782097-d31f1396-eca5-4aba-90bf-bc23a651dee8.PNG)

### VERIFY THE CONNECTION

![4](https://user-images.githubusercontent.com/71564211/139782100-a7fc72cd-7df2-4a68-a8c6-9a23e702ba63.PNG)

## OpenLDAP SERVICE ON CentOS 8
*	hostnamectl set-hostname abc-co.abc.com
*	yum install yum-utils -y
*	yum-config-manager --add-repo=https://repo.symas.com/configs/SOFL/rhel8/sofl.repo
*	yum install openldap openldap-clients openldap-servers symas-openldap-clients symas-openldap-servers perl tar openssl --skip-broken
* slappasswd

![5](https://user-images.githubusercontent.com/71564211/139782327-68d3fbf2-c2a7-4669-a863-96b40c20eff2.PNG)

*	systemctl enable slapd
*	systemctl start slapd

## Configure OpenLDAP via the extension .ldif
Setup the LDAP database before move to next step: 
* ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
*	ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
*	ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

### CREATE EMPTY FOLDER TO STORE .LDIF FILES

#### password.ldif

![6](https://user-images.githubusercontent.com/71564211/139782531-31c71569-5651-4c2a-a608-1ef83f1b6c15.PNG)

#### base.ldif
![7](https://user-images.githubusercontent.com/71564211/139782537-fe8e0f5d-b9e8-405c-9866-7d48b621fa4e.PNG)

=> Execute the command below to add ldif file into Native LDAP:
ldapmodify -Y EXTERNAL -H ldapi:/// -f password.ldif

=> Execute the command below to add ldif file into Native LDAP:
ldapadd -H ldapi:/// -x -D "cn=abc-co,dc=abc,dc=com" -W -f base.ldif

#### people.ldif

![8](https://user-images.githubusercontent.com/71564211/139782604-7eba1e7d-e8be-4caa-ab89-fd82fc81f9cd.PNG)

=> Execute the command below to add ldif file into Native LDAP:
ldapadd -H ldapi:/// -x -D "cn=abc-co,dc=abc,dc=com" -W -f people.ldif

## Create LDAP user allows login to the system and put this user into the OU People:

![9](https://user-images.githubusercontent.com/71564211/139782768-3b8dfb00-ba27-4efa-bfd1-a0d3022d4cb1.PNG)

=> Execute the command below to add ldif file into Native LDAP:
ldapadd -H ldapi:/// -x -D "cn=abc-co,dc=abc,dc=com" -W -f user.ldif

## VERIFY LDAP DATABASE
* ldapsearch -x cn=projectuser1 -b dc=abc,dc=com

![10](https://user-images.githubusercontent.com/71564211/139782871-bb303555-29a5-408d-9826-00cbe051a315.PNG)

## ADDING PASSWORD FOR LDAP USER
* ldappasswd -x -D "cn=abc-co,dc=abc,dc=com" -W -S "uid=projectuser1,ou=People,dc=abc,dc=com"

## KERBEROS SERVICE FOR SYSTEM IDENTIFIER
*	yum install -y realmd sssd oddjob-mkhomedir adcli samba-common samba-common-tools krb5-workstation
*	vi /etc/krb5.conf

![11](https://user-images.githubusercontent.com/71564211/139783122-3ce454b1-6daf-4c54-97a1-94b418fd5f36.PNG)
![12](https://user-images.githubusercontent.com/71564211/139783140-8d41fea9-4db6-450d-a186-d7a20429f8c9.PNG)
![13](https://user-images.githubusercontent.com/71564211/139783141-7fa14a52-c2d5-4d7c-aa62-d496e58baf2b.PNG)
![14](https://user-images.githubusercontent.com/71564211/139783143-c02bed74-fa7a-4b4e-b24a-89715badc1b3.PNG)

## ESTABLISHING CONNECTION FROM CENTOS 8 TO WINDOW SERVER
* Check the network configuration. Be careful the DNS1 parameter (It is the WINDOW SERVER's IP ADDRESS)

![15](https://user-images.githubusercontent.com/71564211/139783302-9485b734-4f46-4a05-a0cf-12ea6651c40f.PNG)

* Restart the Network Manager ifdown ens33, ifup ens33
* Then the nameserver on DNS1 will appear on /etc/resolv.conf

![16](https://user-images.githubusercontent.com/71564211/139783452-d3637cfc-3cc8-405b-9717-735d3651b629.PNG)

## TEST SUCCESS CONNECTION 

![17](https://user-images.githubusercontent.com/71564211/139783521-e329f6d9-2e2c-47ff-bd11-4e4da3774e4e.PNG)

*	Open Server Manager -> Roles -> Active Directory Domain Services -> Active Directory Users and Computers -> Click into the domain name.
*	Right Click into Users -> New -> User -> Then create the User (bigguycorp)
*	Repeat the same step to create second user (testuser)
*	Right Click into Users, Create a Group called ‘BigGuyGroupAPL’
*	Then Right Click the the BigGuyGroupAPL created before, choose Properties -> Members -> Click Add -> and Add the user ‘bigguycorp’ created before into group.

![18](https://user-images.githubusercontent.com/71564211/139783639-9032c1b3-93d6-4018-8d77-73dc0ed5fbd6.PNG)

## ON CENTOS 8, SET THE PERMISSION TO ALLOW ACCESS INTO LDAP DATA WITHIN THE DOMAINNAME 
*	realm deny --all
*	realm permit –groups BigGuyGroupAPL

-> Furthermore permission rules, let find on realm policy commands

![19](https://user-images.githubusercontent.com/71564211/139783815-e9dd47eb-ac37-4382-8e60-c9b79896cd50.PNG)

![20](https://user-images.githubusercontent.com/71564211/139783816-4a62f26f-c243-4330-8c81-dc158281142c.PNG)




