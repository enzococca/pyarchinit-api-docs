# PyArchInit API Documentation — Changelog

> Registro delle modifiche alla documentazione API generata automaticamente
> da `pyarchinit-code_dev/`.
>
> Format: bilingue IT/EN. Le date sono ISO 8601.

---

## [phase3-pgcompat] — 2026-05-11

### Italiano

**Aggiornamento doc API per Phase 3 — PostgreSQL compatibility refactor del s3dgraphy bridge** (pyarchinit tags `phase3-pgcompat-shim-5.6.2-alpha` -> `phase3-pgcompat-consolidation-5.7.4-alpha`).

Phase 3 ha eseguito il flipping completo di tutta la sync pipeline da `sqlite3.connect()` raw a SQLAlchemy + `DbHandle` shim. Sei tag rilasciati in 2 giorni (2026-05-10 -> 2026-05-11), zero residui `sqlite3.connect()` in `modules/s3dgraphy/sync/`. AC-2 byte-identical preservato dall'inizio alla fine.

**Tag rilasciati:**

| Milestone | Tag | Commit |
|---|---|---|
| Foundation | `phase3-pgcompat-shim-5.6.2-alpha` | `7420a6cc` |
| PG-A | `phase3-pgcompat-a-migration-5.7.0-alpha` | `45803d83` |
| PG-B | `phase3-pgcompat-b-export-5.7.1-alpha` | `2121369e` |
| PG-C | `phase3-pgcompat-c-import-5.7.2-alpha` | `cf6ed26e` |
| PG-D | `phase3-pgcompat-d-paradata-5.7.3-alpha` | `b8d73058` |
| Consolidation | `phase3-pgcompat-consolidation-5.7.4-alpha` | `2446b555` |

**Nuovi simboli pubblici (Phase 3):**

In `modules/s3dgraphy/sync/`:

| Simbolo | Modulo | Ruolo |
|---|---|---|
| `DbHandle` | `_db_handle.py` | Wrapper leggero attorno a un engine SQLAlchemy + `sqlite_path` opzionale |
| `_resolve_db_handle(arg)` | `_db_handle.py` | Shim che accetta `Path \| DbHandle \| str` e ritorna un `DbHandle`; entry point backwards-compatible per tutte le API di sync |
| `_conn_slug(handle)` | `_workspace.py` | Slug filesystem-safe per connessioni PG (formato `host_port_dbname`) |
| `_resolve_workspace_dir(handle, sito)` | `_workspace.py` | Ritorna la workspace dir per-sito: SQLite=`handle.sqlite_path.parent`, PG=`<root>/<conn_slug>/<sito>/` |
| `_resolve_workspace_root()` | `_workspace.py` | NUOVO in Consolidation: fallback a 3 livelli (env var `PYARCHINIT_WORKSPACE_DIR` > QSettings `pyarchinit/paradata_workspace` > default `~/pyarchinit/pyarchinit_DB_folder`) |
| `_DryRunRollback` | `graph_ingestor.py` | Sentinel exception per rollback dry-run in `populate_list` |
| `load_sqlite_into_pg(sqlite_path, engine)` | test fixture helper | Mirror di una fixture SQLite in uno schema PG per test cross-engine |

In `scripts/migrations/`:

| Simbolo | Modulo | Ruolo |
|---|---|---|
| `_columns_of(handle, table)` | `_2026_05_node_uuid_backfill_lib.py` | Listing colonne engine-agnostic che rimpiazza `PRAGMA table_info` |

In `gui/pyarchinitConfigDialog.py` (Consolidation):

| Elemento UI | Ruolo |
|---|---|
| Sezione "Paradata Workspace" (QGroupBox) | Nuova sezione nel tab DB Sync; legge/scrive QSettings `pyarchinit/paradata_workspace` con pulsanti Browse + Reset + signal `editingFinished` + `sync()` difensivo |

**Note sulla regenerazione api-docs:**

Due moduli nuovi sotto `modules/s3dgraphy/sync/` hanno nome con prefisso `_` (privato convenzionale):
- `_db_handle.py` (Foundation) — definisce la classe `DbHandle` e lo shim `_resolve_db_handle`
- `_workspace.py` (PG-D + Consolidation) — risoluzione path workspace

