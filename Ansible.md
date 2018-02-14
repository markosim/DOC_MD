#Ansible howto


###Installazione di ansible

Il control node può essere un sistema Linux o UNIX. Microsoft Windows non è supportato come control node, sebbene i sistemi Windows possano essere gestiti come host. Deve essere installato ***Python 2*** (almeno 2.6)
poi basta eseguire

	#yum install -y ansible


Gli host gestiti Linux e UNIX devono avere Python 2 (versione 2.4 o successiva) installato per far funzionare la maggior parte dei moduli.

Comando base per testare un modulo
	
	#ansible "nomehost" -m modulo
	
esempio:
	
	#ansible rheltest -m ping

Host rheltest DEVE essere nell' inventory.


----------

###Inventario
L' inventory statico di default è sotto /etc/ansible/hosts, esempio di inventory statico:

	
	# Ex 1: Ungrouped hosts, specify before any group headers.
	green.example.com
	blue.example.com
	## 192.168.100.1
	## 192.168.100.10

	# Ex 2: A collection of hosts belonging to the 'webservers' group

	[webservers]
	alpha.example.org
	beta.example.org
	192.168.1.100
	192.168.1.110

	# If you have multiple hosts following a pattern you can specify
	# them like this:

	## www[001:006].example.com

	# Ex 3: A collection of database servers in the 'dbservers' group

	[dbservers:children]
	webservers
	dev
	## db01.intranet.mydomain.net
	## db02.intranet.mydomain.net
	## 10.25.1.56
	## 10.25.1.57

	# Here's another example of host ranges, this time there are no
	# leading 0s:

	## db-[99:101]-node.example.com
	[dev]
	rhel-test2
	rheltest

Posso specificare un diverso inventory da usare con l' opzione -i.

	#ansible rheltest -i ./inventory -m ping

Posso eseguire un modulo specificando i gruppi nell'inventory (il modulo sarà  eseguito su ciascun membro del gruppo), es:

	#ansible dev -m ping

Posso vedere da quali host è composto un gruppo(compresi eventuali childeren group):

	[root@redserver ~]# ansible dbservers --list
		  hosts (5):
		alpha.example.org
    	beta.example.org
    	192.168.1.100
    	192.168.1.110
    	rhel-test2

----------

###File di configurazione di ansible:

Per vedere il file di configurazione attualmente usato:

	[root@redserver ~]# ansible --version
	ansible 2.4.1.0
	  config file = /etc/ansible/ansible.cfg
  	configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  	ansible python module location = /usr/lib/python2.7/site-packages/ansible
  	executable location = /usr/bin/ansible
  	python version = 2.7.5 (default, Aug  4 2017, 00:39:18) [GCC 4.8.5 20150623 (Red Hat 4.8.5-16)]


Ansible di default cerca il file di configurazione in 4 path con il seguente ordine:

1. Nella variabile d'ambiente **$ANSIBLE_CONFIG**
2. Nel file **./ansible.cfg** nel path nel quale è staot eseguito il comando ansible
3. Nel file **~/.ansible.cfg** nella home dell' utente che lancia il comando
4. Nel path di default **/etc/ansible/ansible.cfg**


il file di configurazione è suddiviso in sezioni definite da identificativi racchiuse in square brackets

	[root@redserver ~]# grep '^\[' /etc/ansible/ansible.cfg
	[defaults]
	[inventory]
	[privilege_escalation]
	[paramiko_connection]
	[ssh_connection]
	[persistent_connection]
	[accelerate]
	[selinux]
	[colors]
	[diff]

Esempio di un file di configurazione di ansible in cui l' utente "someuser" tramite ssh si gestisce i managed hosts con l' autenticazione via chiave ssh, l' utente someuser è abilitato lanciare comandi come root senza inseirre password(ovviamente tale direttiva deve essere configurata in ogni managed host).

	[defaults]
	inventory = ./inventory			##Path del Ansible inventory.
	remote_user = someuser			##Utente remoto abilitato alla connessione verso i managed hosts
	ask_pass = false				##Prompt per la password disabilitato

	[privilege_escalation]
	become = true					##Enable or disable privilege escalation per le operazioni sui managed hosts
	become_method = sudo			##Il metodo di privilege escalation usato sui managed hosts
	become_user = root				##Lo user account to escalate privileges to on managed hosts
	become_ask_pass = false			##Definisce se il privilege escalation avviene tramite richiesta di password
	
