# php2shell-fix

Strumenti per verificare, correggere e mitigare le vulnerabilità critiche
dell'interprete PHP **CVE-2026-109618** e **CVE-2026-109619** ("php2shell")
su server Linux.

> **English summary.** Bash tools to check, patch and mitigate the PHP
> "php2shell" vulnerabilities (CVE-2026-109618 / CVE-2026-109619) on
> Ubuntu/Debian and Fedora/RHEL, including end-of-life PHP branches (7.4,
> 8.0, 8.1), plus read-only indicators-of-compromise checks. Documentation
> and messages are in Italian. Provided **AS IS, without any warranty**: see
> [DISCLAIMER.md](DISCLAIMER.md).

---

## ⚠️ Avvertenze ed esclusione di responsabilità

> **Il software è fornito "così com'è" ("AS IS"), senza garanzie di alcun tipo,
> esplicite o implicite.** Nei limiti massimi consentiti dalla legge, gli autori
> e i contributori non sono responsabili per alcun danno diretto o indiretto
> (interruzioni di servizio, perdita di dati, malfunzionamenti, mancata o
> incompleta correzione delle vulnerabilità, compromissioni) derivante dal suo
> utilizzo. **L'uso avviene a esclusivo rischio e responsabilità di chi lo
> esegue.**
>
> Prima di qualsiasi utilizzo:
>
> - deve essere impiegato **solo da personale tecnico competente**, in grado di
>   valutarne gli effetti e di ripristinare i sistemi;
> - deve essere **provato in un ambiente di test** rappresentativo dei sistemi
>   di destinazione;
> - devono essere disponibili **backup completi e verificati** (o snapshot) di
>   sistemi, configurazioni e dati, con una procedura di ripristino collaudata.
>
> Il software non sostituisce gli avvisi di sicurezza ufficiali. Testo completo
> in [DISCLAIMER.md](DISCLAIMER.md); prevale in ogni caso la [licenza](LICENSE).

Indicazioni pratiche:

- Partire sempre da `--check` e `--ioc`, che **non modificano nulla**.
- Gli script aggiornano pacchetti, modificano configurazioni e riavviano
  servizi: pianificarne l'esecuzione in una finestra di manutenzione.
- Per i rami PHP fuori supporto (7.4, 8.0, 8.1) la correzione è una **patch
  minima** pubblicata dal PHP Security Team: chiude queste due CVE, non gli
  altri difetti scoperti dopo la fine del supporto. È una misura ponte in
  attesa della migrazione a un ramo supportato.
- Le **mitigazioni** (`--mitiga`) sono **parziali**: riducono il rischio, non
  sostituiscono l'aggiornamento.

---

## Le vulnerabilità in breve

| | |
|---|---|
| CVE | CVE-2026-109618, CVE-2026-109619 |
| Gravità | CVSS v4.0 9.2 |
| Effetto | esecuzione di codice da remoto **senza autenticazione**, tramite richieste HTTP costruite ad arte, prima dell'esecuzione del codice applicativo |
| Configurazioni | confermato con Apache + mod_php; in condizioni specifiche anche con PHP-FPM |
| Versioni corrette | 8.2.35, 8.3.36, 8.4.27, 8.5.12 |
| Rami fuori supporto | 7.4, 8.0, 8.1: solo patch minima sul sorgente, distribuita in backport da distribuzioni e repository |
| Sfruttamento | Proof-of-Concept pubblici; primo tentativo osservato il 17/10/2026, prima della divulgazione |

L'aggiornamento del solo CMS o dei plugin **non** corregge la vulnerabilità:
il difetto è nell'interprete PHP. Dopo l'aggiornamento vanno **riavviati**
PHP-FPM e/o Apache, altrimenti i processi in esecuzione continuano a usare il
codice vulnerabile.

---

## Contenuto

