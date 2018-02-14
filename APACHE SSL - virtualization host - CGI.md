#APACHE SSL - creazioni certificati per connessioni cifrate

###Creazione certificati 
Normalmente le connessioni http avvengono in chiaro. Per poter cifrare una connessionie http è neccessario che il server utilizzi il modullo apache/http ssl/tls per crifrare le connessioni con il client che di default (con i moderno browser supporterà tale connessione).

Solitamente un server esposto su internet offrirà un certificato digitale per garantire che è effettivamente il server "che dice di essere". Tale certificato digitale è composto da questi elementi:

- Una chiave pubblica
- Informazioni riguardanti la sua identità
- Una firma digitale di persona od organizzazione che certifica il legame tra informazioni identificative e chiave pubblica

Solitamente ci si rivolge ad una Certification Authority che ci fornisca dietro pagamento un ceritficato digitale.
Nel nostra situazione casalinga invece creeremo noi tramite gli strumenti di openssl il nostro certificato digitale.

Prima di tutto creiamo la nostre chiave privata:

	openssl genrsa -des3 -out my-CA.key 2048
digitiamo una pass phrase:
"esempio conf https"
verrà generata una chiave privata con l' algoritmo des3

a questo punto generiamo il certificato con la nostra chiave privata, con -new specifichiamo che si tratta di una nuova richiesta in modo da poter inserire tutti i dati relativi al certificato. L’opzione -key indica la chiave privata usata per firmare il certificato, -x509 il formato del certificato, -days la validità in giorni del certificato in questo caso 10 anni, -out il nome del file generato per contenere il certificato:

	openssl req -new -key my-CA.key -x509 -days 3650 -out my-CA.crt 
	Enter pass phrase for my-CA.key:
	You are about to be asked to enter information that will be incorporated
	into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [XX]:IT
	State or Province Name (full name) []:Italy
	Locality Name (eg, city) [Default City]:Rome
	Organization Name (eg, company) [Default Company Ltd]:Mia_company
	Organizational Unit Name (eg, section) []:Mia_cert_auth
	Common Name (eg, your name or your server's hostname) []:Mia_company_CA
	Email Address []:markos76@gmail.com


Per poter leggere testualmente un certificato possiamo usare il seguente comando:

	openssl x509 -in my-CA.crt -text -noout


###Creiamo un certificato per il server


Creazione chiave privata per il server:

	[root]# openssl genrsa -des3 -out my-server.key 1024
con la pass phrase: "esempio server https"

Passiamo a generare la richiesta di certificato my-server.csr. Per comodità vengono riportati in un unico riquadro sia il comando che la corrispondente risposta di OpenSSL

	[root@rhel-test2 cert]# openssl req -new -key my-server.key -out my-server.csr
	Enter pass phrase for my-server.key:
	You are about to be asked to enter information that will be incorporated
	into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [XX]:IT
	State or Province Name (full name) []:Italy
	Locality Name (eg, city) [Default City]:Rome
	Organization Name (eg, company) [Default Company Ltd]:
	Organizational Unit Name (eg, section) []:
	Common Name (eg, your name or your server's hostname) []:sec.rhel-test2.local
	Email Address []:markos76@gmail.com
	[root@rhel-test2 server_cert]# ll
	total 8
	-rw-r--r-- 1 root root 733 Jan 15 12:06 my-server.csr
	-rw-r--r-- 1 root root 963 Jan 15 12:00 my-server.key

**il Common Name fornito deve corrispondere all’indirizzo usato per la connessione SSL. In caso contrario i browser rileveranno un’incongruenza tra dati del certificato e sito web che fornisce tale certificato.**

Come ultimo passo torniamo a travestirci da CA e dalla richiesta di certificato appena creata, generiamo il certificato in formato X.509, firmato con la chiave privata della CA:


	[root@rhel-test2 cert]# openssl x509 -req -in my-server.csr -out my-server.crt -sha1 -CA my-CA.crt -CAkey my-CA.key -CAcreateserial -days 2190
	Signature ok
	subject=/C=IT/ST=Italy/L=Rome/O=Default Company Ltd/CN=sec.rhel-test2.local/emailAddress=markos76@gmail.com
	Getting CA Private Key
	Enter pass phrase for my-CA.key:
	
	[root@rhel-test2 cert]# ll
	total 24
	-rw-r--r-- 1 root root 1476 Jan 15 11:52 my-CA.crt
	-rw-r--r-- 1 root root 1751 Jan 15 11:47 my-CA.key
	-rw-r--r-- 1 root root   17 Jan 15 12:11 my-CA.srl
	-rw-r--r-- 1 root root 1184 Jan 15 12:11 my-server.crt
	-rw-r--r-- 1 root root  733 Jan 15 12:06 my-server.csr
	-rw-r--r-- 1 root root  963 Jan 15 12:00 my-server.key


Consiglio di proteggerli adeguatamente sia a livello di permessi (chmod 400) che predisponendone una copia di sicurezza. Risulta evidente il danno provocato da un’appropriazione indebita o da una loro perdita per un guasto hardware. A questo punto non ci resta che configurare Apache perché, utilizzando alcuni dei file generati, sia in grado di instaurare una connessione SSL

###Configurazione APACHE HTTPD:

Siamo quindi in possesso di: 


- my-CA.key: chiave privata della CA
- my-CA.crt: certificato della CA
- my-server.key: chiave privata del server
- my-server.csr: richiesta di certificato del server
- my-server.crt: certificato del server

a questo punto possiamo utilizzare il file /etc/httpd/conf.d/ssl.conf per definire il nostro virtual host:

	<VirtualHost 192.168.160.188:443>
	ServerAdmin markos76@gmail.com
	DocumentRoot /var/www/html-sec
	ServerName sec.rhel-test2.local
	ErrorLog /var/www/ssl_log/error_log
	# CustomLog /var/www/ssl_log/access_log
	CustomLog logs/ssl_request_log \
          "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"

	<IfModule mod_ssl.c>
	   SSLEngine on
	
	   SSLProtocol all -SSLv2
	
	   #certificato del server:
	   SSLCertificateFile /etc/httpd/ssl/my-server.crt
	
	   #chiave privata del server:
	   SSLCertificateKeyFile /etc/httpd/ssl/my-server.key
	
		   #chain del certificato del server:
	   SSLCertificateChainFile /etc/httpd/ssl/my-CA.crt

	   #Certificate Authority (CA):
	   SSLCACertificateFile /etc/httpd/ssl/my-CA.crt
	
	   SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+EXP
	
	   SetEnvIf User-Agent ".*MSIE.*" \
                         nokeepalive ssl-unclean-shutdown \
                         downgrade-1.0 force-response-1.0

	</IfModule>

	</VirtualHost>


a questo punto se restartiamo il servizio httpd possiamo collegarci in https alla url https://sec.rhel.test.local, vediamo che il nostro browser darà errore "sito non sicuro" perchè chiaramente non può verificare la CA poichè è quella nostra "casalinga".


**Importante: Ogni qual votla restartiamo il nostro server http dobbiamo inserire la pass phrase legata al nostro certifcato.** Nel caso specifico "esempio server https"
A meno che non si proceda come segue per registrare la pass phrase:

	[root]# cp my-server.key my-server.key.pass  
	[root]# openssl rsa -in my-server.key.pass -out my-server.key
	Enter pass phrase for my-server.key.pass:
	writing RSA key



##Virtual hosts
Possiamo creare all' interno della dir /etc/httpd/conf.d/ il file vhost1.conf così composto:


	# vi /etc/httpd/conf.d/vhost1.conf
	<VirtualHost *:80> 
	ServerAdmin admin@vhost1.example.com 
	DocumentRoot /var/www/html/vhost1 
	ServerName vhost1.example.com 
	ErrorLog logs/vhost1-error_log 
	CustomLog logs/vhost1-access_log combined 
	</VirtualHost> 

creando chiaramente la dir specificata nella documentroot. In questo modo possiamo creare molteplici virtual host all' interno del nostro server httpd.
Possiamo fare un check della configurazione dei Virtual host dando il comando :

	 httpd –D DUMP_VHOSTS 



###CGI
Possiamo fare eseguire comandi shell (per pagine dinamiche) mettendo degli script sotto /var/www/cgi-bin/ 

	# vi /var/www/cgi-bin/systime.sh 
	
	#!/bin/bash 
	echo “Content-type: text” 
	echo 
	echo “The current system time is `date`”

Se lo richiamiamo alla url http://www.rhel-test2/cgi-bin/systime.sh

Possiamo richiamre script da un altra locazione andando a modificare la direttiva ScriptAlias nel httpd.conf:
	
	ScriptAlias /cgi-bin/ “/var/dynpage/” 
	<Directory "/var/dynpage"> 
	AllowOverride None 
	Options None 
	Require all granted 
	</Directory> 