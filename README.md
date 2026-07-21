# wp-migrate

CLI shell per migrare un'istanza WordPress tramite una cartella `.wp-migrate` versionata con git. I file sincronizzati sono limitati a `wp-content/uploads`.

## Requisiti

- `git`
- `wp` (WP-CLI)
- `tar`
- `rsync`
- `split`
- `bash`

## Installazione

```bash
chmod +x bin/wp-migrate
```

Se `wp` non è nel `PATH`, puoi indicare manualmente il binario di WP-CLI:

```bash
WP_CLI_BIN=/percorso/al/tuo/wp ./bin/wp-migrate init
```

Se stai usando Local e `wp` fallisce con `env: php: No such file or directory`, indica anche il binario PHP fornito da Local:

```bash
PHP_BIN=/percorso/al/php-di-local \
WP_CLI_BIN=/percorso/al/wp \
./bin/wp-migrate init
```

Esempi:

```bash
./bin/wp-migrate init
./bin/wp-migrate push
./bin/wp-migrate pull
./bin/wp-migrate sync
./bin/wp-migrate sync --db-only
./bin/wp-migrate import ./data/cignozen-migrate-20260721075601.sql.gz
```

## Configurazione

`.wp-migrate/config` e `.wp-migrate/config.local` sono **entrambi puramente locali**: non vengono mai committati nel repository git interno a `.wp-migrate` (sono elencati nel suo `.gitignore`), quindi ogni macchina ha la propria copia e un `pull` non può mai sovrascriverli con i valori di un'altra macchina.

Puoi crearli in due modi:

- interattivamente, con `wp-migrate init` (risponde alle domande e li scrive per te);
- manualmente, copiando i template presenti in questo repository:

```bash
cp config.example <percorso-sito-wordpress>/.wp-migrate/config
cp config.local.example <percorso-sito-wordpress>/.wp-migrate/config.local
# poi modifica i valori con un editor
```

Con `config`/`config.local` già presenti puoi lanciare direttamente `wp-migrate pull` o `wp-migrate push` senza passare da `init`.

## Flusso

### `wp-migrate init`

- crea `.wp-migrate`
- salva la configurazione locale in `.wp-migrate/config` (repository remoto, URL remoto, branch)
- salva la configurazione locale della macchina in `.wp-migrate/config.local` (URL locale)
- inizializza un repository git locale dentro `.wp-migrate`
- collega il remote `origin`

### `wp-migrate push`

- esporta il database in un file temporaneo
- comprime `wp-content/uploads` in un archivio temporaneo
- salva database e archivio in `.wp-migrate` come file singoli oppure chunk `.part-XXXX` se superano circa 50 MB
- genera un manifest per ogni asset e rimuove eventuali chunk obsoleti da push precedenti
- calcola la prossima versione patch a partire dall'ultimo tag `vX.Y.Z` presente sul repository remoto (nessuno stato di versione viene salvato in config)
- crea commit e tag `vX.Y.Z`
- esegue push del branch e dei tag verso il repository remoto

### `wp-migrate pull`

- aggiorna il contenuto di `.wp-migrate` con `git fetch` + `git pull`
- ricostruisce una cache locale in `.wp-migrate/.cache` per gli asset splittati senza sporcare il repository git
- rigenera sempre `.wp-migrate/config` e `.wp-migrate/.gitignore` con i valori di questa macchina dopo la sincronizzazione, cosi da non essere mai bloccato da modifiche locali su questi file

### `wp-migrate sync`

- ricostruisce il database da `.wp-migrate/database.sql` o dai suoi chunk
- ricostruisce l'archivio uploads da `.wp-migrate/uploads.tar.gz` o dai suoi chunk
- sincronizza i file in `wp-content/uploads` con `rsync --delete`
- esegue `wp search-replace "<URL_REMOTO>" "<URL_LOCALE>"`

### `wp-migrate sync --db-only`

- importa solo il database
- esegue `wp search-replace "<URL_REMOTO>" "<URL_LOCALE>"`
- non modifica `wp-content/uploads`

### `wp-migrate import <file.sql|file.sql.gz>`

- importa direttamente un dump SQL esterno (non gestito da `.wp-migrate`), ad esempio uno scaricato manualmente in `./data`
- se il file termina in `.gz` lo decomprime in una cartella temporanea prima dell'import
- non esegue `search-replace`: utile per importare backup grezzi senza toccare la configurazione di `wp-migrate`