| Percorso | A cosa serve |
|---|---|
| [`ubuntu/php-cve-fix-ubuntu.sh`](ubuntu/php-cve-fix-ubuntu.sh) | Controllo, aggiornamento, mitigazioni e indicatori su **Ubuntu / Debian** (apt) |
| [`fedora/php-cve-fix-fedora.sh`](fedora/php-cve-fix-fedora.sh) | Lo stesso su **Fedora / RHEL / Rocky / Alma** (dnf), comprese le installazioni parallele di Remi |
| [`ioc.txt.example`](ioc.txt.example) | Modello del file di indirizzi IP per `--ioc` |
| [`DISCLAIMER.md`](DISCLAIMER.md) | Esclusione di garanzia e di responsabilità |
| [`LICENSE`](LICENSE) | Testo della licenza EUPL-1.2 |

Ogni script ha l'opzione `--help`.

---

## Quale strumento usare

| Situazione | Cosa fare |
|---|---|
| Ubuntu / Debian con release supportata | `ubuntu/php-cve-fix-ubuntu.sh`; per PHP 7.4/8.0/8.1 aggiungere `--aggiungi-sury` |
| Fedora supportata | `fedora/php-cve-fix-fedora.sh` (la correzione arriva dai repository ufficiali) |
| RHEL / Rocky / Alma 8 o 9 | `fedora/php-cve-fix-fedora.sh`; per stream ritirati aggiungere `--aggiungi-remi` |
| Release fuori supporto (es. Ubuntu 20.04, Debian 11) | spesso non esiste un pacchetto corretto gratuito: `--mitiga` subito e vedi [Release fuori supporto](#release-fuori-supporto) |
| Qualsiasi server esposto | `--ioc` per cercare tracce di attacchi dal 17/10/2026 |

---

## Script per Ubuntu / Debian

### Opzioni

| Opzione | Cosa fa | Modifica il sistema |
|---|---|---|
| *(nessuna)* | Controlla ogni ramo PHP installato e, se il repository ha la correzione, aggiorna **solo** i pacchetti di quel ramo; riavvia PHP-FPM e Apache | sì (sudo) |
| `--check` | Solo controllo. Senza sudo usa l'elenco pacchetti già presente | no |
| `--aggiungi-sury` | Se il repository non offre la correzione, aggiunge il repository di Ondřej Surý (`ppa:ondrej/php` su Ubuntu, `packages.sury.org` su Debian) e riprova | sì |
| `--mitiga` | Sui rami che restano vulnerabili applica le mitigazioni temporanee | sì |
| `--post-max=VALORE` | Con `--mitiga`: limite per `post_max_size` (predefinito `32M`) | — |
| `--ioc` | Analisi in sola lettura degli indicatori di compromissione. Da sola non modifica nulla | no |
| `--ioc-ip=FILE` | File con gli indirizzi IP da cercare nei log (attiva anche `--ioc`) | — |
| `--help` | Aiuto | no |

### Esempi

```bash
chmod +x ubuntu/php-cve-fix-ubuntu.sh

# 1. Solo controllo e analisi (nessuna modifica)
./ubuntu/php-cve-fix-ubuntu.sh --check
sudo ./ubuntu/php-cve-fix-ubuntu.sh --ioc

# 2. Server con PHP supportato (8.2 e successivi): aggiornamento, poi analisi
sudo ./ubuntu/php-cve-fix-ubuntu.sh
sudo ./ubuntu/php-cve-fix-ubuntu.sh --ioc

# 3. Server con PHP fuori supporto (7.4, 8.0, 8.1)
sudo ./ubuntu/php-cve-fix-ubuntu.sh --aggiungi-sury --mitiga --ioc
```

`--ioc` **da solo** esegue soltanto l'analisi e non aggiorna nulla. Insieme a
opzioni che intervengono (`--aggiungi-sury`, `--mitiga`) l'analisi viene
eseguita alla fine; senza altre opzioni, per aggiornare e poi analizzare servono
due esecuzioni, come nell'esempio 2.

### Come decide se un ramo è corretto

Basta una delle due condizioni:

1. la versione PHP è uguale o successiva alla prima versione corretta
   (8.2.35, 8.3.36, 8.4.27, 8.5.12);
2. il changelog del pacchetto cita una delle due CVE. È il caso dei backport,
   in cui la versione resta invariata (es. Ubuntu 22.04 `8.1.2-1ubuntu2.28`,
   Sury `7.4.33-31`, `8.0.30-25`, `8.1.34-9`).

Per il pacchetto disponibile nel repository il changelog viene letto
scaricando il `.deb` in una cartella temporanea, **senza installarlo**.

### Il repository Sury e le release fuori supporto

Sury pubblica pacchetti **solo per le release di Ubuntu e Debian ancora
supportate**. Per le release uscite dal supporto standard (ad esempio
Ubuntu 20.04, Debian 11) la cartella della release esiste ma è vuota.
Lo script se ne accorge dopo `apt-get update`, **rimuove il repository** appena
aggiunto e indica le alternative (vedi [Release fuori supporto](#release-fuori-supporto)).

Dopo l'aggiunta di Sury, apt proporrà le versioni Sury anche per gli altri
pacchetti PHP: lo script aggiorna comunque solo i rami vulnerabili.

---

## Script per Fedora / RHEL

Stesse opzioni dello script Ubuntu, con `--aggiungi-remi` al posto di
`--aggiungi-sury`.

Controlla il PHP **di sistema** (`php-common`) e le installazioni **parallele**
di Remi (`php74-php-common`, `php82-php-common`, ...).

| Opzione | Differenze rispetto a Ubuntu |
|---|---|
| `--aggiungi-remi` | Solo RHEL / Rocky / Alma **8 e 9**. Aggiunge EPEL e Remi e, per il PHP di sistema, passa allo stream Remi **dello stesso ramo** (es. `php:8.1` → `php:remi-8.1`), senza cambiare versione maggiore |
| `--mitiga` | Percorsi Red Hat: `/etc/php.d`, `/etc/php-fpm.d` (di sistema), `/etc/opt/remi/phpXX/...` (paralleli). Con SELinux attivo i file creati ricevono il contesto corretto |

Su **Fedora** le release supportate ricevono la correzione dai repository
ufficiali; se è già in `updates-testing` ma non ancora in stabile, lo script lo
segnala e indica il comando per installarla. Una Fedora fuori supporto va
aggiornata di release.

Su **RHEL 10** e derivate `--aggiungi-remi` non è previsto: la correzione va
presa dai repository del fornitore.

```bash
chmod +x fedora/php-cve-fix-fedora.sh
./fedora/php-cve-fix-fedora.sh --check
sudo ./fedora/php-cve-fix-fedora.sh --aggiungi-remi --mitiga --ioc
```

---

## Mitigazioni temporanee (`--mitiga`)

Applicate **solo** ai rami/installazioni che restano vulnerabili dopo il
tentativo di aggiornamento.

| Misura | CVE | Note |
|---|---|---|
| `session.upload_progress.enabled = 0` | 109619 | Si perde solo la barra di avanzamento degli upload basata su questa funzione |
| `fatal_error_backtraces = 0` | 109619 | Solo PHP 8.5 |
| `post_max_size` a 32M (o `--post-max`) | 109618 | **Solo abbassato**, mai alzato. Se il valore attuale è più alto viene segnalato: verificare gli upload dell'applicazione. `upload_max_filesize` viene riportato sotto il nuovo limite |
| `memory_limit = -1` nei pool PHP-FPM | 109618 | Impostato con `php_admin_value` (l'applicazione non può sovrascriverlo), **sempre** insieme a un limite di memoria del servizio systemd (`MemoryMax` = 75% della RAM), così un worker fuori controllo viene fermato senza bloccare la macchina. **Non applicato** con mod_php né senza systemd |

Garanzie:

- prima del riavvio la configurazione viene verificata (`php-fpm -t`,
  `apache2ctl -t` / `httpd -t`); se non è valida, le modifiche ai pool vengono
  annullate e il servizio non viene riavviato;
- i pool PHP-FPM vengono salvati in copia (`*.bak-cve-*`) alla prima modifica;
- le modifiche sono marcate e vengono **tolte automaticamente** quando il ramo
  risulta corretto (anche a un'esecuzione successiva).

File creati o modificati:

| | Ubuntu / Debian | Fedora / RHEL |
|---|---|---|
| Direttive PHP | `/etc/php/X.Y/{apache2,fpm}/conf.d/99-cve-2026-109618-mitigazione.ini` | `/etc/php.d/99-cve-2026-109618-mitigazione.ini` (o `/etc/opt/remi/phpXX/php.d/`) |
| Pool PHP-FPM | blocco marcato in fondo ai file di `/etc/php/X.Y/fpm/pool.d/` | blocco marcato in fondo ai file di `/etc/php-fpm.d/` |
| Limite di memoria | `/etc/systemd/system/phpX.Y-fpm.service.d/99-cve-2026-109618-mitigazione.conf` | `/etc/systemd/system/php-fpm.service.d/99-cve-2026-109618-mitigazione.conf` |

---

## Indicatori di compromissione (`--ioc`)

Analisi **in sola lettura** dei log modificati dal 17/10/2026. Funziona anche
senza sudo, limitatamente ai log leggibili.

| Controllo | Cosa cerca |
|---|---|
| Indirizzi IP | IP indicati nel file di indicatori, nei log di accesso di Apache / Nginx (confronto sull'indirizzo intero) |
| POST con errore | Richieste POST con risposta 5xx, raggruppate per IP |
| Memoria | Errori `Allowed memory size of ... bytes exhausted` |
| Crash | Segmentation fault / abort dei worker nei log e nel log del kernel |
| Processi | Processi figli di Apache / PHP-FPM che non sono Apache / PHP-FPM (es. `sh`, `curl`, `wget`, `nc`, `python`) |
| File | File `.php` modificati nelle document root configurate |

### File degli indirizzi IP

Nessun indirizzo IP è scritto nel codice. Il controllo sugli IP si attiva solo
se esiste un file di indicatori:

- `--ioc-ip=/percorso/file`, oppure
- `ioc.txt` nella stessa cartella dello script.

Formato: un IPv4 o IPv6 per riga, `#` per i commenti (vedi
[`ioc.txt.example`](ioc.txt.example)). Gli indirizzi vanno presi dalle proprie
fonti (CERT, SOC, bollettini) rispettandone le regole di diffusione: per questo
`ioc.txt` è escluso dal repository tramite `.gitignore`.

### Se trova qualcosa

Gli indicatori **non** sono necessariamente compromissioni: un rilascio
legittimo modifica file `.php`, un'applicazione può generare errori 5xx. Se però
compaiono **IP dell'elenco** o **processi sospetti**:

1. non riavviare né "pulire" la macchina;
2. conservare log e, se possibile, uno snapshot;
3. avvisare chi gestisce la sicurezza e valutare l'isolamento del servizio.

---

## Release fuori supporto

Sulle release uscite dal supporto standard (ad esempio Ubuntu 20.04 o
Debian 11) spesso **non esiste un pacchetto PHP corretto gratuito**: Sury non
pubblica più per queste release e le correzioni della distribuzione arrivano
solo tramite programmi di supporto esteso. Le alternative, in ordine di
rapidità:

| Alternativa | Note |
|---|---|
| `--mitiga` | Subito; riduce il rischio ma non chiude la vulnerabilità |
| Limitare l'esposizione | Restringere l'accesso al servizio (rete interna, VPN, WAF) o metterlo in manutenzione finché non è corretto |
| Supporto esteso della distribuzione | Ubuntu Pro (ESM) corregge il PHP originale della distribuzione; Freexian PHP LTS fornisce pacchetti Sury per le release fuori supporto (entrambi a pagamento) |
| Compilazione dal sorgente | PHP compilato con la patch ufficiale del ramo (vedi sotto). Si esce dai pacchetti della distribuzione: gli aggiornamenti successivi sono manuali |
| Migrazione | A una release supportata: è la soluzione strutturale |

### Patch ufficiali per i rami fuori supporto

Pubblicate nel repository ufficiale `php/php-src` ("Avoid state corruption via
bailout"), per chi compila PHP dal sorgente:

| Ramo | Commit |
|---|---|
| 7.4 | [`712e60f9`](https://github.com/php/php-src/commit/712e60f9edfd03f0960fc4fc3c2c375f42b90aa2) |
| 8.0 | [`82f8f31b`](https://github.com/php/php-src/commit/82f8f31b16a035c1b67cc253e79f7cc288f4cc20) |
| 8.1 | [`1d7dec17`](https://github.com/php/php-src/commit/1d7dec178a094e6e75b229ce22754045f8c6070a) |

Aggiungendo `.patch` all'indirizzo del commit si ottiene il file da applicare
con `patch -p1`. Le patch vanno applicate all'ultima release del ramo
(7.4.33, 8.0.30, 8.1.34).

---

## Codici di uscita

| Codice | Significato |
|---|---|
| `0` | Nessun PHP vulnerabile e nessun indicatore trovato |
| `1` | Resta almeno un PHP vulnerabile (anche se mitigato) oppure `--ioc` ha trovato indicatori da verificare |
| `2` | Errore (parametri, permessi, rete, repository) |

Eseguiti con sudo, gli script salvano un log nella cartella `log/` accanto
allo script.

---

## Stato dei test

| Componente | Provato | Non ancora provato |
|---|---|---|
| Script Ubuntu | Ubuntu 24.04 (PHP 8.3, Apache/mod_php, PHP-FPM): aggiornamento, mitigazioni con ripristino del pool identico all'originale, `--ioc`. Su Ubuntu 20.04 reale è stata verificata l'assenza di pacchetti Sury; il rilevamento automatico e la rimozione del repository sono provati in simulazione | Debian; `--aggiungi-sury` su una release supportata con PHP 7.4/8.0/8.1; `MemoryMax` con systemd in esercizio |
| Script Fedora | Logica delle mitigazioni su struttura Red Hat simulata, confronto versioni, `--ioc` | Esecuzione su Fedora / RHEL reale; passaggio di stream Remi |

Segnalazioni di prove su altre piattaforme sono benvenute.

---

## Limiti noti

- Gli script gestiscono PHP installato da **pacchetto** (apt / dnf). PHP compilato
  a mano, in container o in pannelli di hosting va verificato a parte.
- I controlli di `--ioc` sono euristici e pensati per il formato di log
  *combined* di Apache / Nginx.
- La patch per i rami fuori supporto disattiva sempre l'upload progress delle
  sessioni.

---

## Riferimenti

- PHP Security Advisory [GHSA-w8hw-w9fx-j79w](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w) (CVE-2026-109618)
- PHP Security Advisory [GHSA-6wmf-4rv5-p3rp](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp) (CVE-2026-109619)
- [PHP News Archive 2026](https://www.php.net/archive/2026.php)
- [CERT-AGID – Due vulnerabilità critiche in PHP](https://cert-agid.gov.it/news/due-vulnerabilita-critiche-in-php-consentono-lesecuzione-di-codice-da-remoto-rilasciati-aggiornamenti-di-sicurezza/)
- [CERT-EU Security Advisory 2026-018](https://cert.europa.eu/publications/security-advisories/2026-018/)
- [Repository PHP di Ondřej Surý](https://deb.sury.org/) · [Repository Remi](https://rpms.remirepo.net/)

---

## Licenza

Distribuito con licenza **European Union Public Licence v. 1.2 (EUPL-1.2)**:
testo completo nel file [LICENSE](LICENSE).

Il software è fornito "così com'è", senza garanzie di alcun tipo (articoli 7 e 8
della licenza). Vedi anche [DISCLAIMER.md](DISCLAIMER.md).
