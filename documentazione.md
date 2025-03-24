# Progetto sistemi e reti - Documentazione fila 2
Progettazione di una rete aziendale completa di server web, DNS, firewall, domini e VPN.

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
	* [RADIUS](#radius)
6. [Server Ubuntu](#server-ubuntu)
	* [Nginx](#nginx)
	* [Bind9 (router cattedra)](#bind9)

## Utenti e password
| Device                                              |       Username        |  Password   |
|-----------------------------------------------------|:---------------------:|:-----------:|
| Ripristino active directory - Utente windows server |    `Administrator`    | `Admin2021` |
| Server Ubuntu                                       |        `fila2`        | `admin2021` |
| Client Windows (Amministratore, locale)             | `Alessandro Corsista` |             |
| Client Ubuntu                                       |    `administrator`    | `admin2021` |
| pfsense                                             |        `admin`        | `admin2021` |

Gli utenti standard sono configurati per il dominio `5c.fila2.it`,
appartengono al gruppo `Studenti` e sono accessibili sia da Windows che da Linux.\
Agli utenti standard vengono applicate le [policy](#active-directory) del dominio.

## Configurazione rete
I server hanno indirizzi IP statici:
* PfSense -> `172.22.0.1`
* Windows server -> `172.22.0.2`
* Windows server (backup) -> `172.22.0.3`
* Ubuntu server -> `172.22.0.4`\
Tutti gli altri PC hanno indirizzi IP dinamici, stabiliti dalle regole [DHCP](#dhcp).

## pfSense
PfSense è un server basato su FreeBSD che svolge la funzione di firewall e gateway per la rete interna.\
É stato impostato il port forwarding su pfSense per consentire la visualizzazione dei siti da parte dei client esterni alla rete.
![Configurazione port forwarding](/img/port-forwarding/confPortFwd.png)

### DHCP
Rete: `172.22.0.0/16`\
Pool indirizzi: `172.22.0.5` -> `172.22.0.254`\
DNS servers:
* `172.22.0.2`
* `172.22.0.3`
* `100.2.0.254`

Gateway: `172.22.0.1`\
[Domain name](#active-directory): `5c.fila2.it`

### SSH
SSH permette la comunicazione sicura fra due dispositivi attraverso un canale crittografato con chiavi asimmetriche.\
Su [pfSense](#pfsense) è stata impostata una regola di port forwarding per l'accesso al server ubuntu da cattedra.

## Server Windows
Il server Windows rende disponibili diversi servizi, tra cui DNS, Active Directory e RADIUS.\
I servizi attivi vengono gestiti tramite l'aggiunta di ruoli e funzionalità, disponibili in `Server Manager`.

### DNS
Windows Server ha l’indirizzo di loopback come specifica del DNS primario.\
Indirizzo DNS secondario -> `172.22.0.3`.

Sono stati impostati i record del servizio DNS per la risoluzione degli URL:
* `www.fila2.it`
* `corsini.fila2.it`
* `bulgaru.fila2.it`
* `serafini.fila2.it`
* `www.sito2vecchio.com` (ALIAS di `www.fila2.it`)

Il file `named.conf.local` è stato inserito nella zona per `sito2vecchio.com`.

### Active Directory
Le configurazioni di Active Directory permettono la gestione centralizzata degli utenti, dei loro permessi e delle regole all'interno di un dominio di dispositivi.\
Sono state configurate le seguenti Group Policy:
* Impedure agli utenti di installare nuove stampanti
* Impedire la visualizzazione del nome dell’ultimo utente che si è connesso sui pc client
* Eliminare l'opzione spegnimento per gli utenti

### RADIUS

## Server ubuntu

### NGINX
I file di configurazione sono all'interno di `/etc/nginx`.
NGINX gestisce le configurazioni dei siti tramite file di testo.\
Nella cartella `sites-available` sono presenti i file di configurazione (.conf) dei siti gestiti da NGINX.\
Nella cartella `sites-enabled` sono presenti i symlink ai file .conf di `sites-available`.\
La gestione dei siti abilitati tramite symlink permette di risparmiare risorse e 
sincronizzare automaticamente le modifiche ai file di configurazione.

I file dei siti sono divisi per area del sito (anteporre a fila2.it) e contenuti in `/var/www/html`:
* `corsini/corsini.html`
* `bulgaru/bulgaru.html`
* `serafini/serafini.html`
* `fila2/fila2.html`

`www.fila2.it` contiene una pagina di login gestita con PHP.

`www.fila2.it` accetta solo richieste HTTPS (aperta la porta 443).\
File .conf di `www.fila2.it`:
![File di configurazione NGINX per un sito standard](/img/nginx/conf-www.fila2.it.png)

`sicuro.fila2.it` accetta le richieste HTTP e le reindirizza a `www.fila2.it` tramite HTTPS.\
File .conf di `sicuro.fila2.it`:
![File di configurazione NGINX per un sito sicuro](/img/nginx/conf-sicuro.fila2.it.png)

### BIND9
BIND9 è un server DNS che permette la risoluzione dei nomi di dominio in indirizzi IP.

I file di configurazione sono all'interno di `/etc/bind`.\
Viene usato dal server di cattedra (`100.2.0.254`) per rendere accessibili i siti di ciascun gruppo a tutte le file.\
Il server di cattedra reindirizza le richieste DNS risolte ai server pfSense delle rispettive file.\
Il file `named.conf.local` contiene le zone di configurazione del server DNS.\
Il file `named.conf.options` contiene le opzioni di configurazione del server DNS.

Il file `db.fila2.it` contiene i record DNS associati al dominio `fila2.it` (`bulgaru`, `corsini`, `serafini`, `sicuro`, `www`).\
Il file `db.sito2vecchio.com` è un record CNAME che punta al nuovo dominio `www.fila2.it`, in quanto deprecato.
Aggiunte le zone di fila2 sul file `name.conf.default-zones`.
