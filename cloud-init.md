#Preparazione immagine rhel7.3 qcow2 per openstack

**Greazione della vm** (che sarà la nostra immagine da caricare su glance)

creare il volume che conterrà la vm (il vmdk di VMWARE)

	qemu-img create -f qcow2 vm_name.qcow2 50G

lanciare 

	virt-manager

eseguire dalla GUI VIRT-MANAGER installazione guidata della vm

seleziona reiso iso da usare per l'installazione e quando chiede su quale volume creare la vm selezionare il path ed il file .qcow2 creato prima

proseguire con l'installazione, e prima di riavviare smontare il cd virtuale (hda)

	virsh dumpxml vn_name


  centos
...
    
      
      
      
       
    
...


	virsh attach-disk --type cdrom --mode readonly vm_name "" hda

	virsh reboot centos

----

**Registrazione**


Una volta riavviata la vm, registrarla (altrimenti non è possibile scaricare i pacchetti necessari)

	subscription-manager register # (con le credenziali di partec)

	user 
	pass

	subscription-manager attach

	subscription-manager repos #(verificare repo disponibili)

----

**Installare acpid** (che servirà a nova per riavviare poweroff dell' istanza)

	#dhclient eth0 		#(se interfaccia della vm non è ancora configurata)
	#yum install acpid
	#systemctl enable acpid

-----

**Modifica sudoers**
	
Aggiungo utenza partec ai sudoers (verrà comunque aggiunta al gruppo wheel file cloud.cfg)

	visudo

	root    ALL=(ALL)       ALL
	partec  ALL=(ALL)       ALL   ### utente partec può fare sudo per tutti i comandi senza password

----

**Modifica file /etc/ssh/sshd_config**

Disabilitare accesso ssh da root e abilitare richiesta di certificato e contestualmente password solo per l' utente partec

	vi /etc/ssh/sshd_config

	....
	PermitRootLogin no
	....
	PubkeyAuthentication yes
	AuthenticationMethods publickey,password publickey,keyboard-interactive
	PasswordAuthentication yes

queste 3 righe per accesso ssh solo da utente partec con certificato E password

:> /etc/securetty  #### inibisce accesso per utente root

sudo passwd -l root

lock sulla password di root

yum install cloud-init

vi /etc/cloud/cloud.cfg

yum install cloud-utils-growpart

echo "NOZEROCONF=yes" >> /etc/sysconfig/network

vi  /etc/default/grub

modificare
 GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap rhgb quiet"
 con
 GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap console=tty0 console=ttyS0,115200n8"

grub2-mkconfig -o /boot/grub2/grub.cfg

subscription-manager unregister

poweroff

a vm spenta

virt-sysprep -d vm_name 

Rimuove MAC address info