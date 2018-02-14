#Installazione OpenLdap su Rhel/Centos 7

L’acronimo LDAP sta per Lightweight Directory Access Protocol;
Si tratta di una sorta di database che viene utilizzato nel momento in cui abbiamo bisogno di centralizzare delle utenze.

Il concetto di database su LDAP esce un po’ dalla logica dei database comuni come mysql perchè non si tratta di un database relazionale.
Ci sono molti servizi che si possono appoggiare a LDAP per poterne usufruire.
Se vogliamo fare qualche esempio possiamo citare la creazione di una rubrica centrale dove i vari client di posta elettronica possono accedere piuttosto che l’autenticazione a delle risorse condivise messe a disposizione tramite Samba, piuttosto che l’autenticazione su Apache.

La gestione delle informazioni in LDAP è basata sul concetto di entry. Un'entry è una raccolta di attributi che fanno riferimento ad un Distinguished Name (DN). Il DN è unico ed è usato per riferirsi inequivocabilmente all'entry.

##Configurazioni tipiche:



- Unico server che offre  servizi di directory solo in un unico dominio

![](http://www.di-srv.unisa.it/~ads/corso-security/www/CORSO-0203/openLDAP/immagini/configlocal.gif)



- Server che offre servizi di directory per il  dominio locale e lo si configura per ritornare riferimenti ad un servizio superiore capace di occuparsi di richieste esterne al dominio locale.

![](http://www.di-srv.unisa.it/~ads/corso-security/www/CORSO-0203/openLDAP/immagini/configref.gif)

pippo




Questi gli acronimi usati nella struttura LDAP:


- dn: Distingued Name
- uid: User id
- cn: Common Name
- sn: Surname
- l: Location
- ou: Organizational Unit
- o: Organization
- dc: Domain Component
- st: State
- c: Country

Struttura tipo:


                                   |dc=my,dc=organization,dc=org|		
               						/							\
							   	   /							 \
						| ou=people |						  |ou=groups|
						/           \
					   /             \
			|cn=Marco|             |cn=Luca|

###installazione pacchetti (openldap cliente e server + migrationtool):

    yum install -y openldap* migrationtool

Impostiamo ldap root password per l' amministrazione (mettiamo da parte l'hash della password):

	[root@hostname ~]# slappasswd
	New password:
	Re-enter new password:
	{SSHA}Kb/nyOAfg0f38mg0iiKH9neuR5QYxdKD


I file di configurazione si trovano sotto:
**/etc/openldap/slapd.d/**

	[root@hostname ~]# cd /etc/openldap/slapd.d/cn=config
	[root@hostname cn=config]# vi olcDatabase={2}hdb.ldif


Editiamo le righe seguenti in base alle preferenze:
	
	olcSuffix: dc=miodominio,dc=local
	olcRootDN: cn=ldapmanager,dc=miodominio,dc=local  #DN entry per utenza senza restrizioni
	

e aggiungiamo le seguenti righe:

	olcRootPW: {SSHA}Kb/nyOAfg0f38mg0iiKH9neuR5QYxdKD  #password dell'utente rootDN (root del Directory server)
	olcTLSCertificateFile: /etc/pki/tls/certs/miodominioldap.pem 
	olcTLSCertificateKeyFile: /etc/pki/tls/certs/miodominioldapkey.pem


Modificare il file **/etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif** per impostare i diritti di lettura del directory server.
	
	root@linux1 cn=config]# vi olcDatabase={1}monitor.ldif
	olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base=""cn=ldapmanager,dc=miodominio,dc=local" read by * none


Con il comando **#slaptest -u**  test della configurazione.

Start, Enable e Verfica del servizio:

	[root@linux1 cn=config]# systemctl start slapd
	[root@linux1 cn=config]# systemctl enable slapd
	[root@linux1 cn=config]# netstat -lt | grep ldap
	tcp        0      0 0.0.0.0:ldap            0.0.0.0:*               LISTEN
	tcp6       0      0 [::]:ldap               [::]:*                  LISTEN



