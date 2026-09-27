# Wi-Fi USB RTL8188GU / RTL8710BU su Debian 13

---
🇬🇧 **Available in English:** [Read the README in English](README.md)
---

Wi-Fi USB RTL8188GU / RTL8710BU su Debian 13

La configurazione è stata testata su Debian 13 con il kernel della serie Debian 6.12.

I driver Linux funzionano per l'adattatore Wi-Fi USB Realtek RTL8188GU / RTL8710BU con ID USB:

Sul sistema testato, l'adattatore è stato inizializzato correttamente dallo stack di driver Wi-Fi USB del kernel Linux. Il dispositivo può essere gestito dal driver rtl8xxxu integrato nel kernel o dal modulo 8188gu compilato, a seconda della configurazione del kernel e della priorità del driver.

Questo repository documenta una configurazione funzionante testata su Debian 13 con il kernel della serie Debian 6.12.

Importante: questo non è un driver Realtek originale. Il progetto si basa su codice sorgente derivato da altri driver Realtek. Le note originali sul copyright e sulla licenza contenute nei file sorgente sono state mantenute.

Famiglia di chip hardware: Realtek RTL8710B / RTL8188GU ID fornitore USB: 0BDA ID prodotto USB: B711 Descrizione USB: Adattatore WLAN 802.11n Bus: USB 2.0 Wi-Fi: 802.11n, 2,4 GHz Architettura: 1T1R

L'adattatore potrebbe inizialmente essere elencato come:

0BDA:1A2B

e poi passare a:

0BDA:B711

tramite interruttore modalità USB.

Sistema testato

La configurazione qui documentata è stata testata su:

Sistema operativo: Debian GNU/Linux 13.7 (trixie) Kernel: 6.12.107+deb13-amd64 GCC: 14.2.0 Architettura: x86_64 Desktop: KDE Plasma

I file di intestazione del kernel utilizzati per la compilazione corrispondevano a quelli del kernel in esecuzione.

Risultato

Il codice sorgente del driver è stato adattato e compilato con successo per il kernel Debian 13 6.12.

Il modulo kernel risultante è:

8188gu.ko

L'adattatore può essere rilevato come un dispositivo RTL8710B/RTL8188GU e può creare un'interfaccia di rete wireless.

L'interfaccia wireless è stata creata con successo durante la fase di test.

A seconda della configurazione del kernel e della priorità del driver, il dispositivo può essere gestito dal driver rtl8xxxu integrato nel kernel o dal modulo 8188gu compilato.

Durante i test, è stata verificata una connessione wireless funzionante tramite NetworkManager. I log del kernel disponibili non indicano che questa connessione fosse gestita esclusivamente dal modulo personalizzato 8188gu.

Durante i test, è stata verificata una connessione wireless funzionante tramite NetworkManager. I log del kernel disponibili non indicano che questa connessione fosse gestita esclusivamente dal modulo personalizzato 8188gu.

Esempio di interfaccia:

wlxXXXXXXXXXXXX

<img width="505" height="47" alt="nmcli device" src="https://github.com/user-attachments/assets/b7d25f36-914c-4b66-a417-7ed3cdfa4f91" />



Il nome effettivo dell'interfaccia dipende dall'indirizzo MAC dell'adattatore USB.

Comportamento dei LED

L'adattatore USB RTL8188GU può funzionare correttamente anche senza l'attività del LED.

Pertanto, l'attività del LED non deve essere utilizzata come unico indicatore del corretto funzionamento dell'adattatore Wi-Fi.

L'analisi dei fattori determinanti ha rivelato quanto segue:

CONFIG_RTW_SW_LED è abilitato nella configurazione del driver.

Il framework LED è stato inizializzato con successo.

Il codice del driver chiama le funzioni SwLedOn_8710BU() e SwLedOff_8710BU(), ma nell'attuale implementazione USB dell'RTL8710B queste funzioni aggiornano solo lo stato interno del LED (bLedOn) e non eseguono scritture hardware dirette nei registri/GPIO del LED.

