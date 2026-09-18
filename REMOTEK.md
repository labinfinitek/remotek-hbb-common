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
server, server public key, default API server, auto-update forced off.
Protocol, networking and crypto are untouched. Licence: see `NOTICE`;
security: `SECURITY.md`.
