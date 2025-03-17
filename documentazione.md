# Progetto sistemi e reti - Documentazione fila 2
Indice:
1. [Utenti e password](#utenti-e-password)
2. [Configurazione rete](#configurazione-rete)
3. [pfSense](#pfsense)
  	* [DHCP](#dhcp)
  	* [SSH](#ssh)
  	* [VPN](#vpn)
4. [Server Windows](#server-windows)
  	* [DNS](#dns)
  	* [Active Directory](#active-directory)
6. [Server Ubuntu](#server-ubuntu)
  	* [Nginx](#nginx)

## Utenti e password
||Username|Password|
|-|-|-|
|Ripristino active directory - Utente windows server|Administrator|Admin2021|
|Server Ubuntu|fila2|admin2021|
|Client Windows (Amministratore, locale)|Alessandro Corsista||
|Client Ubuntu|administrator|admin2021|
|pfsense|admin|admin2021|

Gli utenti standard sono configurati per il dominio `5c.fila2.it`, appartengono al gruppo `Studenti` e sono accessibili sia da Windows che da Linux.\
Agli utenti vengono applicate le [policy](#active-directory) del dominio.

## Configurazione rete
I server hanno indirizzi IP statici:
* PfSense -> `172.22.0.1`
* Windows server -> `172.22.0.2`
* Windows server (backup) -> `172.22.0.3`
* Ubuntu server -> `172.22.0.4`
Tutti gli altri PC hanno indirizzi IP dinamici, stabiliti dalle regole [DHCP](#dhcp).

## pfSense
PfSense è un server basato su FreeBSD che svolge la funzione di Firewall e gateway per le reti esterne.
### DHCP
Rete: `172.22.0.0/16`\
Pool indirizzi: `172.22.0.5` -> `172.22.0.254`\
DNS servers:
* `172.22.0.2`
* `172.22.0.4`
* `100.2.0.254`

Gateway: `172.22.0.1`\
[Domain name](#active-directory): `5c.fila2.it`

### SSH
SSH permette la comunicazione sicura fra due dispositivi attraverso un canale crittografato (chiavi asimmetriche).\
Su [pfSense](#pfsense) è stata impostata una regola di port forwarding per l'accesso al server ubuntu da cattedra.