Perciò:

un LED a stato solido mancante;

un LED lampeggiante mancante durante il traffico;

Nessuna attività del LED dopo la connessione.

Questo non deve essere considerato una prova di un guasto al driver o all'hardware.

L'adattatore può essere completamente rilevato, gestito dal driver, connesso a una rete WiFi e funzionare normalmente senza alcuna indicazione LED visibile.

Utilizzare i comandi lsusb, iw dev, ip link e NetworkManager per verificarne il funzionamento.

Progetto originale

Il compilatore chiama le funzioni SwLedOn_8710BU() e SwLedOff_8710BU(), ma non le scrive.

Tuttavia, nell'attuale implementazione USB dell'RTL8710B, queste funzioni aggiornano solo lo stato interno del LED (bLedOn) e non eseguono scritture hardware nei registri/GPIO del LED.

Perciò:

un LED a stato solido mancante;

un LED lampeggiante mancante durante il traffico;

Nessuna attività del LED dopo la connessione.

Questo non deve essere considerato una prova di un guasto al driver o all'hardware.

L'adattatore può essere completamente rilevato, gestito dal driver, connesso a una rete WiFi e funzionare normalmente senza alcuna indicazione LED visibile.

Utilizzare i comandi lsusb, iw dev, ip link e NetworkManager per verificarne il funzionamento.

Progetto originale

Il punto di partenza di questo lavoro è:

McMCCRU/rtl8188gu

Il progetto originale identifica il dispositivo come:

RTL8188GU (RTL8710B) — VID:PID 0x0BDA:0xB711

Il repository originale non conteneva un file LICENSE separato. I suoi file sorgente contengono le note di copyright originali e la licenza GPLv2, ove applicabile.

Questo repository conserva quindi i file sorgente originali e le relative intestazioni di copyright/licenza.

Correzioni per Debian 13 / Kernel 6.12

Il codice sorgente originale necessitava di modifiche per essere compilato correttamente con gli header del kernel Debian 13 6.12.

I seguenti file sono stati modificati.

os_dep/linux/ioctl_cfg80211.c

cfg80211_rtw_change_beacon

Il parametro della funzione è stato aggiornato da:

struct cfg80211_beacon_data *info

A:

struct cfg80211_ap_update *info

Di conseguenza, i riferimenti ai dati del beacon sono stati aggiornati da:

info->testa info->lunghezza_testa info->coda info->lunghezza_coda

A:

info->beacon.head info->beacon.head_len info->beacon.tail info->beacon.tail_len

cfg80211_rtw_set_monitor_channel
L'API del kernel corrente richiede il parametro del dispositivo di rete:

struct net_device *ndev

Prima:

struct cfg80211_chan_def *chandef

La dichiarazione della funzione è stata aggiornata di conseguenza.

os_dep/linux/usb_intf.c

La funzione di callback per la chiusura del driver USB è stata aggiornata da:

.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,

A:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

Queste modifiche sono incluse nel commit:

220e7561cb0690ffc2352cdc0bfd80112ea04e6d

Messaggio di commit:

Risolto il problema di compilazione per il kernel 6.12 di Debian 13.

La funzione di callback per la chiusura del driver USB è stata aggiornata da:

.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,

A:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

Queste modifiche sono incluse nel commit:

220e7561cb0690ffc2352cdc0bfd80112ea04e6d

Messaggio di commit:

Risolto il problema di compilazione per il kernel 6.12 di Debian 13.

Costruzione e installazione
Installa gli strumenti di compilazione e i file di intestazione del kernel necessari.

Per un sistema Debian che utilizza il kernel attualmente in esecuzione:

sudo apt install build-essential linux-headers-$(uname -r)

Clona questo repository e accedi alla directory sorgente:

git clone https://github.com/rosasgianluigi-bot/RTL8188GU-RTL8710BU-Debian13.git cd RTL8188GU-RTL8710BU-Debian13

Compila il driver:

