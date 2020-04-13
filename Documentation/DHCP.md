# DHCP

### Installation du paquet

Il est nécessaire de installer le paquet DHCP pour mettre en place notre serveur.
`yum install dhcp`

### Configuration du DHCP

Pour finir de mettre en place sont DHCP, il suffit de définir son étendue dans le fichier de configuration : `/etc/dhcp/dhcpd.conf`

```
authoritative;
ddns-updates on;
ddns-update-style interim;
ignore client-updates;
update-static-leases on;
default-lease-time 600;
max-lease-time 7200;
log-facility local7;

include "/etc/dhcp/ddns.key";

zone domain.lan. {
	primary 10.0.0.2;
	key DDNS_UPDATE;
}

zone 0.0.10.in-addr.arpa. {
	primary 10.0.0.2;
	key DDNS_UPDATE;
}

subnet 10.0.0.0 netmask 255.255.255.0 {
	range 10.0.0.20 10.0.0.120;
	option subnet-mask 255.255.255.0;
	option routers 10.0.0.254;
	option domain-name-servers 10.0.0.2;
	option domain-name "domain.lan";
	ddns-domainname "domain.lan";
	ddns-rev-domainname "in-addr.arpa";
}
```

Dans la configuration ci-dessus, l'étendue DHCP est de 10.0.0.20 à 10.0.0.120

L'inclusion du fichier `/etc/dhcp/ddns.key`permet de faire un lien sécurisé entre le serveur DNS et DHCP afin de garantir un bon fonctionnement entre eux.

### Test à effectuer

Pour tester notre configuration, on peut simplement ajouter une machine client au domaine DNS et lui attribuer une adresse IP automatiquement.