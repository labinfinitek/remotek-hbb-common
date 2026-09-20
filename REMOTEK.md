# Cosa cambia in questo fork rispetto a upstream

Base upstream: **rustdesk/hbb_common** al commit `7e1c392c62d39c364127307cd408421dd5f8cfb0`
(quello a cui punta RustDesk **1.4.9**). Upstream non ha tag: la base e' sempre
il commit puntato dal tag del client. `git log 7e1c392..remotek` mostra la
stessa lista come commit. Le decisioni (ADR) stanno nel repo interno Infinitek.

| File | Modifica | Motivo | ADR |
|---|---|---|---|
| `src/config.rs:72` | `APP_NAME` = `Remotek` | nome del prodotto: finestra, servizio, cartelle, registro, schema URL, stringhe tradotte | 0002 |
| `src/config.rs:70`, `:120` | `PROD_RENDEZVOUS_SERVER` e `RENDEZVOUS_SERVERS` = `remote.infinitek.it` | il client sa dove andare senza configurazione e non usa mai i server pubblici di RustDesk | 0002 |
| `src/config.rs:121` | `RS_PUB_KEY` = chiave pubblica del server Remotek | il server accetta solo client che conoscono la chiave | 0002 |
| `src/config.rs:76` | default dell'opzione `api-server` = `https://remote.infinitek.it` | l'API e' dietro HTTPS (443); senza, il client userebbe `http://<server>:21114` | 0002, 0008 |
| `src/config.rs:77` | `allow-auto-update` forzato a `N` | l'auto-update upstream installerebbe l'exe RustDesk sopra Remotek | 0008 |
| `src/config.rs:77` | nella stessa riga: `enable-lan-discovery`, `enable-record-session`, `allow-auto-record-incoming`, `enable-privacy-mode`, `enable-camera` forzate a `N` e `access-mode` forzata a vuota | default di privacy decisi il 2026-09-20 sul parere del consulente. **Rete locale**: e' l'unico flusso che manda MAC, ID, nome del PC e utente connesso a chiunque sia sulla stessa rete, senza passare dal server (client `src/lan.rs:36-62`); il PC non risponde piu' ai `ping` in broadcast, quindi non compare nella scheda "Rilevate" degli altri Remotek; la scansione in uscita parte ancora (`src/lan.rs:75-82`) e non porta dati del PC; il socket UDP di ascolto resta aperto come in upstream (`src/lan.rs:27-33`, avviato da `src/rendezvous_mediator.rs:139-143`, che non guarda l'opzione): l'opzione toglie la risposta, non la porta. **Registrazione**: copia lo schermo del cliente senza avvisare chi ci lavora davanti; sono due chiavi e non una, perche' la seconda strada non passa dai permessi (vedi sotto). **Modalita' privacy**: oscurare lo schermo del PC controllato contraddice l'accesso presidiato. **Videocamera**: su un PC aziendale e' videosorveglianza (art. 4 L. 300/1970). Il valore e' `N` e non vuoto: per le chiavi `enable-*` `option2bool` accende tutto cio' che non e' esattamente `N`, e per il prefisso `allow-` accende solo l'esatto `Y`. **`access-mode` vuota** tiene su le altre: il client concede il permesso senza nemmeno leggere la chiave `enable-*` appena `access-mode` vale `full` (`src/server/connection.rs:2370-2385`), quindi senza questa voce la combo Permessi delle impostazioni, l'IPC del servizio, `--option` o una strategia dell'API riaccendevano registrazione, modalita' privacy e videocamera; con il valore vuoto i preset di upstream "Accesso completo" e "Solo visualizzazione" non si scelgono piu' e valgono le singole caselle. **`allow-auto-record-incoming` a `N`** e' la seconda porta della registrazione: il registratore lato PC controllato nasce dalla sola opzione (`src/server/video_service.rs:578-581`, `:588-589`, `:959`, `:1042-1071`), senza leggere `enable-record-session` ne' `Permission::Recording`, scrive il filmato su disco in `video_save_directory` e non accende l'icona della telecamera nella finestra di accettazione, che mostra il permesso e non il registratore; era scrivibile da una strategia della console (`src/hbbs_http/sync.rs:286-303`), dalla casella "Automatically record incoming sessions" di Impostazioni > Registrazione, dall'IPC del servizio e da `--option`, cioe' proprio le vie che questa riga chiude per le altre. Prezzo: al cliente che volesse registrare per proprio audit le sessioni che riceve resta solo la strada del `custom.txt` firmato. Registrazione e modalita' privacy restano concedibili **dal solo cliente, per la singola sessione**, dalle due icone della finestra di accettazione (client `flutter/lib/desktop/pages/server_page.dart:794-840`, `src/server/connection.rs:774-776` per la registrazione e `:780-819` per la modalita' privacy): il tecnico non le riapre da solo. Per singolo cliente l'eccezione si fa con un `custom.txt` firmato che tocca le sei chiavi (sono tutte in `KEYS_SETTINGS`), non il preset; resta indisponibile finche' non e' in uso lo strumento di firma (ADR-0003). La **cattura schermata** non e' coperta da nessun default e resta possibile | 0017 |
| `src/config.rs:77` | nella stessa riga di `OVERWRITE_SETTINGS`: `approve-mode` forzato a `click` e `2fa` forzata a vuoto | accesso presidiato di default: ogni sessione in entrata si accetta con un clic sul PC, la password (monouso o permanente) da sola non basta. La 2FA resta spenta: con `click` il clic la sostituisce, e la verifica del codice sarebbe un secondo modo di aprire la sessione senza clic. Ne' l'utente ne' il tecnico li cambiano da impostazioni, riga di comando o strategia dell'API. `approve-mode` lo sostituisce solo un `custom.txt` con `override-settings` firmato con la chiave pubblica Remotek, quella di `read_custom_client` del client (client `src/common.rs`, ADR-0003: non e' piu' quella di RustDesk). La 2FA invece resta spenta in ogni modalita', anche con il `custom.txt` dell'accesso non presidiato: `override-settings` puo' solo dare a `2fa` un altro valore, e il segreto TOTP vale solo se cifrato con la chiave della singola macchina (`symmetric_crypt` in `src/password_security.rs`), quindi nessun `custom.txt` per cliente la riattiva; per riaverla va cambiata questa riga | 0016 |
| `SECURITY.md`, `NOTICE`, `REMOTEK.md` | documenti del fork | licenza, sicurezza, tracciabilita' | REGOLE 13 |