Fare

Installa il modulo compilato:

sudo make install

Dopo l'installazione, aggiornare il database delle dipendenze del modulo:

sudo depmod -a

Quindi ricollega l'adattatore USB o ricarica il driver, a seconda dello stato attuale del sistema.

Verifica del dispositivo USB
Verificare che l'adattatore venga rilevato dal sottosistema USB:

lsusb

L'ID previsto del dispositivo USB è:

0bda:b711

L'adattatore potrebbe inizialmente apparire come:

0bda:1a2b

e poi passare a:

0bda:b711

dopo il passaggio alla modalità USB.

Associazione driver USB
Per verificare quale driver USB è associato al dispositivo:

lsusb -t

Cerca l'adattatore wireless e il relativo driver del kernel.

Interfaccia wireless
Elenca le interfacce wireless disponibili:

sviluppo IW

O:

collegamento IP

Dopo l'inizializzazione avvenuta con successo, dovrebbe apparire un'interfaccia wireless.

Il nome dell'interfaccia può essere generato dall'indirizzo MAC dell'adattatore, ad esempio:

wlxXXXXXXXXXXXX

Il nome effettivo dell'interfaccia dipende dall'indirizzo MAC dell'adattatore USB.

Gestore di rete
Se NetworkManager è installato, verificare lo stato del dispositivo:

stato del dispositivo nmcli

Le reti Wi-Fi disponibili possono essere elencate con:

nmcli device wifi list

Per connettersi a una rete:

nmcli --ask device wifi connect "YOUR_WIFI_NAME" ifname YOUR_WIFI_INTERFACE

Questa --askopzione consente a NetworkManager di richiedere la password Wi-Fi in modo interattivo, senza doverla memorizzare nella riga di comando o in questa documentazione.

Sequenza di verifica di base
Una semplice sequenza di verifica è la seguente da terminale:

lsusb 
lsusb -t 
iw dev
ip link
nmcli device status

Questi controlli verificano l'enumerazione USB, l'associazione del driver del kernel, la creazione dell'interfaccia wireless e lo stato del dispositivo NetworkManager indipendentemente dal LED fisico.

Firmware
La piattaforma RTL8710B utilizza un firmware associato alla famiglia di dispositivi RTL8710B / RTL8188GU.

Il rtl8xxxudriver Linux utilizza file firmware come:

rtlwifi/rtl8710bufw_SMIC.bin rtlwifi/rtl8710bufw_UMC.bin

Durante i test su Debian 13, il file del firmware caricato rtl8xxxuera:

rtlwifi/rtl8710bufw_SMIC.bin

Il firmware è stato caricato correttamente dal kernel durante l'inizializzazione dell'adattatore.

Questo repository contiene anche i dati del firmware RTL8710B incorporati nel codice sorgente del driver.

Importante: il firmware è separato dal codice sorgente del driver Linux e deve essere trattato separatamente ai fini della licenza e della ridistribuzione.

Licenza del firmware

La licenza del firmware è separata dalla licenza del codice sorgente del driver.

Questo repository non garantisce in modo assoluto che i file binari del firmware siano rilasciati sotto licenza GPL. Si consiglia agli utenti di verificare i termini di licenza applicabili forniti dal produttore/fornitore prima di ridistribuire il firmware separatamente.

Origine e provenienza del firmware

Il codice sorgente del driver contiene avvisi di copyright di Realtek e riferimenti alla licenza GPLv2 nei singoli file sorgente.

Il progetto dovrebbe pertanto essere considerato:

Un driver gestito dalla comunità basato su codice sorgente derivato da Realtek; non è una distribuzione ufficiale di driver Realtek; modificato per la compilazione con le API del kernel Debian 13 / Linux 6.12; con il copyright del codice sorgente originale e le note di licenza preservati.

Ai fini della concessione di licenze e della ridistribuzione, il firmware deve essere trattato separatamente dal codice sorgente del driver.

