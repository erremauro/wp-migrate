# Changelog

Tutte le modifiche rilevanti a questo progetto saranno documentate in questo file.

Il formato è basato su [Keep a Changelog](https://keepachangelog.com/it-IT/1.1.0/),
e questo progetto aderisce a [Semantic Versioning](https://semver.org/lang/it/).

## [Unreleased]

## [0.2.0] - 2026-07-21

### Added

- Comando `wp-migrate import <file.sql|file.sql.gz>` per importare un dump SQL esterno (anche compresso) senza passare dal flusso `.wp-migrate`.
- Template `config.example` e `config.local.example` per configurare `.wp-migrate` manualmente, senza passare dal prompt interattivo di `init`.
- Supporto allo split automatico di dump database e archivio uploads in chunk compatibili con il limite GitHub da 100 MB.
- Ricostruzione trasparente degli asset splittati in fase di `pull` e `sync` tramite cache locale `.wp-migrate/.cache`.

### Changed

- `push` genera ora file singoli o chunk `.part-XXXX` con manifest dedicato, ripulendo gli artifact obsoleti prima del commit.
- I chunk generati per gli artifact di migrazione ora usano una dimensione massima di circa 50 MB, in linea con il limite consigliato da GitHub.
- La configurazione locale della macchina ora viene salvata in `.wp-migrate/config.local`, separata dalla configurazione condivisa versionata nel repository.
- `.wp-migrate/config` non viene piu committato nel repository git interno: e ora puramente locale, come `config.local`, e viene rigenerato ad ogni `pull` con i valori di quella macchina. `push` rimuove automaticamente `config`/`.gitignore` dal tracking se provengono da un repository creato con una versione precedente dello script.
- La versione `vX.Y.Z` non viene piu letta/scritta in `config`: `push` la calcola sempre dall'ultimo tag esistente sul repository remoto, cosi che macchine diverse non possano disallinearsi.

### Fixed

- `pull` su una nuova installazione locale inizializzata con `wp-migrate init` non fallisce piu per conflitto tra i file locali `config` e `.gitignore` e il primo checkout del branch remoto.
- `sync` non usa piu accidentalmente l'URL locale come valore sorgente del `search-replace` quando la configurazione locale viene inizializzata e poi allineata dal repository remoto.
- `pull` non fallisce piu con `error: Your local changes ... would be overwritten by merge` su `config`/`.gitignore`: essendo file puramente locali, ora vengono rimossi prima del checkout/merge e rigenerati subito dopo con i valori corretti della macchina.

## [0.1.0] - 2026-03-04

### Added

- CLI `wp-migrate` standalone in shell con i comandi `init`, `push`, `pull` e `sync`.
- Supporto alla configurazione locale tramite cartella `.wp-migrate`.
- Supporto a `WP_CLI_BIN` e `PHP_BIN` per ambienti come Local.

### Changed

- La sincronizzazione file ora riguarda solo `wp-content/uploads`.
- L'archivio dei file migrati ora usa il nome `.wp-migrate/uploads.tar.gz`.

### Notes

- La versionatura dei payload di migrazione avviene tramite repository git interno a `.wp-migrate` e tag `vX.Y.Z`.