Il protocollo di connessione di default ai managed hosts è settato su smart che implicita l' uso della connessione  via ssh. Ansible configura una entry di default nell' inventory: localhost che non è affetta dal parametro di configurazione "remote _user" perchè eseguito in locale.




----------


###Playbook

 Playbook sono dei "libri" dai quali Ansible "legge" le istruzioni da eseguire sui nodi destinatari. 
I Playbook possono anche definire dei ruoli, che eseguiranno altri Playbook per completare la configurazione.

La sintassi YAML prevede l' uso di una specifica indentazione che viene seguita utilizzando gli "spazi" non i "tab", utilizzata per gestire gerarchicamente gli attributi del playbook.
Esempio playbook:

	# This playbook deploys the whole application stack in this site.
	---
	- name: apply common configuration to all nodes
  	  hosts: all

	  tasks:
    - name: "install packages"
      yum: name:"{{ item }}" state=present
      with_items:
        - binutils
        - compat-libcap1
        - compat-libstdc++-33
        - gcc
        - gcc-c++
        - glibc
        - glibc-devel
        - ksh
        - libaio
        - libaio-devel
        - libgcc
        - libgcc.x86_64
        - libstdc++.x86_64
        - libstdc++-devel.x86_64
        - libXi.x86_64
        - libXtst.x86_64
        - make.x86_64
        - sysstat.x86_64
        - unixODBC.x86_64
        - unixODBC.i686
        - unixODBC-devel.x86_64
        - unixODBC-devel.i686

    - name: "create directory"
      file:
        name: "{{ item }}"
        owner: oracle
        group: oracle
        state: directory
        mode: 0775
      with_items:
        - /sw
        - /sw/oracle
        - /sw/oraInventory



E' possibile verificare la sintassi del playbook con il comando:
	
	#ansible-playbook --syntax-check playbook.yml


Utile per la sintassi dei moduli da richiamare è il comando:

	#ansible-doc -l      #lista tutti i moduli 
ad esempio per vedere la sintassi da utilizzare per il modulo service:

	#ansible-doc service
Con il comando ansible-doc è possibile vedere alcuni METADATI riguardo i moduli, in particolare chi sono stati sviluppati e mantenuti (ansible community etc) o il loro stato ( se sono deprecati, preview o stabili) 


Esempio di playbook:

	[root@redserver ansible]# cat playtest2.yml
	---
	- name: "install and start Apache HTTPD"
  	  hosts: rhel-test2
      gather_facts: False

  	  tasks:
        - name: "Primo play: Verifico httpd installato"
          yum:
            name: httpd
            state: installed

        - name: "Copio il file index.html"
          copy:
            src: /tmp/index.html
            dest: /var/www/html/index.html

    	 - name: "Avvio del webserver"
           service:
             name: httpd
             state: started
             enabled: true

	- name: "Secondo play: Verifco presenza e abilitazione firewalld"
      hosts: dbservers

      tasks:
        - name: "firewalld installato"
          yum:
            name: firewalld
            state: present

        - name: "verifico firewalld enabled"
          service: 
            name: firewalld
            status: started
            enabled: yes
	...

i moduli possono essere definiti anche in questo modo es. ultimo modulo service:

	- name "verifico firewalld enabled"
	  service: { name: firewalld, status: started, enabled: yes }


Il valore stringa "name" (del commento) può o meno essere messo sotto singole quote'' o doppie quote""
Per quanto riguarda la definisione degli host, in caso di molteplici, possono essere definiti oltre che così:
	
	hosts:
    - servera
    - serverb
    - serverc

anche in questa maniera:

	hosts: [servera, serverb, serverc]


E' importante utilizzare i moduli appositi per mantenere la natura idempotente di ansible, quindi magari evitare di usare i moduli command, shell, and raw .


###Variabili
E' possibile utilizzare variabili in ansibile per asegnare dianmicamente delle entry ad oggetti come del tipo:
Utenti da creare, pacchetti da installare, servizi da startare, ip da configurare, file da rimuovere, archivi da recuperare da internet, etc etc...

A seconda dello scopo possono essere definite a 3 diversi livelli:

