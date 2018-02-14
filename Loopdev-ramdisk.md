#Loop device e RAMdisk

###Loop device
Esempio:



1. creo un file da 1gb:

		dd if=/dev/zero of=filezero bs=1024 count=1000000

2. comando losetup:

		losetp /dev/loop1 /root/filezero

3. Ora posso trattarlo come un block device, formattandolo montandolo etc.


###RAM DISK

Esempio:

1. Vediamo quanta ram disponibile:

		free -g #o -m (in Mb)
2. Creo il mount point

		 mkdir /ramdisk

3. Monto e creo il ram disk

		mount -t tmpfs -o size=200Mb tmpfs /ramdisk