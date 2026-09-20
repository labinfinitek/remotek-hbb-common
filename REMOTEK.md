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
| `src/config.rs:77` | nella stessa riga: `enable-lan-discovery`, `enable-record-session`, `enable-privacy-mode`, `enable-camera` forzate a `N` e `access-mode` forzata a vuota | default di privacy decisi il 2026-09-20 sul parere del consulente. **Rete locale**: e' l'unico flusso che manda MAC, ID, nome del PC e utente connesso a chiunque sia sulla stessa rete, senza passare dal server (client `src/lan.rs:36-62`); il PC non risponde piu' ai `ping` in broadcast, quindi non compare nella scheda "Rilevate" degli altri Remotek; la scansione in uscita parte ancora (`src/lan.rs:75-82`) e non porta dati del PC. **Registrazione**: copia lo schermo del cliente senza avvisare chi ci lavora davanti. **Modalita' privacy**: oscurare lo schermo del PC controllato contraddice l'accesso presidiato. **Videocamera**: su un PC aziendale e' videosorveglianza (art. 4 L. 300/1970). Il valore e' `N` e non vuoto: per le chiavi `enable-*` `option2bool` accende tutto cio' che non e' esattamente `N`. **`access-mode` vuota** tiene su le altre: il client concede il permesso senza nemmeno leggere la chiave `enable-*` appena `access-mode` vale `full` (`src/server/connection.rs:2370-2385`), quindi senza questa voce la combo Permessi delle impostazioni, l'IPC del servizio, `--option` o una strategia dell'API riaccendevano registrazione, modalita' privacy e videocamera; con il valore vuoto i preset di upstream "Accesso completo" e "Solo visualizzazione" non si scelgono piu' e valgono le singole caselle. Registrazione e modalita' privacy restano concedibili **dal solo cliente, per la singola sessione**, dalle due icone della finestra di accettazione (client `flutter/lib/desktop/pages/server_page.dart:794-840`, `src/server/connection.rs:774-779`): il tecnico non le riapre da solo. Per singolo cliente l'eccezione si fa con un `custom.txt` firmato che tocca le chiavi `enable-*` (sono tutte in `KEYS_SETTINGS`, come `access-mode`), non il preset; resta indisponibile finche' non e' in uso lo strumento di firma (ADR-0003). La **cattura schermata** non e' coperta da nessun default e resta possibile | 0017 |
| `src/config.rs:77` | nella stessa riga di `OVERWRITE_SETTINGS`: `approve-mode` forzato a `click` e `2fa` forzata a vuoto | accesso presidiato di default: ogni sessione in entrata si accetta con un clic sul PC, la password (monouso o permanente) da sola non basta. La 2FA resta spenta: con `click` il clic la sostituisce, e la verifica del codice sarebbe un secondo modo di aprire la sessione senza clic. Ne' l'utente ne' il tecnico li cambiano da impostazioni, riga di comando o strategia dell'API. `approve-mode` lo sostituisce solo un `custom.txt` con `override-settings` firmato con la chiave di `read_custom_client` del client (oggi ancora quella di RustDesk, finche' non entra la nostra). La 2FA invece resta spenta in ogni modalita', anche con il `custom.txt` dell'accesso non presidiato: `override-settings` puo' solo dare a `2fa` un altro valore, e il segreto TOTP vale solo se cifrato con la chiave della singola macchina (`symmetric_crypt` in `src/password_security.rs`), quindi nessun `custom.txt` per cliente la riattiva; per riaverla va cambiata questa riga | 0013, 0008 |
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
a `custom.txt` cannot turn it back on), and four privacy defaults forced off
(LAN discovery, session recording, privacy mode, camera) with `access-mode`
pinned to empty, so that the "full access" preset cannot override them; a
signed `custom.txt` can turn those four back on per customer.
Protocol, networking and crypto are untouched. Licence: see `NOTICE`;
security: `SECURITY.md`.
