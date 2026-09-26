Wi-Fi USB RTL8188GU / RTL8710BU su Debian 13

I driver Linux funzionano per l'adattatore Wi-Fi USB Realtek RTL8188GU / RTL8710BU con ID USB:

0BDA:B711

Questo repository documenta una configurazione funzionante testata su Debian 13 con il kernel della serie Debian 6.12.

Importante: questo non è un driver Realtek originale. Il progetto si basa su codice sorgente derivato da altri driver Realtek. Le note originali sul copyright e sulla licenza contenute nei file sorgente sono state mantenute.

Famiglia di chip hardware: Realtek RTL8710B / RTL8188GU ID fornitore USB: 0BDA ID prodotto USB: B711 Descrizione USB: Adattatore WLAN 802.11n Bus: USB 2.0 Wi-Fi: 802.11n, 2,4 GHz Architettura: 1T1R

L'adattatore potrebbe inizialmente essere enumerato come:

0BDA:1A2B

e poi passare a:

0BDA:B711

tramite usb-modeswitch.

Sistema verificato

La configurazione qui documentata è stata testata su:

Sistema operativo: Debian GNU/Linux 13.7 (trixie) Kernel: 6.12.107+deb13-amd64 GCC: 14.2.0 Architettura: x86_64 Desktop: KDE Plasma

I file di intestazione del kernel utilizzati per la compilazione corrispondevano a quelli del kernel in esecuzione.

Risultato

Il codice sorgente del driver è stato adattato e compilato con successo per il kernel 6.12 di Debian 13.

Il modulo kernel risultante è:

8188gu.ko

L'adattatore può essere rilevato come un dispositivo RTL8710B/RTL8188GU e può creare un'interfaccia di rete wireless.

Durante i test è stata verificata una connessione wireless funzionante tramite NetworkManager. I log del kernel disponibili non indicano che questa connessione sia stata gestita esclusivamente dal modulo personalizzato 8188gu.

Esempio di interfaccia:

wlxXXXXXXXXXXXX

<img width="505" height="47" alt="nmcli device" src="https://github.com/user-attachments/assets/b7d25f36-914c-4b66-a417-7ed3cdfa4f91" />










Il nome effettivo dell'interfaccia dipende dall'indirizzo MAC dell'adattatore USB.

Comportamento del LED

L'adattatore USB RTL8188GU può funzionare correttamente anche in assenza di attività LED.

Pertanto l'attività del LED non deve essere utilizzata come unica indicazione del corretto funzionamento dell'adattatore Wi-Fi.

Durante l'analisi dei fattori determinanti è emerso che:

CONFIG_RTW_SW_LED è abilitato nella configurazione del driver.

Il framework LED è stato inizializzato correttamente.

La compilazione chiama SwLedOn_8710BU()e SwLedOff_8710BU(), ma senza scriverla

Tuttavia, nell'attuale implementazione USB dell'RTL8710B, queste funzioni aggiornano solo lo stato interno del LED ( bLedOn) e non eseguono scritture hardware nei registri/GPIO del LED.

Perciò:

un LED a stato solido mancante;

un LED lampeggiante mancante durante il traffico;

nessuna attività del LED dopo la connessione

Non deve essere considerato di per sé una prova di un guasto del driver o dell'hardware.

L'adattatore può essere completamente rilevato, gestito dal driver, connesso a una rete WiFi e funzionare normalmente senza alcuna indicazione LED visibile.

Utilizzare i comandi lsusb, iw dev, ip link e NetworkManager per verificare il funzionamento.

Progetto originale

Il punto di partenza di questo lavoro è:

McMCCRU/rtl8188gu

Il progetto originale identifica il dispositivo come:

RTL8188GU (RTL8710B) — VID:PID 0x0BDA:0xB711

Il repository originale non conteneva un file LICENSE separato. I suoi file sorgente contengono le note originali sul copyright e sulla licenza GPLv2, ove applicabile.

Questo repository conserva quindi i file sorgente originali e le relative intestazioni di copyright/licenza.

Correzioni per Debian 13 / Kernel 6.12

Il codice sorgente originale necessitava di modifiche per essere compilato correttamente con gli header del kernel 6.12 di Debian 13.

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

info->beacon.head info->beacon.head_len info->beacon.tail info->beacon.tail_len 2. cfg80211_rtw_set_monitor_channel

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

Correzione della build per il kernel 6.12 di Debian 13.


Installa gli strumenti di compilazione e i file di intestazione del kernel necessari.

Per un sistema Debian che utilizza il kernel attualmente in esecuzione:

sudo apt install build-essential linux-headers-$(uname -r)

Clona questo repository e accedi alla directory dei sorgenti:

clone git https://github.com/rosasgianluigi-bot/RTL8188GU-Debian13.git cd RTL8188GU-Debian13

Compilare:

make

Installare:

sudo make install

Dopo l'installazione, aggiornare il database delle dipendenze del modulo con il comando:

sudo depmod -a

Quindi ricollega l'adattatore USB o reinstalla il driver, a seconda delle esigenze del sistema.

Verifica del dispositivo USB

Verifica che l'adattatore sia visibile con il comando:

lsusb

L'ID del dispositivo previsto è:

0bda:b711

È inoltre possibile verificare l'associazione del driver USB con:

lsusb -t Controllo dell'interfaccia wireless

Elenca le interfacce wireless:

sviluppo IW

O:

collegamento IP

Una volta che l'adattatore è stato inizializzato correttamente, dovrebbe comparire un'interfaccia wireless.

Il nome dell'interfaccia può essere generato dall'indirizzo MAC dell'adattatore, ad esempio:

wlxXXXXXXXXXXXX NetworkManager

Se NetworkManager è installato, le reti Wi-Fi disponibili possono essere elencate con:

Elenco dispositivi wifi nmcli

Per connettersi:

nmcli --ask device wifi connect "YOUR_WIFI_NAME" ifname YOUR_WIFI_INTERFACE

L'opzione --ask consente a NetworkManager di richiedere la password Wi-Fi senza doverla inserire nella riga di comando o in questa documentazione.

Firmware

La piattaforma RTL8710B utilizza un firmware associato alla famiglia di dispositivi RTL8710B/RTL8188GU.

Il driver Linux rtl8xxxu utilizza file firmware come:

rtlwifi/rtl8710bufw_SMIC.bin rtlwifi/rtl8710bufw_UMC.bin

Questo repository contiene anche i dati del firmware RTL8710B incorporati nel codice sorgente del driver.

Licenza del firmware

La licenza del firmware è separata dalla licenza del codice sorgente del driver.

Questo repository non garantisce in modo assoluto che i file binari del firmware siano rilasciati sotto licenza GPL. Gli utenti sono tenuti a verificare i termini di licenza applicabili forniti dal produttore/fornitore prima di ridistribuire il firmware separatamente.

Origine e provenienza del firmware

Il codice sorgente del driver contiene avvisi di copyright di Realtek e riferimenti alla licenza GPLv2 nei singoli file sorgente.

Il progetto va quindi inteso come:

Un driver gestito dalla comunità e basato su codice sorgente derivato da Realtek; non è una distribuzione ufficiale di driver Realtek; modificato per la compilazione con le API del kernel Debian 13 / Linux 6.12; con copyright e note di licenza originali del codice sorgente preservati.

Il firmware deve essere trattato separatamente dal codice sorgente del driver per quanto riguarda le licenze e la ridistribuzione.

Considerazioni note sulla commutazione della modalità USB

Alcuni adattatori basati su questo hardware inizialmente si presentano come dispositivi di archiviazione/CD-ROM USB:

0BDA:1A2B

Dopo aver attivato la modalità USB, la funzione wireless diventa:

0BDA:B711

Se il dispositivo wireless non viene visualizzato, controllare il comando lsusb prima e dopo il cambio di modalità USB può aiutare a identificare il problema.

Un LED spento non significa necessariamente che l'adattatore non funzioni.

Verificare sempre l'effettiva enumerazione delle porte USB, il driver del kernel, l'interfaccia wireless e la connessione di rete.