Note sul cambio di modalità USB
Alcuni adattatori basati su questo hardware potrebbero inizialmente presentarsi come dispositivi di archiviazione USB o lettori CD-ROM.

L'identificazione USB iniziale potrebbe essere:

0BDA:1A2B

Dopo aver attivato la modalità USB, l'adattatore wireless potrebbe apparire come:

0BDA:B711

Questo cambio di modalità può essere gestito da usb-modeswitch, a seconda del dispositivo e della configurazione del sistema.

Per verificare l'identificazione USB corrente:

lsusb

Se l'adattatore viene inizialmente elencato come 0BDA:1A2B, attendi qualche istante e ricontrolla dopo il processo di cambio modalità:

lsusb

L'identificazione prevista del dispositivo wireless è:

0BDA:B711

Se il dispositivo wireless non viene visualizzato, prima di apportare modifiche ai driver, controlla l'enumerazione USB e i messaggi del kernel.

Per esempio:

lsusb

E:

dmesg | tail -n 50

Un LED spento non indica necessariamente che l'adattatore sia guasto.

Verificare sempre l'effettiva enumerazione delle porte USB, l'associazione del driver del kernel, l'interfaccia wireless e la connettività di rete indipendentemente dal LED fisico.

Prima di creare servizi di ripristino personalizzati, verificare l'autorizzazione USB e gli strumenti di sicurezza USB.

Sui sistemi moderni come Debian 13 (Kernel 6.12+), all'avvio potrebbe verificarsi un problema di sincronizzazione: la chiavetta USB viene rilevata correttamente da lsusb (ID 0bda:b711), ma l'interfaccia di rete Wi-Fi (wlx...) non compare in ip link a meno che non si scolleghi e ricolleghi fisicamente il dispositivo.

L'autorizzazione USB può essere influenzata da USBGuard, dai parametri del kernel, dalle regole udev o dalle politiche di sicurezza del sistema.

Controllo:

systemctl status usbguard

Se USBGuard non è necessario, è possibile disattivarlo:

sudo systemctl disable --now usbguard

Verificare:

systemctl is-enabled usbguard systemctl is-active usbguard

verifica dell'autorizzazione USB
Se lsusb rileva l'adattatore ma non viene visualizzata alcuna interfaccia wireless:

Controllo:

cat /sys/bus/usb/devices//authorized

Se il risultato è:

0

Il dispositivo USB viene rilevato ma non autorizzato.

Abilitalo con:

echo 1 | sudo tee /sys/bus/usb/devices//authorized

Dopo l'autorizzazione, dovrebbe apparire l'interfaccia wireless.

Esempio:

cat /sys/bus/usb/devices//authorized

resi:

0

Dopo l'autorizzazione:

echo 1 | sudo tee /sys/bus/usb/devices//authorized

L'interfaccia wireless appare.

Se il problema persiste dopo aver verificato le impostazioni di autorizzazione USB, è possibile utilizzare un servizio di ripristino systemd per reinizializzare automaticamente l'adattatore all'avvio. La causa esatta potrebbe dipendere dalla temporizzazione del controller USB, dall'inizializzazione del firmware del dispositivo o dalle politiche di autorizzazione USB del sistema.

Risoluzione dei problemi: dopo il riavvio, la scheda USB viene rilevata ma l'interfaccia wireless non è presente.

Sul sistema Debian 13 testato, l'adattatore USB è stato rilevato correttamente dopo l'avvio con:

lsusb

Mostrando:

0bda:b711

mentre l'interfaccia wireless non era inizialmente presente in:

collegamento IP

Il problema era legato all'inizializzazione del dispositivo USB durante il processo di avvio.

Verifica l'autorizzazione USB

Se l'adattatore compare nell'output di lsusb ma non viene creata alcuna interfaccia wireless, verificare se il dispositivo USB è autorizzato:

cat /sys/bus/usb/devices/2-4/authorized

Il percorso del dispositivo potrebbe essere diverso su un altro sistema. Utilizzare il percorso effettivo del dispositivo USB visualizzato dal sistema.

