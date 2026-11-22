# Gestione C/C — Home Assistant Integration

[![HACS](https://img.shields.io/badge/HACS-Custom%20Integration-41BDF5.svg)](https://www.hacs.xyz/)
[![Integration version](https://img.shields.io/badge/integration-1.1.15-blue.svg)](https://github.com/stefanodr/gestione-cc-integration/releases)

**Italiano · English**

Integrazione personalizzata per Home Assistant che collega l'istanza locale di **Gestione C/C** e ne rende disponibili esclusivamente gli stati tecnici e tre comandi di manutenzione autorizzati. Non importa in Home Assistant dati contabili, conti, saldi o movimenti.

Custom Home Assistant integration for connecting to a local **Gestione C/C** instance. It exposes technical status only and three explicitly authorised maintenance commands. It does not import accounting data, accounts, balances or transactions into Home Assistant.

> **Separazione dei componenti / Component separation**
>
> - **Gestione C/C Home Assistant App** (server, interfaccia e database): [gestione-cc-home-assistant](https://github.com/stefanodr/gestione-cc-home-assistant)
> - **Questa integrazione HACS** (sensori tecnici e comandi): [gestione-cc-integration](https://github.com/stefanodr/gestione-cc-integration)
>
> L'integrazione non installa l'App: l'App deve essere già installata e funzionante. / The integration does not install the App: the App must already be installed and running.

---

## Indice / Contents

- [Italiano](#italiano)
  - [Requisiti](#requisiti)
  - [Installazione con HACS](#installazione-con-hacs)
  - [Configurazione](#configurazione)
  - [Entità create](#entità-create)
  - [API e compatibilità](#api-e-compatibilità)
  - [Sicurezza e privacy](#sicurezza-e-privacy)
  - [Risoluzione dei problemi](#risoluzione-dei-problemi)
  - [Aggiornamento e migrazione](#aggiornamento-e-migrazione)
  - [Segnalazioni](#segnalazioni)
- [English](#english)
  - [Requirements](#requirements)
  - [Installation with HACS](#installation-with-hacs)
  - [Configuration](#configuration)
  - [Created entities](#created-entities)
  - [API and compatibility](#api-and-compatibility)
  - [Security and privacy](#security-and-privacy)
  - [Troubleshooting](#troubleshooting)
  - [Updates and migration](#updates-and-migration)
  - [Issues](#issues)

# Italiano

## Requisiti

- Home Assistant con HACS installato.
- Home Assistant Core deve poter raggiungere via rete locale l'Home Assistant App Gestione C/C.
- App Gestione C/C installata, avviata e con API per l'integrazione disponibili.
- URL HTTPS locale dell'App, certificato CA privata dell'App e token dedicato.
- Il certificato CA deve essere copiato in un percorso leggibile da Home Assistant Core (ad esempio `/ssl/gestione-cc-ca.crt`).

L'integrazione è progettata per connessioni HTTPS verso IP privati o nomi host locali. Non accetta URL HTTP o indirizzi pubblici.

## Installazione con HACS

**Repository personalizzato HACS**

`https://github.com/stefanodr/gestione-cc-integration`

Categoria da selezionare: **Integrazione**.

[![Apri Home Assistant e aggiungi Gestione C/C a HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=stefanodr&repository=gestione-cc-integration&category=integration)

In alternativa, per l'aggiunta manuale:

1. Aprire **HACS → Integrazioni**.
2. Aprire il menu in alto a destra e selezionare **Repository personalizzati**.
3. Incollare il percorso sopra, scegliere **Integrazione** e premere **Aggiungi**.
4. Cercare **Gestione C/C**, installare e riavviare Home Assistant.
5. Aggiungere l'integrazione da **Impostazioni → Dispositivi e servizi → Aggiungi integrazione → Gestione C/C**.

### Installazione manuale

Per sviluppo o diagnosi, copiare la cartella `custom_components/gestione_cc` in `/config/custom_components/gestione_cc` e riavviare Home Assistant. Per gli aggiornamenti ordinari è preferibile HACS.

## Configurazione

Durante il flusso di configurazione inserire:

| Campo | Descrizione |
|---|---|
| URL locale Gestione C/C | Indirizzo HTTPS locale dell'App, senza percorso aggiuntivo, ad esempio `https://192.168.1.27:8443`. |
| Token integrazione | Token dedicato generato nelle **Impostazioni dell'app Gestione C/C** → riquadro **Integrazione Home Assistant**. Non si genera nella configurazione dell'add-on né nella schermata di configurazione dell'integrazione Home Assistant. |
| Percorso certificato CA | Percorso del certificato CA privata accessibile da Home Assistant Core, ad esempio `/ssl/gestione-cc-ca.crt`. |

Il flusso verifica che l'URL sia HTTPS e locale, che il certificato sia leggibile e che l'endpoint tecnico risponda con autenticazione valida. La verifica TLS non viene disabilitata.

### Come generare e inserire il token

1. Aprire l'applicazione **Gestione C/C** nel browser (non la pagina di Home Assistant). Nel proprio impianto, l'indirizzo locale è ad esempio `https://192.168.1.27:8443`.
2. Aprire **Impostazioni** all'interno dell'app Gestione C/C.
3. Individuare il riquadro **Integrazione Home Assistant**.
4. Premere **Genera / rinnova token**.
5. Copiare subito il token mostrato e conservarlo temporaneamente in un luogo sicuro: per sicurezza viene mostrato una sola volta e l'app ne conserva solo l'impronta crittografica, non il valore originale.
6. Tornare alla procedura **Impostazioni Home Assistant → Dispositivi e servizi → Aggiungi integrazione → Gestione C/C** e incollare il token nel campo **Token integrazione**, insieme all'URL locale e al percorso del certificato CA.

**Attenzione:** non premere prima **Revoca token**. Il pulsante **Genera / rinnova token** crea un nuovo token e invalida automaticamente quello precedente. Se il token viene perso o non viene copiato, premere nuovamente **Genera / rinnova token** e copiare il nuovo valore: il precedente non può essere recuperato. Non condividere il token e non inserirlo in segnalazioni, screenshot o messaggi.

## Entità create

### Blueprint: notifiche Google Drive

È disponibile un blueprint per ricevere notifiche sullo smartphone quando:

- Google Drive si scollega o perde l'autorizzazione;
- il backup notturno non riesce;
- il backup notturno non risulta completato entro l'orario previsto.

[![Importa il blueprint Gestione C/C - Notifiche Google Drive](https://my.home-assistant.io/badges/blueprint.svg)](https://my.home-assistant.io/redirect/blueprint/?blueprint_url=https%3A%2F%2Fgithub.com%2Fstefanodr%2Fgestione-cc-integration%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fgestione_cc%2Fnotifiche_google_drive.yaml)

**Configurazione**

1. Premere il pulsante **Importa il blueprint** qui sopra e confermare l'importazione in Home Assistant.
2. Aprire **Impostazioni → Automazioni e scene → Blueprint**, selezionare **Gestione C/C - Notifiche Google Drive** e creare una nuova automazione.
3. Selezionare i tre sensori binari dell'integrazione: Google Drive collegato, backup notturno non riuscito e backup notturno mancante.
4. Selezionare lo smartphone su cui è installata l'app Home Assistant Companion e salvare.

I sensori vengono scelti dall'interfaccia: non è necessario che i loro entity ID coincidano con quelli suggeriti dall'integrazione. Per esempio, Home Assistant può aver assegnato nomi personalizzati con prefisso `space_`.

La notifica di backup non riuscito viene inviata solo se, al momento dell'evento, il sensore Google Drive risulta ancora attivo. Le notifiche usano tag distinti per aggiornare la notifica dello stesso tipo invece di accumulare duplicati.


Le entità sono associate a un unico dispositivo **Gestione C/C**. Gli ID suggeriti sono:

### Sensori

| Entity ID | Informazione |
|---|---|
| `sensor.gestione_cc_app_version` | Versione dell'applicazione |
| `sensor.gestione_cc_addon_version` | Versione dell'Home Assistant App |
| `sensor.gestione_cc_uptime` | Tempo di attività |
| `sensor.gestione_cc_database_size` | Dimensione database |
| `sensor.gestione_cc_database_last_write` | Ultima modifica database |
| `sensor.gestione_cc_database_last_check` | Data ultimo controllo integrità |
| `sensor.gestione_cc_backup_local_count` | Numero di backup locali conservati |
| `sensor.gestione_cc_backup_local_last` | Data ultimo backup locale |
| `sensor.gestione_cc_backup_drive_last` | Data ultimo backup Google Drive |
| `sensor.gestione_cc_storage_total` | Spazio totale |
| `sensor.gestione_cc_storage_free` | Spazio libero |
| `sensor.gestione_cc_storage_used` | Spazio utilizzato |
| `sensor.gestione_cc_documents_count` | Numero di allegati |
| `sensor.gestione_cc_documents_bytes` | Dimensione allegati |
| `sensor.gestione_cc_jobs_queued` | Processi in coda |
| `sensor.gestione_cc_jobs_running` | Processi in esecuzione |
| `sensor.gestione_cc_jobs_done` | Processi completati |
| `sensor.gestione_cc_jobs_failed` | Processi non riusciti |
| `sensor.gestione_cc_ocr_queued` | Operazioni OCR in coda |
| `sensor.gestione_cc_ocr_running` | Operazioni OCR in elaborazione |
| `sensor.gestione_cc_ocr_failed` | Operazioni OCR non riuscite |

### Sensori binari

- `binary_sensor.gestione_cc_app_online`: disponibilità dell'app.
- `binary_sensor.gestione_cc_database_integrity`: esito dell'ultimo controllo d'integrità.
- `binary_sensor.gestione_cc_google_drive_connected`: stato del collegamento Google Drive.
- `binary_sensor.gestione_cc_google_drive_backup_failed`: errore del backup notturno Google Drive.
- `binary_sensor.gestione_cc_google_drive_backup_overdue`: backup notturno non completato entro l'orario previsto.

### Pulsanti

- `button.gestione_cc_local_backup`: avvia un backup locale del database SQLite.
- `button.gestione_cc_google_drive_backup`: avvia il backup su Google Drive, se il servizio è già collegato nell'App.
- `button.gestione_cc_integrity_check`: avvia il controllo d'integrità SQLite.

I pulsanti accodano un'operazione sul server dell'App. Non trasferiscono il database o i file di backup in Home Assistant. Il backup locale resta sullo stesso server e non sostituisce una copia indipendente.

## API e compatibilità

L'integrazione comunica con l'API tecnica autenticata dell'App, sotto il percorso `/api/integration/v1/`.

| Funzione | Endpoint |
|---|---|
| Verifica connessione/configurazione | `GET /health` |
| Stato database | `GET /database/status` |
| Stato backup | `GET /backups/status` |
| Stato spazio | `GET /storage/status` |
| Stato allegati | `GET /documents/status` |
| Stato processi | `GET /jobs/status` |
| Backup locale | `POST /commands/local-backup` |
| Backup Google Drive | `POST /commands/google-drive-backup` |
| Controllo integrità | `POST /commands/integrity-check` |

Tutte le richieste usano un token Bearer dedicato e la verifica del certificato TLS.

**Compatibilità di riferimento:** il codice estratto è la versione **1.1.15** e la documentazione di partenza identifica **Gestione C/C App 2.1.37** come versione abbinata. La coppia App 2.1.37 / integrazione 1.1.15 è il riferimento iniziale di questo repository. La compatibilità con versioni precedenti o successive dell'App non va presunta: sarà dichiarata dopo i relativi test. Le versioni dell'App e dell'integrazione sono indipendenti.

L'intervallo di aggiornamento generale è 60 secondi; i dati meno dinamici sono interrogati con intervalli interni più lunghi (database, backup e spazio ogni 5 minuti; allegati e processi ogni 2 minuti).

## Sicurezza e privacy

- La connessione è HTTPS con verifica CA obbligatoria.
- Sono accettati solo IP privati e host locali ammessi dal flusso di configurazione.
- Il token è conservato nei dati di configurazione di Home Assistant: proteggere accesso amministrativo e backup di Home Assistant.
- L'integrazione non crea sensori finanziari e non legge conti, saldi, movimenti, descrizioni o contenuti dei documenti.
- I file di backup non vengono trasferiti all'integrazione.
- I tre comandi di manutenzione sono autenticati e limitati alle operazioni esposte dall'API tecnica.
- Questa integrazione non abilita accesso WAN. Non pubblicare l'endpoint tramite port forwarding o reverse proxy pubblico.

## Risoluzione dei problemi

| Problema | Verifica |
|---|---|
| Impossibile aggiungere l'integrazione | Verificare che l'App sia avviata e che Home Assistant Core raggiunga l'URL HTTPS locale. |
| Certificato non trovato | Verificare percorso, nome file e permessi di lettura del certificato CA da parte di Home Assistant Core. |
| Errore TLS | Verificare che il certificato CA corrisponda all'App e che l'URL usato sia incluso nei nomi/IP coperti dal certificato. |
| Token non valido o revocato | Generare/verificare il token nell'App e riconfigurare l'integrazione. |
| Sensori non disponibili | Controllare i log di Home Assistant e lo stato dell'API tecnica dell'App. |
| Icona non visibile nella pagina dell'integrazione | Verificare che sia installata la revisione più recente del componente e riavviare Home Assistant. L'icona è inclusa in `custom_components/gestione_cc/brand/icon.png` ed è supportata da Home Assistant 2026.3 o successivo. |
| Pulsante non avvia l'operazione | Verificare i log dell'App, lo stato del relativo servizio e l'eventuale operazione già in esecuzione. |

## Aggiornamento e migrazione

Questa release iniziale del repository separato mantiene il dominio Home Assistant `gestione_cc`, il flusso di configurazione, gli identificativi univoci e le piattaforme esistenti. **Non rimuovere e non riconfigurare l'integrazione già installata solo perché è stato creato questo nuovo repository.** La migrazione dell'origine HACS è una procedura separata e deve essere verificata prima di essere eseguita, per evitare di rimuovere accidentalmente la voce di configurazione o cambiare gli entity ID.

La copia dell'integrazione nel repository dell'App viene mantenuta durante la fase di transizione. Non installare contemporaneamente due copie del componente con lo stesso dominio da sorgenti differenti.

## Segnalazioni

Aprire una segnalazione in [GitHub Issues](https://github.com/stefanodr/gestione-cc-integration/issues). Non allegare database, backup, token, certificati privati, cookie o dati bancari reali. Includere le versioni di Home Assistant, HACS, integrazione e App, insieme ai passaggi per riprodurre il problema.

---

# English

## Requirements

- Home Assistant with HACS installed.
- Home Assistant Core must be able to reach the Gestione C/C Home Assistant App over the local network.
- The App must be installed, running and exposing its integration API.
- The local HTTPS URL, the App's private CA certificate and a dedicated integration token are required.
- The CA certificate must be stored at a path readable by Home Assistant Core (for example, `/ssl/gestione-cc-ca.crt`).

The integration is designed for HTTPS connections to private IP addresses or local hostnames. HTTP and public URLs are rejected.

## Installation with HACS

**HACS custom repository**

`https://github.com/stefanodr/gestione-cc-integration`

Select category: **Integration**.

[![Open Home Assistant and add Gestione C/C to HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=stefanodr&repository=gestione-cc-integration&category=integration)

Alternatively, add it manually:

1. Open **HACS → Integrations**.
2. Open the top-right menu and select **Custom repositories**.
3. Paste the repository path above, select **Integration**, and click **Add**.
4. Find **Gestione C/C**, install it and restart Home Assistant.
5. Add the integration from **Settings → Devices & services → Add integration → Gestione C/C**.

### Manual installation

For development or troubleshooting, copy `custom_components/gestione_cc` to `/config/custom_components/gestione_cc` and restart Home Assistant. HACS is recommended for normal updates.

## Configuration

Enter the following values in the configuration flow:

| Field | Description |
|---|---|
| Local Gestione C/C URL | Local HTTPS address of the App, without an additional path, e.g. `https://192.168.1.27:8443`. |
| Integration token | Dedicated token generated in the **Gestione C/C application settings** → **Home Assistant Integration** panel. It is not generated in the add-on settings or in the Home Assistant integration configuration screen. |
| CA certificate path | Path to the private CA certificate readable by Home Assistant Core, e.g. `/ssl/gestione-cc-ca.crt`. |

The configuration flow checks that the URL is local and uses HTTPS, that the certificate is readable, and that the authenticated technical health endpoint responds correctly. TLS verification is never disabled.

### How to generate and enter the token

1. Open the **Gestione C/C application** in a browser (not the Home Assistant page). Your local address may look like `https://192.168.1.27:8443`.
2. Open **Settings** inside the Gestione C/C application.
3. Find the **Home Assistant Integration** panel.
4. Click **Generate / renew token**.
5. Copy the token immediately and keep it temporarily in a secure place. For security, it is displayed only once; the App stores only its cryptographic hash, not the original value.
6. Return to **Home Assistant Settings → Devices & services → Add integration → Gestione C/C** and paste the token into the **Integration token** field, together with the local URL and CA certificate path.

**Important:** do not click **Revoke token** first. **Generate / renew token** creates a new token and automatically invalidates the previous one. If the token is lost or was not copied, click **Generate / renew token** again and copy the newly displayed value; the previous token cannot be recovered. Never share the token or include it in issue reports, screenshots or messages.

## Created entities

### Blueprint: Google Drive notifications

A blueprint is available to send smartphone notifications when:

- Google Drive is disconnected or its authorisation expires;
- the nightly backup fails;
- the nightly backup has not completed by the expected time.

[![Import the Gestione C/C - Google Drive notifications blueprint](https://my.home-assistant.io/badges/blueprint.svg)](https://my.home-assistant.io/redirect/blueprint/?blueprint_url=https%3A%2F%2Fgithub.com%2Fstefanodr%2Fgestione-cc-integration%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fgestione_cc%2Fnotifiche_google_drive.yaml)

**Setup**

1. Click **Import the blueprint** above and confirm the import in Home Assistant.
2. Open **Settings → Automations & scenes → Blueprints**, select **Gestione C/C - Notifiche Google Drive**, and create an automation.
3. Select the integration's three binary sensors: Google Drive connected, nightly backup failed and nightly backup overdue.
4. Select the smartphone running the Home Assistant Companion app and save.

Sensors are selected in the UI, so their entity IDs do not need to match the integration's suggested IDs. For example, Home Assistant may assign customised IDs with a `space_` prefix.

The backup-failed notification is sent only if the Google Drive connected sensor is still on when the event occurs. Separate notification tags are used so a new notification of the same type updates the existing one instead of creating duplicates.


All entities belong to one **Gestione C/C** device. Suggested entity IDs are listed above in the Italian section; they include 21 sensors, five binary sensors and three maintenance buttons. Home Assistant may adjust entity IDs if conflicts already exist.

The three buttons queue maintenance operations on the App server. They do not transfer the database or backup files into Home Assistant. A local backup remains on the same server and is not a substitute for an independent copy.

## API and compatibility

The integration communicates with the App's authenticated technical API under `/api/integration/v1/`.

| Function | Endpoint |
|---|---|
| Connection/configuration check | `GET /health` |
| Database status | `GET /database/status` |
| Backup status | `GET /backups/status` |
| Storage status | `GET /storage/status` |
| Documents status | `GET /documents/status` |
| Jobs status | `GET /jobs/status` |
| Local backup | `POST /commands/local-backup` |
| Google Drive backup | `POST /commands/google-drive-backup` |
| Integrity check | `POST /commands/integrity-check` |

All requests use a dedicated Bearer token and mandatory TLS certificate verification.

**Reference compatibility:** the extracted code is integration version **1.1.15**; the source documentation identifies **Gestione C/C App 2.1.37** as its paired App version. App 2.1.37 / integration 1.1.15 is the initial reference combination for this repository. Compatibility with earlier or later App versions must not be assumed until tested. App and integration versions are independent.

The general coordinator interval is 60 seconds. Less dynamic data use longer internal polling intervals (database, backups and storage every 5 minutes; documents and jobs every 2 minutes).

## Security and privacy

- HTTPS with mandatory CA verification.
- Only private IP addresses and local hostnames accepted by the configuration flow.
- The token is stored in Home Assistant configuration: protect administrative access and Home Assistant backups.
- No financial sensors are created. The integration does not read accounts, balances, transactions, descriptions or document contents.
- Backup files are not transferred to the integration.
- The three maintenance commands are authenticated and limited to the operations exposed by the technical API.
- This integration does not enable WAN access. Do not expose its endpoint through public port forwarding or a public reverse proxy.

## Troubleshooting

| Problem | Check |
|---|---|
| Integration cannot be added | Make sure the App is running and Home Assistant Core can reach its local HTTPS URL. |
| Certificate not found | Check the path, filename and Home Assistant Core read permissions. |
| TLS error | Check that the CA certificate belongs to the App and that the URL hostname/IP is covered by the certificate. |
| Invalid or revoked token | Verify or generate the token in the App and reconfigure the integration. |
| Entities unavailable | Check Home Assistant logs and the App's technical API status. |
| Integration icon is missing | Make sure the latest component revision is installed and restart Home Assistant. The icon is bundled at `custom_components/gestione_cc/brand/icon.png` and is supported by Home Assistant 2026.3 or later. |
| Button does not start an operation | Check App logs, service status and whether another operation is already running. |

## Updates and migration

This initial release in the separate repository preserves the Home Assistant domain `gestione_cc`, configuration flow, unique IDs and existing platforms. **Do not remove or reconfigure an already installed integration just because this new repository has been created.** Moving the HACS source is a separate procedure and must be verified before execution to avoid accidentally removing the config entry or changing entity IDs.

The integration copy in the App repository is retained during the transition. Do not install two copies of the component with the same domain from different sources at the same time.

## Issues

Report problems in [GitHub Issues](https://github.com/stefanodr/gestione-cc-integration/issues). Never attach databases, backups, tokens, private certificates, cookies or real banking data. Include Home Assistant, HACS, integration and App versions, plus clear reproduction steps.

---

## Credits and version

- Integration domain: `gestione_cc`
- Integration version: **1.1.15**
- Reference App version: **2.1.37**
- Source App repository: [stefanodr/gestione-cc-home-assistant](https://github.com/stefanodr/gestione-cc-home-assistant)
- Issues: [stefanodr/gestione-cc-integration/issues](https://github.com/stefanodr/gestione-cc-integration/issues)

Copyright © 2026 Stefano Di Ricco. See the repository owner for licensing and usage permissions.
