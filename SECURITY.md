# Sicurezza / Security

*Italiano prima, English below.*

## Segnalare una vulnerabilita'
Scrivi a **security@infinitek.it**. Non aprire issue pubbliche per
vulnerabilita'. Indica: prodotto e versione (tag `remotek-*` del client o di
questo repository), come riprodurre, impatto.

Questo repository e' la libreria comune (`libs/hbb_common`) del client
Remotek: tempi, safe harbor e pubblicazione degli avvisi sono quelli del
client, https://github.com/labinfinitek/remotek-client/blob/remotek/SECURITY.md
(conferma entro 3 giorni lavorativi, valutazione entro 10, correzione secondo
gravita', avviso nel changelog del client).

Versioni supportate: l'ultimo tag `remotek-*` di questo repository, che
coincide con lo stesso tag del client. Questa libreria e' una versione
modificata di `hbb_common` di RustDesk (AGPL-3.0): le vulnerabilita' del
codice upstream vanno segnalate anche a RustDesk; noi le riportiamo nel fork
appena la correzione e' disponibile.

## Reporting a vulnerability
Email **security@infinitek.it**. Please do not file public issues for
vulnerabilities. Include product and version, reproduction steps and impact.
This repository is the shared library of the Remotek client: the policy
(acknowledgement within 3 business days, triage within 10, fix by severity,
safe harbor for good-faith research) is the one published in the client
repository. Supported versions: the latest `remotek-*` tag.