1. Livello globale, definite via riga di comando oppure nell' ansible configuration file.
2. Livello di playbook, definite nel playbook e quindi nelle strutture relative (es. tasks).
3. Livello  di host, definite negli host group o host individuali, tramite l' inventario,fact gathering o task registrati.

 **Definizione e utilizzo a livello di playbook**

Si possono definire all'inizio del playbook:
	
	- hosts: all
	  vars:
        user: joe
	    home: /home/joe

oppure possono essere definite in file yaml esterni:

	- hosts: all
	  vars_files:
	    - var/user.yml

il file dovrà seguire la sintassi e l'indentazione yaml, ad esempio:

	cat var/user.yml
	---	
	user: joe
	home: /home/joe
per utilizzare le variabili:

	host: all
	vars:
	  user: joe
	tasks:
	  name: "Crea l'utente {{ user }}"
	  user:
	    name: "{{ user }}"

importante mettere le doppie quote altrimenti ansible lo interpreterà come la definizione di un dizionario.


 **Definizione a livello di host group o host singoli**

Nel file inventory posso definire per host singolo:

	[servers]
	server1.dominio.local user: joe

oppure per gruppo di host:

	[servers]
	server1.dominio.local
	server2.dominio.local
	
	[servers:vars]
	user: pippo

oppure per gruppi di sottogruppi:

	[serversA]
	server1.dominio.local
	server2.dominio.local

	[serversB]
	server3.dominio.local
	server4.dominio.local

	[servers:children]
	serversA
	serversB

	[servers:vars]
	user=joe

**Importante:**
Le variabili definite a livello di host "comandano" su quelle definite a livello di gruppo; quelle definite a livello di playbook comandano su quelle definite nei file di inventory.


E' altamente consigliato invece di modificare il file inventory creare due directory host_vars e group_vars all' interno delle quali editare file yaml col nome corrispondente all hostname e al gruppo per le quali vengono definite tali variaibili. Esempio cartella project  :


	project
	├── ansible.cfg
	├── group_vars
	│   ├── servers
	│   ├── serversA
	│   └── serversB
	├── host_vars
	│   ├── server1.dominio.local
	│   ├── server2.dominio.local
	│   ├── server3.dominio.local
	│   └── server4.dominio.local
	├── inventory
	└── playbook.yml 



**Definizione a livello di command line:**
si può forzare l'uso di eventuali variabili con il comando (viene eseguito il playbook soltanto sul host demo2 definendo la variabile package:

	ansible-playbook main.yml --limit=demo2.example.com -e "package=apache"


###Variabili e Array

E' possibile definire degli array piuttosto che utilizzare numerose variabili definendole in questo modo in un file yaml, quindi invece che usare tutte queste varaibili:

	user1_first_name: Bob
	user1_last_name: Jones
	user1_home_dir: /users/bjones
	user2_first_name: Anne
	user2_last_name: Cook
	user3_home_dir: /users/acook

Posso definire i seguenty array:

	users:
	  bjones:
        first_name: Bob
        last_name: Jones
    	home_dir: /users/bjones
      acook:
        first_name: Anne
        last_name: Cook
        home_dir: /users/acook

e richiamarli con questa sintassi:

	# Returns 'Bob'
	users['bjones']['first_name']
	
	# Returns '/users/acook'
	users['acook']['home_dir']

oppure con questa (ma occhio a non confoderla con altre statement python)

	# Returns 'Bob'
	users.bjones.first_name
	
	# Returns /user/acook
	users.acook.home_dir

Registrare variabili:
Gli amministratori possono registrare l'output di determinati comandi per vari scopi (debug etc..) utilizzando register: install_result al termine di un task e creando il seguente task successivo per debug, ad esempio:
tasks:

	  - name: "Verifica latest version pkg"
	    yum:
	      name: "{{ item }}"
          state: latest
          with_items: [ firewalld, httpd, php, php-mysql, mariadb-server ]
	      register: install_result

	 - debug: var=install_result


