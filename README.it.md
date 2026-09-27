Wi-Fi USB RTL8188GU / RTL8710BU su Debian 13

---
🇬🇧 **Available in English:** [Read the README in English](README.md)
---

Questo repository documenta una configurazione funzionante di un adattatore Wi-Fi USB Realtek RTL8188GU / RTL8710BU su Debian 13 con kernel Linux serie 6.12.

Il dispositivo USB testato ha il seguente identificativo:

VID:PID = 0BDA:B711

La configurazione qui descritta è stata sviluppata e testata su Debian GNU/Linux 13 con KDE Plasma.

Importante: questo repository documenta una configurazione testata. Non si tratta di una distribuzione ufficiale di driver Realtek.

Hardware

L'adattatore testato è identificato come:

Realtek RTL8188GU / RTL8710B
VID:PID = 0BDA:B711

Descrizione USB tipica:

Adattatore WLAN 802.11n RTL8188GU

Caratteristiche principali:

Famiglia Realtek RTL8188GU / RTL8710B
USB 2.0
802.11n
2,4 GHz
1T1R
ID fornitore USB: 0BDA
ID prodotto USB: B711

Alcuni adattatori potrebbero inizialmente mostrare un ID USB diverso prima del cambio di modalità USB:

0BDA:1A2B

e successivamente passare a:

0BDA:B711

Questo comportamento potrebbe essere correlato al comando usb-modeswitch.

Sistema testato

La configurazione qui documentata è stata testata su:

Sistema operativo: Debian GNU/Linux 13 (trixie)
Kernel: 6.12.107+deb13-amd64
Architettura: x86_64
Desktop: KDE Plasma
GCC: 14.2.0

Gli header del kernel utilizzati per la compilazione corrispondevano al kernel in esecuzione.

Il modulo driver risultante è:

8188gu.ko

Progetto driver originale

Il punto di partenza di questo lavoro è:

https://github.com/McMCCRU/rtl8188gu

Il progetto originale identifica l'adattatore come:

RTL8188GU (RTL8710B)    VID:PID = 0x0BDA:0xB711

Il progetto originale non è una distribuzione ufficiale del driver Realtek. Si basa su codice sorgente derivato dai sorgenti del driver Realtek.

I file sorgente originali e le relative note di copyright/licenza sono conservati in questo repository.

Correzioni di compatibilità con Debian 13 / Kernel 6.12

Il codice sorgente originale necessitava di modifiche per la compilazione con gli header del kernel Linux 6.12 di Debian 13.

os_dep/linux/ioctl_cfg80211.c

cfg80211_rtw_change_beacon

Il parametro della funzione è stato aggiornato da:

struct cfg80211_beacon_data *info

a:

struct cfg80211_ap_update *info

I riferimenti ai dati del beacon corrispondenti sono stati aggiornati da:

info->head
info->head_length
info->tail
info->tail_length

a:

info->beacon.head
info->beacon.head_len
info->beacon.tail
info->beacon.tail_len
cfg80211_rtw_set_monitor_channel

L'API del kernel corrente richiede il parametro del dispositivo di rete:

struct net_device *ndev

La dichiarazione della funzione è stata aggiornata di conseguenza.

os_dep/linux/usb_intf.c

La funzione di callback per la chiusura del driver USB è stata modificata da:

.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,

a:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

Questo si è reso necessario perché il membro drvwrap non è più disponibile nell'API del kernel utilizzata da Debian 13 / Linux 6.12.

Compilazione e installazione

Installa gli strumenti di compilazione e gli header del kernel necessari,quindi da terminale diamo il comando:

sudo apt install build-essential linux-headers-$(uname -r)

Cloniamo il repository,quindi sempre da terminale diamo il comando:

git clone https://github.com/rosasgianluigi-bot/RTL8188GU-RTL8710BU-Debian13.git

Accedi alla directory sorgente,sempre da terminale diamo:

cd RTL8188GU-RTL8710BU-Debian13

Compiliamo il file Makefile con il comando da terminale:

make

(non dovrebbe restituire errori,se ci fossero,potrebbero venire da una versione del vostro kernel diversa dalla 6.12.107)

Sempre da terminale diamo il comando per l'installazione:

sudo make install

Aggiorna il database delle dipendenze del modulo,sempre da terminale diamo il comando:

sudo depmod -a

Dopo l'installazione, ricollega l'adattatore USB o ricarica il driver in base allo stato attuale del sistema.