Se il risultato è:

0

Il dispositivo USB è stato rilevato ma non è autorizzato.

Per autorizzare manualmente il dispositivo:

echo 1 | sudo tee /sys/bus/usb/devices/2-4/authorized

Dopo l'autorizzazione, verificare:

collegamento IP

E:

sviluppo IW

Sul sistema testato, l'autorizzazione del dispositivo USB ha provocato la comparsa dell'interfaccia wireless.

USBGuard

Nel corso dell'indagine è stato verificato anche USBGuard.

Per verificarne lo stato:

systemctl is-enabled usbguard

E:

systemctl is-active usbguard

Se USBGuard non è intenzionalmente utilizzato sul sistema, non si deve presumere che sia la causa del problema. Verificare il suo stato prima di apportare modifiche alla configurazione.

regola di autorizzazione USB udev

È stata testata una regola udev per autorizzare automaticamente i dispositivi USB:

/etc/udev/rules.d/01-usb-allow-all.rules

con:

SOTTOSISTEMA=="usb", AZIONE=="aggiungi", AMBIENTE{TIPO DISPOSITIVO}=="dispositivo_usb", ATTR{autorizzato}="1"

La regola può essere ispezionata con:

cat /etc/udev/rules.d/01-usb-allow-all.rules

La regola è stata verificata con udevadm, ma si noti che udevadm test opera in modalità di test e non scrive direttamente il valore risultante nell'attributo autorizzato del dispositivo.

Dopo aver modificato le regole udev, ricaricale:

sudo udevadm control --reload-rules

E:

trigger sudo udevadm

Quindi ricollega l'adattatore o riavvia il computer e verifica dal terminale:

lsusb
ip link
iw dev 

Importante il percorso esatto del dispositivo USB, ad esempio:

2-4

dipende dal sistema e non si deve presumere che sia identico su un altro computer.

La sequenza diagnostica dovrebbe quindi essere la seguente:

lsusb
ip link
iw dev
ls /sys/bus/usb/devices/

e quindi verificare l'attributo autorizzato del dispositivo USB corrispondente.

Non è richiesto alcun servizio di reset systemd personalizzato come parte della configurazione documentata.

Il sistema testato è stato in grado di inizializzare correttamente l'adattatore dopo l'avvio senza richiedere una disconnessione/riconnessione fisica.

Questo repository documenta nello specifico le modifiche necessarie per il kernel Debian 13 testato:

6.12.107+deb13-amd64

Altre versioni del kernel potrebbero richiedere ulteriori modifiche.

