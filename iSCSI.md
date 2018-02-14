#iSCSI

"Internet SCSI" è un protocollo di comunicazione che permette di inviare comandi a dispositivi di memoria SCSI fisicamente collegati a server e/o altri dispositivi remoti (come ad esempio NAS o SAN). Molto utilizzato in ambienti SAN poiché permette di consolidare l'archiviazione dei dati su dispositivi virtuali, collegati attraverso la rete, dando l'illusione di disporre localmente di un disco fisico che invece si trova in realtà su un dispositivo di storage remoto.

A differenza del protocollo Fibre Channel consente l'impacchettamento su TCP/IP rendendo così possibile l'utilizzo dell'infrastruttura di rete esistente che rende di fatto possibile l'utilizzo su distanze maggiori. E' inoltre molto più economico poichè non si usano fibre ottiche ma è cmq necessario/consigliato far viaggiare il traffico iscsi su segmenti lan dedicati.

Il client utilizza quindi un driver, detto **initiator**, che consente di inviare all'host dove sono fisicamente ospitati i dischi, detto **target**, i comandi che consentono di leggere e scrivere il disco virtuale. L'initiator tipicamente si identifica tramite un codice alfanumerico, detto **IQN** (acronimo inglese di "**iSCSI Qualified Name**". E' possibile implementare regole di accesso in base all' ip sorgente.

A differenza di nfs o cifs (usate per file/directory sharing) permette di presentare al client delle lun viste come **raw block disk device**. 

###Terminologia:

- **ACL**: access control list per controllare l'accesso a target LUN da parte di client.

- **Addressing**: Con iSCSI ad ogni server viene associato indirizzo univoco per ogni target server. Solitamente viene utilizzata la nomenclatura **IQN (iSCSI qualified name)**: Es: iqn.2015-01.com.example:testiscsilun0. Dove iqn identifica il format usato (iqn appunto) 2015-01 la data di registrazione del dominio example.com. Poi segue il dominio in reverse mode com.example e poi una stringa univoca a propria scelta.

- **Alias**: E' appunto un alias impostabile per identificare le LUN

- **Authentication**: iSCSI supporta 3 tipi di autenticazione
	-  **CHAP initiator authentication** (one way solo l' initiaztor si autentica).  
	-  **mutual CHAP authentication**  (two way - entrambi inittiator e target si autenticano tra loro).
	-  **Demo mode** , impostata di default per disabilitare l'autenticazione.


- **Backstore**: E' il device collegato localmente al server target, il backend della LUN presentata allo initiator. Può essere un disco intero (fisico o virtuale)(block device), una partizione(block device), una RAID partition(block), un LVM logical volume (block), un file (fileio) o un RAM disk

- **Initiator**: E' il client che accede alle LUN iSCSI. Può essere sia software che hardware. 
	-   Software, è un modulo del kernel che utilizza iSCSI per trattare una LUN iscsi come una block device
	-   Hardware è una scheda HBA dedicata che fa la stessa cosa dell' initiator software

- **iSNS**: Internet storage name server. E' il protocollo usato per lo scanning delle LUN sulla rete

- **LUN**: Logical Unit Number rappresentano un singolo indirizzabile scsi disk sharato del target. Dal punto di vista del initiator è semplicemnte un altro scsi disk disponibile. I programmi come fdisk e lvm lo trattano come un disco locale.

- **Node**: Rappresenta un initiator o un target, provvisto di IP o iSCSI address

- **Portal**: E' la combinazione dell' ip address e di una porta (tipo socket)alla quale è in ascolto il target server e alla quale inititator si collega. Di default è la porta 3260

- **Target**: E' un server che emula uno storage che condivide LUN utilizzate deall' initiator. Può essere un hardware RAID dedicato(storage appliance) o un server linux con il software caricati nel kernel. Il termine target può indicare una LUN o un insieme di LUN. Per non far confusione si distingue quindi un target-server e un target LUN

- **Target Portal LUN (TPG)**  Si intendo una op più network portal corrispondenti a determinate LUN.