Verifica del dispositivo USB

Verifica se l'adattatore USB viene rilevato,da terminale:

lsusb

Il dispositivo previsto è:

0bda:b711

Se l'adattatore appare inizialmente come:

0bda:1a2b

attendi qualche istante e ricontrolla:

lsusb

L'identificazione finale prevista del dispositivo wireless è:

0bda:b711

Se l'adattatore non viene visualizzato, controlla l'enumerazione USB e i messaggi del kernel prima di modificare la configurazione del driver.

Comandi utili:

lsusb

e:

dmesg | tail -n 50

Associazione del driver USB

Per ispezionare il dispositivo USB e il relativo driver:

lsusb -t

L'adattatore wireless dovrebbe apparire insieme al driver del kernel associato.

Il sistema potrebbe contenere entrambi i seguenti driver:

rtl8xxxu

e:

8188gu

come possibili driver per questo dispositivo.

La selezione del driver dipende dalla configurazione del kernel, dagli alias e dalla priorità del driver.

Pertanto, la sola presenza del modulo 8188gu non dimostra automaticamente che sia il driver attualmente in uso per la gestione dell'adattatore.

Interfaccia wireless

Elenca le interfacce wireless:

iw dev

oppure:

ip link

Dopo l'inizializzazione, dovrebbe essere presente un'interfaccia wireless.

Il nome dell'interfaccia potrebbe essere simile a:

wlxXXXXXXXXXXXX

Il nome effettivo dipende dall'indirizzo MAC dell'adattatore USB.

Per l'adattatore testato, un'interfaccia di esempio era:

wlx90de806b12e1
Verifica di base del driver

Una sequenza diagnostica utile è:

lsusb
lsusb -t
iw dev
ip link
nmcli device status

Questi comandi verificano, in modo indipendente:

Enumerazione USB
Associazione del driver USB
Creazione dell'interfaccia wireless
Stato dell'interfaccia di rete
Stato del dispositivo NetworkManager

La connettività di rete deve essere testata indipendentemente dal LED fisico.

(SEZIONE MOLTO IMPORTANTE) Comportamento del LED

L'adattatore RTL8188GU / RTL8710BU può funzionare correttamente anche quando il suo LED fisico non è acceso.

Pertanto:

L'assenza di attività del LED non deve essere considerata, di per sé, una prova di un guasto del driver o dell'hardware.

Durante l'analisi del codice sorgente del driver, è stato verificato il seguente comportamento:

CONFIG_RTW_SW_LED è abilitato nella configurazione del driver;
il framework del LED è inizializzato;
il driver chiama SwLedOn_8710BU();
il driver chiama SwLedOff_8710BU();
nell'attuale implementazione USB dell'RTL8710B, queste funzioni aggiornano lo stato interno del LED ma non eseguono necessariamente un'operazione diretta sul LED/GPIO hardware.

Di conseguenza, l'adattatore può funzionare normalmente anche in assenza di:

LED permanentemente acceso;
LED lampeggiante;
LED non visibile durante il traffico di rete.

L'adattatore può quindi essere:

rilevato correttamente dalla porta USB;
associato a un driver del kernel;
rappresentato da un'interfaccia wireless;
rilevato da NetworkManager;
connesso a una rete Wi-Fi;
utilizzato normalmente per il traffico di rete;

anche quando il LED fisico rimane spento.

Verificare sempre il funzionamento effettivo utilizzando:

lsusb
iw dev
ip link
nmcli device status

e, se necessario, una connessione di rete effettiva o una scansione Wi-Fi.

Firmware

La piattaforma RTL8710B / RTL8188GU utilizza un firmware associato alla famiglia di dispositivi RTL8710B.

Il driver Linux rtl8xxxu utilizza i seguenti file firmware:

/lib/firmware/rtlwifi/rtl8710bufw_SMIC.bin
/lib/firmware/rtlwifi/rtl8710bufw_UMC.bin

Durante i test su Debian 13, il firmware caricato da rtl8xxxu era:

rtlwifi/rtl8710bufw_SMIC.bin

Il firmware è stato caricato correttamente durante l'inizializzazione dell'adattatore.

Questo repository potrebbe contenere anche dati del firmware RTL8710B incorporati nel codice sorgente del driver.

Licenza del firmware

La licenza del firmware è separata dalla licenza del codice sorgente del driver Linux.

Questo repository non dichiara che i file binari del firmware siano rilasciati sotto licenza GPL.

