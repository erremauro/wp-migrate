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
```

## Flusso

### `wp-migrate init`

- crea `.wp-migrate`
- salva la configurazione in `.wp-migrate/config`
- inizializza un repository git locale dentro `.wp-migrate`
- collega il remote `origin`

### `wp-migrate push`

- esporta il database in un file temporaneo
- comprime `wp-content/uploads` in un archivio temporaneo
- salva database e archivio in `.wp-migrate` come file singoli oppure chunk `.part-XXXX` se superano circa 50 MB
- genera un manifest per ogni asset e rimuove eventuali chunk obsoleti da push precedenti
- incrementa la versione patch in config
- crea commit e tag `vX.Y.Z`
- esegue push del branch e dei tag verso il repository remoto

### `wp-migrate pull`

- aggiorna il contenuto di `.wp-migrate` con `git fetch` + `git pull`
- ricostruisce una cache locale in `.wp-migrate/.cache` per gli asset splittati senza sporcare il repository git

### `wp-migrate sync`

- ricostruisce il database da `.wp-migrate/database.sql` o dai suoi chunk
- ricostruisce l'archivio uploads da `.wp-migrate/uploads.tar.gz` o dai suoi chunk
- sincronizza i file in `wp-content/uploads` con `rsync --delete`
- esegue `wp search-replace "<URL_REMOTO>" "<URL_LOCALE>"`

### `wp-migrate sync --db-only`

- importa solo il database
- esegue `wp search-replace "<URL_REMOTO>" "<URL_LOCALE>"`
- non modifica `wp-content/uploads`