Entrambi NON sono attualmente nella nav di `mkdocs.yml` poiché la generatrice salta i moduli underscore-prefissati. Aggiungerli alla nav è deferito alla prossima rigenerazione API completa o a un follow-up doc commit dedicato; i loro docstring sono già aggiornati nel source upstream e popoleranno automaticamente le pagine `.md` generate.

**Decisioni di design Phase 3:**

- Foundation introduce `DbHandle` come thin wrapper: solo `engine` (SQLAlchemy) + `sqlite_path` opzionale (per back-compat su SQLite). Nessuna logica di dispatch nello shim — i call site possono scegliere se branch su `handle.is_postgres`.
- `db_path` keyword preservato su tutte le API pubbliche per back-compat (`Path | DbHandle | str` accepted); rename a `db_input` deferito a Phase 4 / 5.8.x con DeprecationWarning cycle.
- SQLite filesystem behavior byte-identical preservato attraverso ogni milestone (workspace dir per ParadataStore/GroupStore = `db_path.parent`, AC-2 byte-identical preservato).
- PG workspace separa per connessione + per sito (`<root>/<host_port_dbname>/<sito>/`); root configurabile via env var / QSettings / default in Consolidation.
- `_DryRunRollback` sentinel pattern in PG-C garantisce rollback completo del transazione in dry-run mode senza usare savepoint engine-specific.

### English

**API doc update for Phase 3 — s3dgraphy bridge PostgreSQL compatibility refactor** (pyarchinit tags `phase3-pgcompat-shim-5.6.2-alpha` -> `phase3-pgcompat-consolidation-5.7.4-alpha`).

Phase 3 completed the full flip of the sync pipeline from raw `sqlite3.connect()` to SQLAlchemy + `DbHandle` shim. Six tags shipped over 2 days (2026-05-10 -> 2026-05-11), zero residual `sqlite3.connect()` in `modules/s3dgraphy/sync/`. AC-2 byte-identical preserved end-to-end.

**Tags shipped:**

| Milestone | Tag | Commit |
|---|---|---|
| Foundation | `phase3-pgcompat-shim-5.6.2-alpha` | `7420a6cc` |
| PG-A | `phase3-pgcompat-a-migration-5.7.0-alpha` | `45803d83` |
| PG-B | `phase3-pgcompat-b-export-5.7.1-alpha` | `2121369e` |
| PG-C | `phase3-pgcompat-c-import-5.7.2-alpha` | `cf6ed26e` |
| PG-D | `phase3-pgcompat-d-paradata-5.7.3-alpha` | `b8d73058` |
| Consolidation | `phase3-pgcompat-consolidation-5.7.4-alpha` | `2446b555` |

**New public symbols (Phase 3):**

In `modules/s3dgraphy/sync/`:

| Symbol | Module | Role |
|---|---|---|
| `DbHandle` | `_db_handle.py` | Lightweight wrapper around a SQLAlchemy engine + optional `sqlite_path` |
| `_resolve_db_handle(arg)` | `_db_handle.py` | Shim that accepts `Path \| DbHandle \| str` and returns a `DbHandle`; backwards-compatible entry point for all sync APIs |
| `_conn_slug(handle)` | `_workspace.py` | Filesystem-safe slug for PG connections (format `host_port_dbname`) |
| `_resolve_workspace_dir(handle, sito)` | `_workspace.py` | Returns per-site workspace dir: SQLite=`handle.sqlite_path.parent`, PG=`<root>/<conn_slug>/<sito>/` |
| `_resolve_workspace_root()` | `_workspace.py` | NEW in Consolidation: 3-tier fallback (env var `PYARCHINIT_WORKSPACE_DIR` > QSettings `pyarchinit/paradata_workspace` > default `~/pyarchinit/pyarchinit_DB_folder`) |
| `_DryRunRollback` | `graph_ingestor.py` | Sentinel exception for dry-run rollback in `populate_list` |
| `load_sqlite_into_pg(sqlite_path, engine)` | test fixture helper | Mirrors a SQLite fixture into a PG schema for cross-engine tests |

In `scripts/migrations/`:

| Symbol | Module | Role |
|---|---|---|
| `_columns_of(handle, table)` | `_2026_05_node_uuid_backfill_lib.py` | Engine-agnostic column listing replacing `PRAGMA table_info` |

In `gui/pyarchinitConfigDialog.py` (Consolidation):