Risoluzione dei problemi: Chiavetta USB non rilevata al riavvio (condizione di gara all'avvio)

Sui sistemi moderni come Debian 13 (Kernel 6.12+), potrebbe verificarsi un problema di sincronizzazione all'avvio: la chiavetta USB viene rilevata correttamente da lsusb (ID 0bda:b711), ma l'interfaccia di rete Wi-Fi (wlx...) non compare in ip link a meno che non si scolleghi e ricolleghi fisicamente il dispositivo.

Ciò accade perché il modulo viene caricato dal kernel prima che il firmware della chiavetta USB abbia completato la transizione elettronica successiva al cambio di modalità.

Per risolvere automaticamente e in modo permanente questo problema su qualsiasi porta USB del PC, segui questi passaggi per creare un servizio Systemd dedicato che esegua un riavvio software del dispositivo all'avvio.

Rimuovere il modulo dal caricamento anticipato
Assicurati che il modulo non sia presente nel file /etc/modules. Apri il file:

sudo nano /etc/modules
Se vedi la riga 8188gu, cancellala, salva ( CTRL+O, Enter), ed esci ( CTRL+X).

Creare il servizio di riavvio automatico
Crea un nuovo file di servizio in Systemd:

sudo nano /etc/systemd/system/rtl8188gu-restart.service

<img width="1168" height="315" alt="riavvio" src="https://github.com/user-attachments/assets/94b8a9df-e50e-4dc1-8ed7-a067256eb5f6" />

Salva il file ( CTRL+O, Enter) ed esci ( CTRL+X).

Abilitare il servizio:
Informa Systemd della modifica e abilita il servizio in modo che si avvii automaticamente ogni volta che il computer viene acceso:

dal terminale:

sudo systemctl daemon-reload

sudo systemctl enable rtl8188gu-restart.service

Riepilogo della verifica

La configurazione documentata è stata verificata attraverso i seguenti passaggi:

Enumerazione del dispositivo USB 
Commutazione della modalità USB su 0BDA:B711
Inizializzazione del firmware RTL8710B
Creazione dell'interfaccia wireless
Rilevamento della rete wireless
La connessione a NetworkManager è stata testata con successo. L'adattatore è stato utilizzato con successo per il Wi-Fi su Debian 13; i log documentati non indicano l'uso esclusivo del modulo personalizzato 8188gu per questa connessione.

Backup

È stato creato un backup locale completo dell'ambiente di lavoro separatamente da questo repository Git.

Il backup contiene:

Codice sorgente del driver; cronologia Git; file del firmware utilizzati durante i test; modulo del kernel installato; modulo del kernel ricompilato; informazioni di sistema; checksum; documentazione.

Il backup viene intenzionalmente mantenuto separato dal repository Git pubblico. Compatibilità del kernel

Questo repository documenta nello specifico le modifiche necessarie per il kernel Debian 13 testato:

6.12.107+deb13-amd64

Altre versioni del kernel potrebbero richiedere ulteriori modifiche.

I log di test disponibili non stabiliscono che la connessione di rete sia stata gestita esclusivamente dal modulo personalizzato 8188gu. A seconda della configurazione del kernel e della priorità del driver, il dispositivo potrebbe invece essere gestito dal driver rtl8xxxu integrato nel kernel.

Backup

È stato creato un backup locale completo dell'ambiente di lavoro separatamente da questo repository Git.

Il backup contiene, ove applicabile:

Codice sorgente del driver; cronologia Git; file del firmware utilizzati durante i test; modulo del kernel installato; modulo del kernel ricompilato; informazioni di sistema; checksum; documentazione.

Il backup viene volutamente mantenuto separato dal repository Git pubblico.

Disclaimer

Questo repository è fornito come documentazione tecnica di una configurazione funzionante.

Le revisioni hardware, le revisioni del firmware, le versioni del kernel, i controller USB e le configurazioni di distribuzione possono variare da sistema a sistema.

Non vi è alcuna garanzia che il driver funzioni senza modifiche su tutti i dispositivi RTL8188GU / RTL8710BU.

Quando si testa un driver wireless o si modifica la configurazione di rete, assicurarsi sempre, se possibile, di avere a disposizione una connessione di rete alternativa funzionante.

Crediti e ringraziamenti

Questo progetto si basa sul lavoro contenuto nel progetto originale rtl8188gu di McMCCRU ed è stato adattato e testato per i moderni kernel Linux.

Un ringraziamento speciale a:

@McMCCRU

per il repository originale rtl8188gu, che ha fornito il codice sorgente e il supporto iniziale per le versioni precedenti di Linux.

Ulteriori attività di compatibilità del kernel e test su Debian 13:

Gianluigi Rosas

Piattaforma di test 
Sistema operativo: Debian GNU/Linux 13.7 
Kernel: 6.12.107+deb13-amd64 
Desktop: KDE Plasma 
Adattatore: UNICO WA2763 Chip: Realtek RTL8188GU-RTL8710BU 
ID USB: 0BDA:B711 Wi-Fi: 802.11n, 2,4 GHz

Chip UNICO WA2763 Realtek Semiconductor Corp. Adattatore WLAN RTL8188GU 802.11n


<img width="300" height="638" alt="Unicowa2763" src="https://github.com/user-attachments/assets/cbe73a20-87c3-43ba-82c0-2c668a5c70e1" />