Pertanto, la ridistribuzione del firmware deve essere considerata separatamente dalla ridistribuzione del codice sorgente del driver.

Gli utenti sono tenuti a verificare la licenza applicabile e i termini di ridistribuzione prima di ridistribuire i file binari del firmware.

Commutazione della modalità USB

Alcune versioni di questo hardware potrebbero inizialmente presentarsi come un dispositivo di archiviazione USB/CD-ROM.

L'identificazione iniziale potrebbe essere:

0BDA:1A2B

Dopo il cambio di modalità USB, l'adattatore potrebbe apparire come:

0BDA:B711

A seconda del dispositivo e della configurazione di sistema, questo può essere gestito tramite:

usb-modeswitch

Verifica l'identificazione USB corrente con:

lsusb

Se l'adattatore appare inizialmente come 0BDA:1A2B, attendi qualche istante e ricontrolla:

lsusb

Se il dispositivo wireless non viene ancora visualizzato, controlla l'enumerazione USB e i messaggi del kernel prima di apportare modifiche ai driver.

NetworkManager

Se NetworkManager è installato, verifica lo stato del dispositivo con:

nmcli device status

Elenca le reti Wi-Fi disponibili:

nmcli device wifi list

È possibile creare una connessione in modo interattivo con:

nmcli --ask device wifi connect "NOME_WIFI" ifname INTERFACCIA_WIFI

L'opzione --ask consente a NetworkManager di richiedere la password Wi-Fi in modo interattivo anziché inserirla direttamente nella riga di comando.

NetworkManager, iwd e wpa-supplicant
Nota importante sulla configurazione di Debian 13

Durante i test di questo adattatore sul sistema Debian 13 documentato, la gestione del Wi-Fi è stata analizzata in relazione a:

NetworkManager
iwd
wpa-supplicant

La configurazione funzionante utilizza iwd anziché wpa_supplicant come supplicant Wi-Fi.

Sul sistema testato:

iwd

è il supplicant Wi-Fi attivo, mentre:

wpa_supplicant

è mascherato.

Questa distinzione è importante.

Il driver USB stesso non richiede l'utilizzo diretto di wpa_supplicant. Il driver fornisce l'interfaccia wireless allo stack di rete di Linux; NetworkManager e il backend Wi-Fi configurato gestiscono la connessione di rete.

Pertanto:

Non sbloccare o abilitare wpa_supplicant solo perché è presente l'adattatore RTL8188GU.

Modificare il supplicant Wi-Fi senza verificare la configurazione esistente di NetworkManager può introdurre un secondo percorso di gestione della rete in competizione.

Verificare il backend Wi-Fi

Controllare lo stato di iwd:

systemctl status iwd --no-pager

Controllare lo stato di wpa_supplicant:

systemctl status wpa_supplicant --no-pager

Nella configurazione testata, iwd è il componente attivo e wpa_supplicant non viene utilizzato come supplicant attivo.

Per verificare se wpa_supplicant è mascherato:

systemctl is-enabled wpa_supplicant

Un risultato come:

masked

significa che l'avvio del servizio è stato intenzionalmente impedito.

Questa impostazione non dovrebbe essere modificata a meno che non vi sia un motivo specifico per cambiare la configurazione di gestione del Wi-Fi.

Inizializzazione Wi-Fi dopo il riavvio

Durante i test è stato riscontrato uno specifico problema di inizializzazione.

Dopo il riavvio, l'adattatore USB è stato correttamente rilevato da:

lsusb

e ha mostrato:

0bda:b711

mentre l'interfaccia wireless risultava temporaneamente assente da:

ip link

Il problema era legato all'inizializzazione del dispositivo wireless USB durante il processo di avvio.

È importante sottolineare che l'enumerazione USB e la creazione dell'interfaccia wireless sono fasi separate.

Ad esempio:

Dispositivo USB rilevato

0BDA:B711 visibile in lsusb

Inizializzazione del driver del kernel

Interfaccia wireless creata

NetworkManager / iwd

Pertanto, la visualizzazione dell'adattatore in lsusb non dimostra di per sé che l'interfaccia wireless sia già pronta per l'uso.

Reinizializzazione di iwd

Durante l'indagine, si è scoperto che riavviare iwd è un metodo efficace per reinizializzare il percorso di gestione Wi-Fi quando l'adattatore viene rilevato tramite USB ma l'interfaccia wireless non è immediatamente disponibile per NetworkManager.

Verificare prima lo stato attuale:

