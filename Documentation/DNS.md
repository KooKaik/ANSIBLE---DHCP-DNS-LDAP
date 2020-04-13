# DNS

### Installation des paquets
Pour mettre en place notre serveur DNS, il faut installer les paquets de BIND .
`yum install bind bind-utils`


### Configuration des fichiers de zone du DNS
Pour le bon fonctionnement du DNS, il est nécessaire de configurer 3 fichiers. 

`/etc/named.conf` permet de définir les différentes zones.
```
options {
	listen-on port 53 { 127.0.0.1;10.0.0.2; };
	// listen-on-v6 port 53 { ::1; };
	directory "/var/named";
	dump-file "/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	recursing-file "/var/named/data/named.recursing";
	secroots-file "/var/named/data/named.secroots";
	allow-query { localhost;10.0.0.0/24; };

	/*
	- If you are building an AUTHORITATIVE DNS server, do NOT 			enable recursion.
	- If you are building a RECURSIVE (caching) DNS server, you need to enable recursion.
	- If your recursive DNS server has a public IP address, you MUST enable access control to limit queries to your legitimate users. Failing to do so will cause your server to become part of large scale DNS amplification attacks. Implementing BCP38 within your network would greatly reduce such attack surface
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.root.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
	channel default_debug {
		file "data/named.run";
		severity dynamic;
	};
};

zone "." IN {
	type hint;
	file "named.ca";
};

//include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

zone "domain.lan" IN {
	type master;
	file "db.domain.lan";
	allow-update { key DDNS_UPDATE; };
	allow-query { any; };
};

zone "0.0.10.in-addr.arpa" IN {
	type master;
	file "db.0.0.10.in-addr.arpa";
	allow-update { key DDNS_UPDATE; };
	allow-query { any; };
};
```

`/etc/named/db.domain.lan` represente la zone forward
```
$TTL            604800
$ORIGIN         domain.lan.
@               IN      SOA     srv-dns01.domain.lan.   root.domain.lan. (
                1               ; Serial
                604800          ; Refresh
                86400           ; Retry
                2419200         ; Expire
                604800 )        ; Negative Cache TTL
;
@               IN      NS      srv-dns01.domain.lan.
@               IN      A       10.0.0.2
srv-dns01       IN      A       10.0.0.2
srv-dhcp01      IN      A       10.0.0.3
srv-ldap01      IN      A       10.0.0.4

```

`/etc/named/db.0.0.10.in-addr.arpa` represente la zone reverse
```
$TTL 	604800
$ORIGIN 0.0.10.in-addr.arpa.
@ 		IN 	SOA 	srv-dns01.domain.lan. root.domain.lan. (
		1 			; Serial
		604800 		; Refresh
		86400 		; Retry
		2419200 	; Expire
		60480 ) 	; Negative Cache TTL
;
@		IN	NS		srv-dns01.domain.lan.
2		IN	PTR		srv-dns01.domain.lan.
3		IN	PTR 	srv-dhcp01.domain.lan.
4		IN	WWW		srv-ldap01.domain.lan.

```

### Test du serveur DNS

Pour tester notre serveur DNS, on peut dans notre cas utiliser les commandes `dig srv-dns01.domain.lan` et `dig -x 10.0.0.2`


Exemple de `dig srv-dns01.domain.lan`
```
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-9.P2.el7 <<>> srv-dns01.domain.lan
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9328
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;srv-dns01.domain.lan.          IN      A

;; ANSWER SECTION:
srv-dns01.domain.lan.   604800  IN      A       10.0.0.2

;; AUTHORITY SECTION:
domain.lan.             604800  IN      NS      srv-dns01.domain.lan.

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: mar. mars 31 10:19:38 CEST 2020
;; MSG SIZE  rcvd: 79
```
Exemple de `dig -x 10.0.0.2`

```
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-9.P2.el7 <<>> -x 10.0.0.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44831
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;2.0.0.10.in-addr.arpa.         IN      PTR

;; ANSWER SECTION:
2.0.0.10.in-addr.arpa.  604800  IN      PTR     srv-dns01.domain.lan.

;; AUTHORITY SECTION:
0.0.10.in-addr.arpa.    604800  IN      NS      srv-dns01.domain.lan.

;; ADDITIONAL SECTION:
srv-dns01.domain.lan.   604800  IN      A       10.0.0.2

;; Query time: 0 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: mar. mars 31 10:22:00 CEST 2020
;; MSG SIZE  rcvd: 114
```