Risoluzione dei problemi: Chiavetta USB non rilevata al riavvio (condizione di gara all'avvio)

Sui sistemi moderni come Debian 13 (Kernel 6.12+), potrebbe verificarsi un problema di sincronizzazione all'avvio: la chiavetta USB viene rilevata correttamente da lsusb (ID 0bda:b711), ma l'interfaccia di rete Wi-Fi (wlx...) non compare in ip link a meno che non si scolleghi e ricolleghi fisicamente il dispositivo.

Ciò accade perché il modulo viene caricato dal kernel prima che il firmware della chiavetta USB abbia completato la transizione elettronica successiva al cambio di modalità.

Per risolvere automaticamente e in modo permanente questo problema su qualsiasi porta USB del PC, segui questi passaggi per creare un servizio Systemd dedicato che esegua un riavvio software del dispositivo all'avvio.

1. Rimuovere il modulo dal caricamento anticipato
Assicurati che il modulo non sia presente nel file /etc/modules. Apri il file:

sudo nano /etc/modules
Se vedi la riga 8188gu, cancellala, salva ( CTRL+O, Enter), ed esci ( CTRL+X).

2. Creare il servizio di riavvio automatico
Crea un nuovo file di servizio in Systemd:

sudo nano /etc/systemd/system/rtl8188gu-restart.service
Incolla il seguente blocco di configurazione all'interno:

[Unit]
Description=Force Hardware Reset and Load RTL8188GU
After=multi-user.target usb-modeswitch.service

[Service]
Type=oneshot
RemainAfterExit=yes
# 1. Remove the module to avoid conflicts and zombie states
ExecStartPre=/sbin/modprobe -r 8188gu
# 2. Soft reset the USB device (Disable and re-enable authorization)
ExecStartPre=/bin/sh -c 'for dev in /sys/bus/usb/devices/*; do if [ -f "$dev/idVendor" ] && [ "$(cat $dev/idVendor)" = "0bda" ] && [ "$(cat $dev/idProduct)" = "b711" ]; then echo 0 > "$dev/authorized"; /bin/sleep 2; echo 1 > "$dev/authorized"; fi; done'
# 3. Wait for USB bus reactivation
ExecStartPre=/bin/sleep 2
# 4. Load final driver
ExecStart=/sbin/modprobe 8188gu

[Install]
WantedBy=multi-user.target

Salva il file ( CTRL+O, Enter) ed esci ( CTRL+X).

3. Abilita il servizio
Informa Systemd della modifica e abilita il servizio in modo che si avvii automaticamente ogni volta che il computer viene acceso:

sudo systemctl daemon-reload

sudo systemctl enable rtl8188gu-restart.service
Fatto! Al successivo riavvio, la chiavetta Unico/Realtek verrà ripristinata tramite software e l'interfaccia Wi-Fi sarà attiva e pronta all'uso fin dall'avvio, indipendentemente dalla porta USB in cui è inserita.

Compatibilità del kernel

Questo repository documenta nello specifico le modifiche necessarie per il kernel Debian 13 testato:

6.12.107+deb13-amd64

Altre versioni del kernel potrebbero richiedere ulteriori modifiche.

Riepilogo della verifica

La configurazione documentata è stata verificata attraverso le seguenti fasi:

Enumerazione del dispositivo USB. Commutazione della modalità USB su 0BDA:B711. Inizializzazione del firmware RTL8710B. Creazione dell'interfaccia wireless. Rilevamento della rete wireless. La connessione a NetworkManager è stata testata con successo. L'adattatore è stato utilizzato con successo per il Wi-Fi su Debian 13; i log documentati non indicano l'uso esclusivo del modulo personalizzato 8188gu per tale connessione.

Backup

È stato creato un backup locale completo dell'ambiente di lavoro separatamente da questo repository Git.

Il backup contiene:

Codice sorgente del driver; cronologia Git; file del firmware utilizzati durante i test; modulo del kernel installato; modulo del kernel ricompilato; informazioni di sistema; checksum; documentazione.

Il backup viene volutamente mantenuto separato dal repository Git pubblico.

Disclaimer

Questo repository è fornito come documentazione tecnica di una configurazione funzionante.

Le revisioni hardware, le revisioni del firmware, le versioni del kernel e le configurazioni della distribuzione possono variare da un sistema all'altro.

Non viene fornita alcuna garanzia che il driver funzioni senza modifiche su tutti i dispositivi RTL8188GU / RTL8710BU.

Quando si testa un driver wireless esterno al pacchetto di installazione, è sempre necessario disporre di una connessione di rete funzionante.

Crediti e ringraziamenti
Questo progetto è un fork aggiornato e adattato ai moderni kernel Linux. Un ringraziamento speciale a:

Un ringraziamento a @McMCCRU per il repository originale rtl8188gu , che ha fornito il codice sorgente e il supporto iniziale per versioni precedenti come Ubuntu 20.04.
Senza il loro lavoro iniziale di reverse engineering e pulizia del codice Realtek, non sarebbe stato possibile estendere il supporto per questa chiavetta Wi-Fi alle attuali versioni di Debian.

Ulteriori attività di compatibilità del kernel e test su Debian 13:

Gianluigi Rosas

Piattaforma di test:

Debian GNU/Linux 13.7 — Linux 6.12.107

Chiavetta UNICO WA2763 chip Realtek Semiconductor Corp. Adattatore WLAN RTL8188GU 802.11n


<img width="300" height="638" alt="Unicowa2763" src="https://github.com/user-attachments/assets/cbe73a20-87c3-43ba-82c0-2c668a5c70e1" />






