systemctl status iwd --no-pager

Se l'interfaccia è presente ma la gestione Wi-Fi non è ancora disponibile, la procedura di ripristino testata è stata:

sudo systemctl restart iwd

Quindi verificare:

iw dev
nmcli device status

e, se necessario:

nmcli device wifi list


Importante,il riavvio di iwd deve essere considerato un passaggio di ripristino/diagnostica per la configurazione testata.

Non è un requisito del driver RTL8188GU.

Il driver e il supplicant Wi-Fi sono componenti separati.

wpa_supplicant

wpa_supplicant è un componente separato per l'autenticazione/supplicant Wi-Fi.

Non è necessario eseguire entrambi:

iwd

e:

wpa_supplicant

come supplicant concorrenti per la stessa interfaccia Wi-Fi gestita da NetworkManager.

Sul sistema Debian 13 testato, wpa_supplicant era mascherato e al suo posto veniva utilizzato iwd.

Pertanto, durante la risoluzione dei problemi di questa scheda, è necessario innanzitutto determinare quale supplicant è configurato e attivo, anziché abilitare automaticamente wpa_supplicant.

Verifiche utili:

systemctl is-enabled iwd
systemctl is-active iwd
systemctl is-enabled wpa_supplicant
systemctl is-active wpa_supplicant

La configurazione esatta di NetworkManager dovrebbe essere mantenuta a meno che non vi sia un motivo specifico per modificarla.

Autorizzazione USB

Se l'adattatore viene visualizzato in:

lsusb

ma non viene creata alcuna interfaccia wireless, verificare se il dispositivo USB è autorizzato.

Innanzitutto, identificare il percorso effettivo del dispositivo USB.

Ad esempio:

/sys/bus/usb/devices/2-4/

Il percorso dipende dal sistema e non deve essere considerato identico su un altro computer.

Verificare con:

cat /sys/bus/usb/devices/<dispositivo>/authorized

Se il risultato è:

0

il dispositivo USB è stato rilevato ma non è autorizzato.

Può essere autorizzato manualmente con:

echo 1 | sudo tee /sys/bus/usb/devices/<dispositivo>/authorized

Quindi verificare con:

ip link

e:

iw dev

Nel sistema testato, l'autorizzazione USB è stata parte dell'indagine sul problema di inizializzazione all'avvio.

GESTIONE PACCHETTO Usbguard

Anche USBGuard è stato esaminato durante la risoluzione dei problemi,io personalmente l'ho disinstallato,se voi volete tenerlo ma disattivarlo:

Per verificarne lo stato:

systemctl is-enabled usbguard

e:

systemctl is-active usbguard

Se USBGuard non viene utilizzato intenzionalmente, non si deve automaticamente presumere che sia la causa di un problema di inizializzazione dell'adattatore.

È necessario verificarne lo stato prima di modificare qualsiasi configurazione.

Regola di autorizzazione USB di udev

Durante l'indagine è stata testata la seguente regola udev:

/etc/udev/rules.d/01-usb-allow-all.rules

con:

SUBSYSTEM=="usb", ACTION=="add", ENV{DEVTYPE}=="usb_device", ATTR{authorized}="1"

Esaminare la regola con:

cat /etc/udev/rules.d/01-usb-allow-all.rules

Dopo aver modificato le regole udev, ricaricarle con:

sudo udevadm control --reload-rules

e:

sudo udevadm trigger

Quindi ricollegare l'adattatore o riavviare il sistema e verificare:

lsusb
ip link
iw dev

udevadm test opera in modalità di test e non scrive automaticamente il valore risultante nell'attributo authorized del dispositivo.

Sequenza diagnostica

Se l'adattatore non funziona dopo l'avvio, non reinstallare immediatamente il driver.

Utilizzare la seguente sequenza:

lsusb
lsusb -t
ip link
iw dev
nmcli device status

Quindi, controlla i percorsi dei dispositivi USB:

ls /sys/bus/usb/devices/

Se l'adattatore è visibile in lsusb ma l'interfaccia wireless non è presente, verifica l'autorizzazione USB e l'inizializzazione del driver prima di modificare NetworkManager o il driver.

Importante: Rilevamento USB vs. Disponibilità Wi-Fi

I seguenti stati non sono equivalenti:

Dispositivo USB rilevato
Driver del kernel caricato
Interfaccia wireless creata
NetworkManager segnala Wi-Fi disponibile
Connesso a una rete Wi-Fi