Cosa **non** cambia: protocollo (`protos/`), rete, cifratura, `ORG` (macOS),
porte di default, i link alla documentazione upstream (`LINK_DOCS_*`).
Nessun workflow: i valori sono controllati dallo script di verifica del client.

Licenza: AGPL-3.0 come RustDesk, vedi `NOTICE`. Segnalazioni di sicurezza:
`SECURITY.md`.

---

# What this fork changes (English)
Upstream base: **rustdesk/hbb_common** at commit `7e1c392` (the one RustDesk
1.4.9 points to). Only `src/config.rs` changes: application name, rendezvous
server, server public key, default API server, auto-update forced off,
attended access by default (`approve-mode` = `click`: every incoming session
must be accepted on the controlled PC; client 2FA forced off in every mode,
a `custom.txt` cannot turn it back on), and the privacy defaults forced off
(LAN discovery, session recording — both `enable-record-session` and
`allow-auto-record-incoming`, which starts a local recorder without asking
the permission — privacy mode, camera) with `access-mode` pinned to empty,
so that the "full access" preset cannot override them. A signed `custom.txt`
could turn those back on per customer, but that exception is not available
yet: the signing tool is not in use (ADR-0003). LAN discovery is off as an
answer, not as a socket: the UDP listener still binds, as upstream, and no
longer replies.
Protocol, networking and crypto are untouched. Licence: see `NOTICE`;
security: `SECURITY.md`.