| UI element | Role |
|---|---|
| "Paradata Workspace" QGroupBox | New section in the DB Sync tab; reads/writes QSettings `pyarchinit/paradata_workspace` with Browse + Reset buttons + `editingFinished` signal + defensive `sync()` |

**Notes on api-docs regeneration:**

Two new modules under `modules/s3dgraphy/sync/` are named with a `_` prefix (conventional private modules):
- `_db_handle.py` (Foundation) — defines the `DbHandle` class and the `_resolve_db_handle` shim
- `_workspace.py` (PG-D + Consolidation) — workspace path resolution

Neither is currently in `mkdocs.yml` nav since the generator skips underscore-prefixed modules. Adding them to the nav is deferred to the next full API regen or a follow-up doc commit; their docstrings are already updated in the upstream source and will populate the auto-generated `.md` pages automatically.

**Phase 3 design decisions:**

- Foundation introduces `DbHandle` as a thin wrapper: just an `engine` (SQLAlchemy) + optional `sqlite_path` (for SQLite back-compat). No dispatch logic in the shim itself — call sites can choose whether to branch on `handle.is_postgres`.
- `db_path` keyword preserved across all public APIs for back-compat (`Path | DbHandle | str` accepted); rename to `db_input` deferred to Phase 4 / 5.8.x with DeprecationWarning cycle.
- SQLite filesystem behavior preserved byte-identically through every milestone (workspace dir for ParadataStore/GroupStore = `db_path.parent`, AC-2 byte-identical preserved).
- PG workspace splits per connection + per site (`<root>/<host_port_dbname>/<sito>/`); root is configurable via env var / QSettings / default in Consolidation.
- `_DryRunRollback` sentinel pattern in PG-C ensures full transaction rollback in dry-run mode without using engine-specific savepoints.

---

## [ai07-locationnodegroup] — 2026-05-10

### Italiano

**Aggiornamento doc API per AI07 — `LocationNodeGroup` migration + AI08-F1 m:n hierarchical** (tag pyarchinit `phase2-ai07-locationnodegroup-5.6.0-alpha`).

Rigenerati 14 file `.md` per i moduli sotto `modules/s3dgraphy/sync/` riflettendo i nuovi simboli AI07. Aggiornato `API_INDEX.md` con +38 classi e +49 funzioni rispetto allo snapshot 2026-05-09.

**Nuovi simboli pubblici (AI07):**

| Simbolo | Modulo | Ruolo |
|---|---|---|
| `compute_primary` | `graph_projector.py` | Selezione `is_primary` per US con membership multiple |
| `DEFAULT_PRIMARY_PRIORITY` | `graph_projector.py` | Ordine priorità default (struttura > attivita > area > ...) |
| `_emit_toponym_chain` | `graph_projector.py` | Stage 3: emette catena ricorsiva LocationNodeGroup(kind='toponym') da `site_table` |
| `_resolve_node_class_and_kind` | `group_projector.py` | Mapping pyarchinit dimension → (s3dgraphy class, kind enum) |
| `_GROUP_KIND_TO_S3D` | `group_projector.py` | Tabella di dispatch |
| `_promote_legacy_activitynodegroup` | `graph_ingestor.py` | Stage B re-import: up-conversion in-memory di file 5.5.x legacy |
| `SQL_BACKED_KINDS_SPATIAL` | `graph_ingestor.py` | Set delle 6 dimensioni spaziali |
| `_DIM_TO_KIND` | `graph_ingestor.py` | Mapping dim → LocationNodeGroup.kind (study/functional) |
| `CycleDetectedError` | `graph_ingestor.py` | Eccezione del walker ricorsivo |
| `_apply_group_folders_to_sql` | `graph_ingestor.py` | **Riscritto ricorsivo** (signature invariata) — descende in folder yEd nesting |
| `_resolve_group_visual` | `graphml_writer.py` | Lookup palette: dimension → kind enum → defaults |
| `_inject_other_locations_data` | `graphml_writer.py` | Stage 4f: emette `<data key="s3d:other_locations">` su nodi US |
| `_inject_other_locations_badges` | `graphml_writer.py` | Stage 4g: render badge inline yEd `<y:NodeLabel>` (sandwich position) |
| `_GROUP_KIND_PALETTE` | `graphml_writer.py` | **Esteso** con 3 chiavi kind (toponym/study/functional) oltre alle 8 dim |
| `KNOWN_EDGE_TYPES` | `edge_registry.py` | Frozenset delle edge type accettate (strutturali ∪ paradata) |
| `_STRUCTURAL_NON_PARADATA_EDGES` | `edge_registry.py` | Override per evitare classificazione errata di `is_in_location` |