Esempio di playbook con variabili dichiarate e richiamate nei vari task:

	  name: Deploy and start Apache HTTPD service
	  hosts: webserver
	  vars:
	    web_pkg: httpd
	    firewall_pkg: firewalld
	    web_service: httpd
	    firewall_service: firewalld
	    python_pkg: python-httplib2
	    rule: http

	  tasks:
	    - name: Required packages are installed and up to date
	      yum:
	        name:
	          - "{{ web_pkg  }}"
	          - "{{ firewall_pkg }}"
	          - "{{ python_pkg }}"
	        state: latest

	    - name: The {{ firewall_service }} service is started and enabled
	      service:
	        name: "{{ firewall_service }}"
	        enabled: true
	        state: started

	    - name: The {{ web_service }} service is started and enabled
	      service:
	        name: "{{ web_service }}"
	        enabled: true
	        state: started

	    - name: Web content is in place
	      copy:
	        content: "Example web content"
	        dest: /var/www/html/index.html

	    - name: The firewall port for {{ rule }} is open
	      firewalld:
	        service: "{{ rule }}"
	        permanent: true
	        immediate: true
	        state: enabled

	- name: Verify the Apache service
	  hosts: localhost
	  become: false
	  tasks:
	    - name: Ensure the webserver is reachable
	      uri:
	        url: http://servera.lab.example.com
	        status_code: 200

###Ansible facts:
Sono dellle varibili specifiche di un managed host recuperate e valorizzate attraverso il modulo setup che gira di default(a meno che non venga disablitato nel playbook o nel file di configurazione generale di ansible). Queste variabili possono essere usate come altre varibili definite nei modi precedentemente descritti.
Alcune delle più usate sono queste:

	Fact    									Variable
	Short hostname							ansible_hostname
	Fully-qualified domain 					name ansible_fqdn
	Main IPv4 address (based on routinng)	ansible_default_ipv4.address
	list of the names of all net int.		ansible_interfaces
	Main disk first partition size			ansible_devices.vda.partitions.vda1.size
	A list of DNS servers					ansible_dns.nameservers
	Version of the currently run kernel		ansible_kernel

Un modo per visualizzare i facts valorizzati è questo:

	---
	- hosts: all
  	  tasks:
  		- name: Prints various Ansible facts
          debug:
          msg: >
            The default IPv4 address of {{ ansible_fqdn }} 
            is {{ ansible_default_ipv4.address }}

Ricordiamoci che quando il valore di una variabile è estratto da un array/dizionario si può richiamare in questi due modi:

**ansible\_default\_ipv4.address** può essere scritto **ansible\_default\_ipv4['address']**

come anche 

**ansible\_devices.vda.partitions.vda1.size** può essere scritto **ansible_devices['vda']['partitions']['vda1']['size']**

Per disabilitare l'esecuzione automatica del gathering dei facts su tutti gli hosts si può inserire nel playbook:

	---
	- name: This play gathers no facts automatically
	  hosts: large_farm
	  gather_facts: no

ma se si vuole abilitare per un singolo host basterà inserire il task **setup**:

	tasks:
    - name: Manually gather facts
      setup:

