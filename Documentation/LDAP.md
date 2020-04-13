# LDAP

## OpenLDAP

### Installation des paquets

Pour mettre en place notre serveur LDAP, il faut d'abord installer différents paquets:
`yum install -y openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel`

### Configuration de OpenLDAP

Fichier pour la création du profil admin :  `db.ldif`
```
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=domain,dc=lan

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=domain,dc=lan

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: password
```
`ldapmodify -Y EXTERNAL  -H ldapi:/// -f db.ldif`


Fichier de mise en place du monitor: `monitor.ldif`
```
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base="cn=ldapadm,dc=domain,dc=lan" read by * none
```
`ldapmodify -Y EXTERNAL  -H ldapi:/// -f monitor.ldif`

### Application des schémas OpenLDAP

`ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif`
`ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif`
`ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif`

### Mise en place de la base de données

Fichier de configuration de la BDD : `base.ldif`
```
dn: dc=domain,dc=lan
dc: domain
objectClass: top
objectClass: domain

dn: cn=ldapadm ,dc=domain,dc=lan
objectClass: organizationalRole
cn: ldapadm
description: LDAP Manager

dn: ou=People,dc=domain,dc=lan
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=domain,dc=lan
objectClass: organizationalUnit
ou: Group
```
`ldapadd -x -W -D "cn=ldapadm,dc=domain,dc=lan" -f base.ldif`

### Création d'un profil utilisateur de test

 Nom du fichier de l'utilisateur: `testuser.ldif`
```
dn: uid=testuser,ou=People,dc=domain,dc=lan
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: testuser
uid: testuser
uidNumber: 9999
gidNumber: 100
homeDirectory: /home/testuser
loginShell: /bin/bash
gecos: testuser [Admin (at) domain]
userPassword: password
shadowLastChange: 17058
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
```
Ajout de l'utilisateur à la base de donnée LDAP
`ldapadd -x -W -D "cn=ldapadm,dc=domain,dc=lan" -f testuser.ldif`

### Test du serveur LDAP

Pour tester notre serveur LDAP, on peut utiliser une machine client.
Il faut d'abord taper pluusieurs commande avec de connecter notre machine client au serveur LDAP et autoriser l'utilisateur à se connecter.

`yum install -y openldap-clients nss-pam-ldapd`

Connection au serveur LDAP:
`authconfig --enableldap --enableldapauth --ldapserver=10.0.0.4 --ldapbasedn="dc=domain,dc=lan" --enablemkhomedir --update`

Autorisation du nouvelle utilisateur sur cette machine:
`getent passwd testuser`

On peut maintenant se connecter avec notre utilisateur LDAP sur la machine client.

## PHPLdapAdmin

### Installation du paquet
Pour mettre en place un interface graphique pour notre serveur LDAP, il faut d'abord installer ce paquet:
`yum install epel-release`

### Configuration PHPLdapAdmin

On peut ensuite configurer différents paramètres dans le fichier `/etc/httpd/conf.d/phpldapadmin.conf` comme le nom de domaine, le message de bienvenue sur l'interface ou alors l'utilisateur par défaut.

### Test de l'interface graphique

Pour se connecter sur l'interface graphique, il faut ouvrir un naviguateur et taper cette adresse dans notre cas:
`http://10.0.0.4/phpldapadmin`

On peut ensuite se connecter avec un utilisateur de notre serveur LDAP.