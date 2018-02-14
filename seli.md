##SELINUX

###Storia e caratteristiche:

Il Security-Enhanced Linux (**SELinux**) è un modulo di sicurezza del kernel Linux che fornisce un insieme di strumenti per utilizzare e monitorare il controllo degli accessi incluso il Mandatory Access Control **MAC**, tutto questo utilizzando i framework Linux Security Modules **LSM**.

I concetti base di SELinux possono essere ricondotti ad alcuni progetti della **National Security Agency** (NSA) statunitense

La maggior parte dei sistemi operativi usano un sistema di controllo degli accessi
discrezionale **DAC**, che stabilisce come i soggetti (processi) interagiscono tra di loro e con gli oggetti(file,dir e dispositivi.
Nei sistemi operativi che usano DAC, gli utenti controllano i permessi sui file (oggetti) di loro proprietà.
Per esempio, sui sistemi operativi Linux®, gli utenti possono rendere la loro home directory leggibile a
tutti, dando accesso agli altri utenti ed ai processi ad informazioni potenzialmente sensibili.

SELinux aggiunge al kernel un Mandatory access control. Questo MAC definisce uno strato di amministrazione delle policy di sicurezza su tutti i processi e i file del sistema, utilizzando delle labels (etichette) che possono essere anche finemente descritte. MAC  

Gli utenti e le politiche di SELinux non devono essere correlati agli utenti e alle politiche del sistema Linux. Per ogni utente o processo, SELinux assegna un **context** a tre stringhe costituito da **nome utente**, **ruolo** e **dominio** (o **tipo**)