Si possono filtrare i dati recuperati col gather ad esempio per prendere inforamzioni sui dischi, sulle interfacce di rete o sugli utenti , per esempio per recuperare solo le info sulla configurazione dell'eth0:

	[user@demo ~]$ ansible demo1.example.com -m setup -a 'filter=ansible_eth0'
	demo1.lab.example.com | SUCCESS => {
	"ansible_facts": {
        "ansible_eth0": {
            "active": true,
            "device": "eth0",
            "ipv4": {
                "address": "172.25.250.10",
                "broadcast": "172.25.250.255",
                "netmask": "255.255.255.0",
                "network": "172.25.250.0"
            },
            "ipv6": [
                {
                    "address": "fe80::5054:ff:fe00:fa0a",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "52:54:00:00:fa:0a",
            "module": "virtio_net",
            "mtu": 1500,
            "pciid": "virtio0",
            "promisc": false,
            "type": "ether"
        }
    },
    "changed": false
}



Comandi:

	ansible "hostname" -m "modulo -a 'argomento del modulo o eventuali filtri'
	es.
	ansible host1 -m command -a "hostname"
	
	ansible-playbook "playbook"
	ansible-playbook -C playtest3.yml --limit=host1     (-C non fa operazioni, --limit limita esecuizione sul host indicato

###Inclusioni di task e variabili da file esterni

Utilizzato soprattutto ad esempio quando invoco un ruolo e con la direttiva when vedo il tipo di sistema operativo del managed host e includo dei task /variabili ad hoc per quel sistema operativo. 

In un playbook possono essere inclusi task e variabili da file esterni in questo modo, usando la direttiva **include**:


	tasks:
  	- name: Include tasks to install the database server
      include: tasks/db_server.yml

Il modulo include_vars può includere variaibli definite in un file JSON o YAML che prevaricano le variabili definite nel playbook stesso

 	tasks:
  	- name: Include the variables from a YAML or JSON file
      include_vars: vars/variables.yml

**Esempio di inclusione di task**

Nel playbook includiamo il file task environment.yml con il modulo **include**

	---
	- name: Install, start, and enable services
  	  hosts: all
  	  tasks:
  	- name: Includes the tasks file and defines the variables
      include: environment.yml
      vars:
      	package: mariadb-server
      	service: mariadb
      	state: started
    	register: output

  	- name: Debugs the included tasks
      debug:
        var: output

il file environment.yml sarà di questo tipo (prenderà le variabili definite nel task stesso del playbook):

	- name: Installs the {{ package }} package
  	  yum:
        name: "{{ package }}"
        state: latest

	- name: Starts the {{ service }} service
  	  service:
        name: "{{ service }}"
        state:  "{{ state }}"

**Esempio di inclusione di variabili**

file yaml variabili paths.yml 
	
	---
	paths:
  	  fileserver: /home/student/srv/filer/{{ ansible_fqdn }}
      dbpath: /home/student/srv/database/{{ ansible_fqdn }}

il playbook al cui interno, in un task viene richiamato il file paths.yml con il modulo **include_vars**

	---
	- name: Configure fileservers
  	  hosts: fileservers
  	  tasks:
    	- name: Imports the variables file
      	  include_vars: paths.yml

    - name: Creates the remote directory
      file:
        path: "{{ paths.fileserver }}"
        state: directory
        mode: 0755
      register: result

    - name: Debugs the results
      debug:
        var: result

### Iterazione con i cicli:

Cicli semplici con direttiva with_items:

	- name: Postfix and Dovecot are running
	  service:
	    name: "{{ item }}"
	    state: started
	  with_items:
	    - postfix
	    - dovecot
La lista usata da with_items può anche essere definita con una variabile:

	vars:
 	 mail_services:
  	   - postfix
       - dovecot

	tasks:
  	  - name: Postfix and Dovecot are running
        service:
          name: "{{ item }}"
          state: started
        with_items: "{{ mail_services }}"

oppure derivare da dizionari:

	- name: Users exist and are in the correct groups 
	  user:
	    name: "{{ item.name }}"
	    state: present
	    groups: "{{ item.groups }}"
	  with_items:
	    - { name: 'jane', groups: 'wheel' }
	    - { name: 'joe', groups: 'root' }

###Ansible condition:

Esempi:

1.

	---
	- hosts: all
	  vars:
	    run_my_task: true

	  tasks:
	    - name: httpd package is installed
	      yum:
	        name: httpd
	      when: run_my_task

2.

	---
	- hosts: all
	  vars:
	    my_service: httpd

	  tasks:
	    - name: "{{ my_service }} package is installed"
	      yum:
	        name: "{{ my_service }}"
	      when: my_service is defined



Table 5.2. Esempi di condizioni:

	Operator								Example

	Equal (value is a string)					ansible_machine == "x86_64"
	Equal (value is numeric)					max_memory == 512
	Less than									min_memory < 128
	Greater than								min_memory > 256
	Less than or equal to						min_memory <= 256
	Greater than or equal to					min_memory >= 512
	Not equal to								min_memory != 512
	Variable exists								min_memory is defined
	Variable does not exist						min_memory is not defined
	Variable is set to 1,True,or yes			available_memory
	Variable is set to 0,False,or no			not available_memory
	First variable's value is present as
	 a value in second variable's list			my_special_user in superusers


Esempio:

		--
		- hosts: all
		  vars:
		    my_special_user: devops

		    superusers:
		      - root
		      - devops
		      - toor

		  tasks:
		    - name: Task runs if my_special_user is in superusers
		      user:
		        name: "{{ my_special_user }}"
		        groups: wheel
		        append: yes
		      when: my_special_user in superusers

Condizioni multiple:

	ansible_kernel == 3.10.0-327.el7.x86_64 and inventory_hostname in groups['staging']
	ansible_distribution == "RedHat" or ansible_distribution == "Fedora"
	(ansible_distribution == "RedHat" and ansible_distribution_major_version == 7) or (ansible_distribution == "Fedora" and ansible_distribution_major_version == 23)

Interazione di Condizioni su Cicli:

	- name: install mariadb-server if enough space on root
	  yum:
	    name: mariadb-server
	    state: latest
	  with_items: "{{ ansible_mounts }}"
	  when: item.mount == "/" and item.size_available > 300000000

per ogni ansible mounts/partizione verifica che sia  "/" e che ci sia spazio.

	- hosts: all
	  tasks:
	    - name: Postfix server status
	      command: /usr/bin/systemctl is-active postfix 1
	      ignore_errors: yes2
	      register: result3

	    - name: Restart Apache HTTPD if Postfix running
	      service:
	        name: httpd
	        state: restarted
	      when: result.rc == 04

il primo task verifica che postfix sia running e registra il result. Il secondo task restarta httpd se il return code del modulo command è 0 (exit status)

###Ansible Handlers

Gli handlers ansible sono una sorta di task che vengono avviati quando un "trigger" evento definito in un task li attiva.
Vengono comunque lanciati solo quando tutti i task del playbook sono terminati.

Esempio1:

	tasks:
	  - name: copy demo.example.conf configuration template
	    copy:
	      src: /var/lib/templates/demo.example.conf.template
	      dest: /etc/httpd/conf.d/demo.example.conf
	    notify:
	      - restart_apache

	handlers:
	  - name: restart_apache
	    service:
	      name: httpd
	      state: restarted

il task copia il file di configurazione e notifica un "restart apache" all handlers che segue, terminati gli altri eventuali task viene avviato l handler restart_apache. L handler mantiene l' idempotenza nel senso che se il playbook viene rilanciato ma non avviene una change perchè il file è già stato copiato non viene triggerato l'evento e quindi l' handler non eseguito. 



Esempio2: 

	tasks:
	  - name: copy demo.example.conf configuration template
	    copy:
	      src: /var/lib/templates/demo.example.conf.template
	      dest: /etc/httpd/conf.d/demo.example.conf
	    notify:
	      - restart_mysql
	      - restart_apache

	handlers:
	  - name: restart_mysql
	    service:
	      name: mariadb
	      state: restarted

	  - name: restart_apache
	    service:
	      name: httpd
	      state: restarted


In questo play il task copia triggera due handlers trattandoli come array...eseguendoli come un ciclo. Questi vengono cmq avviati sempre dopo tutti i task del playbook e nell'ordine con i quali sono strutturati sotto handlers:

Imprtanti cose da considerare per gli handlers:

- Gli handlers vengono avviati sempre nell' ordine in cui è strutturata la sezione **handlers:** del play non nell' ordine del notify

- Gli handlers vengono avviati sempre dopo l'esecuzione di tutti i task del playbook presenti nella sezione **task:**

- Se due handlers hanno erroneamente lo stesso nome solo uno verrà avviato.

- Handlers specificati in file inclusi con **include** non verranno notificati(triggerati)

- Se più di un task notifica un hadler questo verrà cmq  avviato solo una volta. Se nessun task notifica un handler questo non veràà eseguito.

- Se un task con la notifica al handler non viene eseguito (ad esempio perchè un pacchetto è già stato installato) (o meglio viene eseguito ma non ha effetti "changed") **Ansible notifies handlers only if the task acquires the CHANGED status**


Esempio3:

	---
	- name: Installing Mariadb server
	  hosts: databases
	  vars:
	    db_packages:
	      - mariadb-server
	      - MySQL-python
	    db_service: mariadb
	    src_file: "http://materials.example.com/task_control/my.cnf.template"
	    dst_file: /etc/my.cnf

	  tasks:
	    - name: Install {{ db_packages }} package
	      yum:
	        name: "{{ item }}"
	        state: latest
	      with_items: "{{ db_packages }}"
	      notify:
	        - start_service
	    - name: Download and install {{ dst_file }}
	      get_url:
	        url: "{{ src_file }}"
	        dest: "{{ dst_file }}"
	        owner: mysql
	        group: mysql
	        force: yes
	      notify:
	        - restart_service
	        - set_password
	
	  handlers:
	    - name: start_service
	      service:
	        name: "{{ db_service }}"
	        state: started
	
	    - name: restart_service
	      service:
	        name: "{{ db_service }}"
	        state: restarted
	
	    - name: set_password
	      mysql_user:
	        name: root
	        password: redhat