Un errore in una fase non indica necessariamente un errore nella fase precedente.

Ad esempio:

lsusb

potrebbe mostrare:

0bda:b711

mentre:

ip link

non mostra ancora l'interfaccia wireless.

Questa situazione dovrebbe essere diagnosticata come un problema di inizializzazione piuttosto che essere immediatamente considerata un adattatore difettoso.

Compatibilità del kernel

Questo repository documenta specificamente le modifiche necessarie per:

Debian 13
Kernel Linux 6.12.107+deb13-amd64

Altre versioni del kernel potrebbero richiedere ulteriori modifiche.

Il driver originale non è stato scritto specificamente per l'API del kernel Linux 6.12 di Debian 13.

Riepilogo della verifica

La configurazione documentata è stata verificata attraverso le seguenti fasi:

Enumerazione USB dell'adattatore.
Commutazione della modalità USB su 0BDA:B711.
Inizializzazione del firmware RTL8710B.
Inizializzazione del driver del kernel.
Creazione dell'interfaccia wireless.
Rilevamento delle reti Wi-Fi disponibili.
Rilevamento dell'interfaccia Wi-Fi da parte di NetworkManager.
Funzionamento Wi-Fi corretto su Debian 13.
Ripristino del percorso di gestione Wi-Fi tramite iwd quando necessario.
Inizializzazione corretta dopo il riavvio senza necessità di scollegare e ricollegare fisicamente il dispositivo.

I risultati dei test disponibili non stabiliscono che ogni connessione di rete riuscita sia necessariamente gestita esclusivamente dal modulo personalizzato 8188gu.

A seconda della configurazione del kernel e della priorità del driver, l'adattatore potrebbe invece essere gestito dal driver integrato nel kernel:

rtl8xxxu

driver.

Backup

È stato creato un backup locale completo dell'ambiente di lavoro separatamente da questo repository Git pubblico.

Il backup può contenere:

codice sorgente del driver;
cronologia Git;
file del firmware utilizzati durante i test;
modulo del kernel installato;
modulo del kernel ricompilato;
informazioni di sistema;
checksum;
file di configurazione;
documentazione.

Il backup viene intenzionalmente mantenuto separato dal repository pubblico.

Filosofia di risoluzione dei problemi

Quando si risolve un problema con questo adattatore:

Iniziare con controlli non distruttivi.
Verificare l'enumerazione USB prima di modificare il driver.
Verificare l'associazione del driver prima di reinstallare qualsiasi cosa.
Verificare l'interfaccia wireless prima di modificare NetworkManager.

Verificare il supplicant Wi-Fi prima di modificare iwd o wpa_supplicant.

Non utilizzare il LED fisico come unico indicatore diagnostico.

Evitare di sostituire più componenti contemporaneamente.

Se possibile, mantenere disponibile una connessione di rete alternativa funzionante.

Utili controlli di primo livello sono:

lsusb
lsusb -t
iw dev
ip link
nmcli device status
Avvertenza

Questo repository è fornito come documentazione tecnica di una configurazione funzionante.

Le revisioni hardware, le revisioni del firmware, le versioni del kernel, i controller USB, le configurazioni di NetworkManager e le configurazioni di distribuzione possono variare da sistema a sistema.

Non vi è alcuna garanzia che il driver funzioni senza modifiche su tutti i dispositivi RTL8188GU / RTL8710BU.

Ringraziamenti

Questo progetto si basa sul progetto originale rtl8188gu di:

McMCCRU

Repository originale:

https://github.com/McMCCRU/rtl8188gu

Il progetto originale ha fornito il codice sorgente e il supporto iniziale per RTL8188GU / RTL8710B.

Ulteriori attività di compatibilità del kernel, test e documentazione per Debian 13 / Linux 6.12:

Gianluigi Rosas

Piattaforma di test 
Sistema operativo: Debian GNU/Linux 13.7 
Kernel: 6.12.107+deb13-amd64 
Desktop: KDE Plasma 
Adattatore: UNICO WA2763 Chip: Realtek RTL8188GU-RTL8710BU 
ID USB: 0BDA:B711 Wi-Fi: 802.11n, 2,4 GHz

Chip UNICO WA2763 Realtek Semiconductor Corp. Adattatore WLAN RTL8188GU 802.11n


<img width="300" height="638" alt="Unicowa2763" src="https://github.com/user-attachments/assets/cbe73a20-87c3-43ba-82c0-2c668a5c70e1" />