**Decisioni di design Emanuel** (issue [zalmoxes-laran/s3Dgraphy#5](https://github.com/zalmoxes-laran/s3Dgraphy/issues/5)):
- Q1: `attivita` resta `ActivityNodeGroup` (non deprecato)
- Q2: la libreria 0.1.41 legge `ActivityNodeGroup + group_kind` come opaco; up-conversion vive nel projector di pyarchinit (consumer-side)
- Multi-projection georef: siblings paritari via P161
- Toponym chain: endorsed, dedupe cross-site con UUID deterministico

### English

**API doc update for AI07 — `LocationNodeGroup` migration + AI08-F1 m:n hierarchical** (pyarchinit tag `phase2-ai07-locationnodegroup-5.6.0-alpha`).

Regenerated 14 `.md` files for modules under `modules/s3dgraphy/sync/` reflecting the new AI07 symbols. Updated `API_INDEX.md` with +38 classes and +49 functions vs the 2026-05-09 snapshot.

**New public symbols (AI07):**

| Symbol | Module | Role |
|---|---|---|
| `compute_primary` | `graph_projector.py` | `is_primary` selection per US with multiple memberships |
| `DEFAULT_PRIMARY_PRIORITY` | `graph_projector.py` | Default priority order (struttura > attivita > area > ...) |
| `_emit_toponym_chain` | `graph_projector.py` | Stage 3: recursive LocationNodeGroup(kind='toponym') chain from `site_table` |
| `_resolve_node_class_and_kind` | `group_projector.py` | Mapping pyarchinit dimension → (s3dgraphy class, kind enum) |
| `_GROUP_KIND_TO_S3D` | `group_projector.py` | Dispatch table |
| `_promote_legacy_activitynodegroup` | `graph_ingestor.py` | Stage B re-import: in-memory up-conversion of legacy 5.5.x files |
| `SQL_BACKED_KINDS_SPATIAL` | `graph_ingestor.py` | Set of the 6 spatial dimensions |
| `_DIM_TO_KIND` | `graph_ingestor.py` | Mapping dim → LocationNodeGroup.kind (study/functional) |
| `CycleDetectedError` | `graph_ingestor.py` | Recursive walker exception |
| `_apply_group_folders_to_sql` | `graph_ingestor.py` | **Rewritten recursive** (signature unchanged) — descends into yEd folder nesting |
| `_resolve_group_visual` | `graphml_writer.py` | Palette lookup: dimension → kind enum → defaults |
| `_inject_other_locations_data` | `graphml_writer.py` | Stage 4f: emits `<data key="s3d:other_locations">` on US nodes |
| `_inject_other_locations_badges` | `graphml_writer.py` | Stage 4g: inline yEd `<y:NodeLabel>` badges (sandwich position) |
| `_GROUP_KIND_PALETTE` | `graphml_writer.py` | **Extended** with 3 kind keys (toponym/study/functional) on top of the 8 dim keys |
| `KNOWN_EDGE_TYPES` | `edge_registry.py` | Frozenset of accepted edge types (structural ∪ paradata) |
| `_STRUCTURAL_NON_PARADATA_EDGES` | `edge_registry.py` | Override preventing misclassification of `is_in_location` |

**Emanuel's design decisions** (issue [zalmoxes-laran/s3Dgraphy#5](https://github.com/zalmoxes-laran/s3Dgraphy/issues/5)):
- Q1: `attivita` stays as `ActivityNodeGroup` (not deprecated)
- Q2: library 0.1.41 reads `ActivityNodeGroup + group_kind` as opaque; up-conversion lives in pyarchinit's projector (consumer-side)
- Multi-projection georef: paritary siblings via P161
- Toponym chain: endorsed, cross-site dedupe with deterministic UUID

---

## [s3dgraphy-sync-bridge] — 2026-05-09

### Italiano

**Aggiunta documentazione API per il bridge bidirezionale `pyarchinit ↔ s3dgraphy` (Phase 2: AI01 → AI08-F2).**

15 nuovi file `.md` generati via AST a partire da docstrings + signature dei moduli:

| Modulo | Elementi documentati | Ruolo |
|---|---|---|
| `modules/s3dgraphy/sync/__init__.py` | 1 | Public API + lazy `get_vocab_provider()` |
| `modules/s3dgraphy/sync/vocab_types.py` | 6 | Dataclasses: `EdgeType`, `Family`, `ParadataType`, `UnitType`, `VisualRule`, `VocabularyVersion` |
| `modules/s3dgraphy/sync/vocab_provider_core.py` | 15 | Parser dei pillars JSON di s3dgraphy + API di query |
| `modules/s3dgraphy/sync/vocab_provider.py` | 13 | Singleton Qt-aware con hot-reload |
| `modules/s3dgraphy/sync/uuid7.py` | 1 | Generatore UUID v7 monotonico (RFC 9562) |
| `modules/s3dgraphy/sync/edge_registry.py` | 4 | Stili archi EM 1.5 + classificazione paradata |
| `modules/s3dgraphy/sync/graph_projector.py` | 10 | Proiezione SQL → `s3dgraphy.Graph` (read-side) |
| `modules/s3dgraphy/sync/graph_ingestor.py` | 22 | Ingest GraphML → SQL atomico (write-side) |
| `modules/s3dgraphy/sync/graphml_writer.py` | 19 | Pipeline GraphML completa + per-dim visual style + folder injection |
| `modules/s3dgraphy/sync/paradata_store.py` | 25 | CRUD site-scoped per `paradata_<sito>.graphml` |
| `modules/s3dgraphy/sync/group_store.py` | 25 | CRUD site-scoped per `groups_<sito>.graphml` |
| `modules/s3dgraphy/sync/group_projector.py` | 4 | Specifiche di gruppo derivate da SQL + ad-hoc merge |
| `modules/s3dgraphy/sync/conflict_resolver.py` | 2 | Stub AI04 (sempre `GRAPH_WINS`); promozione in AI06+ |
| `modules/s3dgraphy/sync/ingest_result.py` | 3 | Risultato aggregato + record di conflitto |
| `modules/s3dgraphy/sync/_legacy_paradata_svgs.py` | 0 | SVG legacy (no-docstring) |

**Aggiornati anche gli indici principali:**

- `API_INDEX.md`:
  - +37 righe nella tabella **Classes** (inserite alfabeticamente).
    Principali: `GraphProjector`, `GraphIngestor`, `ParadataStore`, `GroupStore`, `VocabProvider`, `VocabProviderCore`, `IngestResult`, `ConflictResolver`, `GraphMLExportError`, plus 28 dataclasses/exceptions correlate.
  - +42 righe nella tabella **Functions** (alfabetiche). Include funzioni private dell'API interna (`_apply_group_folders_to_sql`, `_inject_group_folders`, `_propagate_node_uuid_and_us`, ecc.) per consultabilità durante review.

- `README.md`:
  - **Project Statistics** aggiornate: 543→558 file analizzati, 597→634 classi, 776→818 funzioni, 5157→5228 metodi.
  - 15 nuovi link nella sezione **Documentation** dopo l'anchor `matrix_graph_visualizer.py`.

**Tooling:** generato via `/tmp/gen_s3d_apidocs.py` (parsing AST + matching del template) e `/tmp/update_apiindex.py` (insert alfabetico + bump statistiche). Riproducibile: lanciando di nuovo gli script i file vengono rigenerati senza duplicati.

**Note di scope:** la documentazione API è **fuori dalla working tree git** del plugin (`/Users/enzo/Downloads/pyarchinit-code_dev/`), e non viene pushata su GitHub. Riflette i moduli implementati fino al tag `housekeeping-2026-05-09` (HEAD `c6a06a8c`).

### English

**Added API documentation for the bidirectional `pyarchinit ↔ s3dgraphy` bridge (Phase 2: AI01 → AI08-F2).**

15 new `.md` files generated via AST from each module's docstrings + signatures:

| Module | Documented elements | Role |
|---|---|---|
| `modules/s3dgraphy/sync/__init__.py` | 1 | Public API + lazy `get_vocab_provider()` |
| `modules/s3dgraphy/sync/vocab_types.py` | 6 | Dataclasses: `EdgeType`, `Family`, `ParadataType`, `UnitType`, `VisualRule`, `VocabularyVersion` |
| `modules/s3dgraphy/sync/vocab_provider_core.py` | 15 | s3dgraphy JSON pillars parser + query API |
| `modules/s3dgraphy/sync/vocab_provider.py` | 13 | Qt-aware singleton with hot-reload |
| `modules/s3dgraphy/sync/uuid7.py` | 1 | Monotonic UUID v7 generator (RFC 9562) |
| `modules/s3dgraphy/sync/edge_registry.py` | 4 | EM 1.5 edge styles + paradata classification |
| `modules/s3dgraphy/sync/graph_projector.py` | 10 | SQL → `s3dgraphy.Graph` projection (read-side) |
| `modules/s3dgraphy/sync/graph_ingestor.py` | 22 | Atomic GraphML → SQL ingest (write-side) |
| `modules/s3dgraphy/sync/graphml_writer.py` | 19 | Full GraphML pipeline + per-dim visual style + folder injection |
| `modules/s3dgraphy/sync/paradata_store.py` | 25 | Site-scoped CRUD for `paradata_<sito>.graphml` |
| `modules/s3dgraphy/sync/group_store.py` | 25 | Site-scoped CRUD for `groups_<sito>.graphml` |
| `modules/s3dgraphy/sync/group_projector.py` | 4 | SQL-derived group specs + ad-hoc merge |
| `modules/s3dgraphy/sync/conflict_resolver.py` | 2 | AI04 stub (always `GRAPH_WINS`); promoted in AI06+ |
| `modules/s3dgraphy/sync/ingest_result.py` | 3 | Aggregated result + conflict record |
| `modules/s3dgraphy/sync/_legacy_paradata_svgs.py` | 0 | Legacy SVGs (no docstrings) |

**Top-level indexes also updated:**

- `API_INDEX.md`:
  - +37 rows in the **Classes** table (alphabetically inserted).
    Highlights: `GraphProjector`, `GraphIngestor`, `ParadataStore`, `GroupStore`, `VocabProvider`, `VocabProviderCore`, `IngestResult`, `ConflictResolver`, `GraphMLExportError`, plus 28 related dataclasses/exceptions.
  - +42 rows in the **Functions** table (alphabetical). Includes internal-API private helpers (`_apply_group_folders_to_sql`, `_inject_group_folders`, `_propagate_node_uuid_and_us`, etc.) for review-time greppability.

- `README.md`:
  - **Project Statistics** bumped: 543→558 files, 597→634 classes, 776→818 functions, 5157→5228 methods.
  - 15 new links in the **Documentation** section, inserted after the `matrix_graph_visualizer.py` anchor.

**Tooling:** generated via `/tmp/gen_s3d_apidocs.py` (AST parsing + template matching) and `/tmp/update_apiindex.py` (alphabetical insert + stats bump). Reproducible: re-running the scripts regenerates the files without duplicates.

**Scope note:** the API documentation lives **outside the plugin git working tree** (`/Users/enzo/Downloads/pyarchinit-code_dev/`) and is not pushed to GitHub. It reflects modules implemented up to tag `housekeeping-2026-05-09` (HEAD `c6a06a8c`).

---

## [baseline] — 2026-05-01

### Italiano

Documentazione API generata inizialmente per `pyarchinit` (master + Stratigraph_00001 fino al tag `phase2-ai03-graphml-delegation-5.2.0-alpha`):

- 543 file Python analizzati
- 597 classi documentate (5157 metodi)
- 776 funzioni top-level
- File principali: `API_INDEX.md`, `API_REFERENCE.md`, `CLASS_DIAGRAM.md`, `README.md`, `index.md`, `conf.py` (config Sphinx) + `_build/` (output rendered).

### English

Initial API documentation generated for `pyarchinit` (master + Stratigraph_00001 up to tag `phase2-ai03-graphml-delegation-5.2.0-alpha`):

- 543 Python files analyzed
- 597 classes documented (5157 methods)
- 776 top-level functions
- Top-level files: `API_INDEX.md`, `API_REFERENCE.md`, `CLASS_DIAGRAM.md`, `README.md`, `index.md`, `conf.py` (Sphinx config) + `_build/` (rendered output).
