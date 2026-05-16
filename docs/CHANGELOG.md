# PyArchInit API Documentation — Changelog

> Registro delle modifiche alla documentazione API generata automaticamente
> da `pyarchinit-code_dev/`.
>
> Format: bilingue IT/EN. Le date sono ISO 8601.

---

## [yed-f-fix-duplicate-primary] — 2026-05-16

### Italiano

**Hotfix yE-F duplicate-primary** (pyarchinit tag `yed-f-fix-duplicate-primary-5.9.0.1-alpha`, commit `1653363c` + 1 follow-up PG conftest). Predecessore: `yed-f-multifolder-5.9.0-alpha`.

**Bug**: su import yEd graphml reali con paradata multi-folder (es. `DOC.01` referenced da 5 folders `VA01..VA05`), il primary `attivita` veniva sovrascritto dall'ULTIMO folder iterato, e il valore primary risultava duplicato dentro `other_locations`.

**Root cause**: `_apply_yed_folder_dimensions` (`modules/s3dgraphy/sync/yed_import_pipeline.py:911-995`) itera ogni folder ed esegue `UPDATE us_table SET <dim>=<value> WHERE node_uuid IN (...)`. Per le row paradata fold (N occorrenze yEd → 1 us_table row → 1 node_uuid condiviso), ogni `UPDATE` per-folder colpisce la stessa row; vince l'ultima iterazione.

**Fix**: quando `dim == "attivita"`, filtra fuori i node_uuid paradata dall'`UPDATE` di `_apply_yed_folder_dimensions` — yE-F ha già scritto il primary corretto (primo folder in document order) durante l'INSERT fold in `_write_us_rows`. Il SELECT pre-filtro usa `unita_tipo IN ('DOC','Combinar','Extractor','property')` per identificare le row paradata.

**Simboli modificati**:

- `modules/s3dgraphy/sync/yed_import_pipeline.py`:
  - NEW `_PARADATA_NODEDUP_UTS: frozenset[str]` promosso a module-level (era local in `_write_us_rows`).
  - `_apply_yed_folder_dimensions(conn, folders, sito, uuid_map)` — aggiunto paradata-skip pre-filtro per `dim == "attivita"` (12 nuove righe, ~966-978).
- `tests/sync/test_yef_fold.py`:
  - NEW test `test_apply_yed_folder_dimensions_skips_paradata_attivita` — regression: attivita resta sul primo folder document-order e non duplicata in other_locations.
- `tests/sync/conftest_pg.py`:
  - PG `us_table` DDL esteso con colonna `other_locations TEXT` per parità PG-online con SQLite.

**Test delta**: 351 → **352 sync test passed** / 35 skipped (PG offline) / 0 failed.

**PG portability**: fix puro SQL portabile — `text()` + named bind params, nessuna sintassi PG-specific o SQLite-specific. Funziona su entrambi i backend. DDL conftest PG aggiornato per copertura PG-online completa.

**AC-2 byte-identical baseline**: preservato (resolver attiva solo quando il fan-out è stato eseguito; questo fix tocca solo l'`UPDATE` path import-time).

### English

**yE-F duplicate-primary hotfix** (pyarchinit tag `yed-f-fix-duplicate-primary-5.9.0.1-alpha`, commit `1653363c` + 1 follow-up PG conftest update). Predecessor: `yed-f-multifolder-5.9.0-alpha`.

**Bug**: on real yEd graphml imports with multi-folder paradata (e.g. `DOC.01` appearing in 5 folders `VA01..VA05`), the primary `attivita` was overwritten by the LAST iterated folder, and the primary value was duplicated inside `other_locations`.

**Root cause**: `_apply_yed_folder_dimensions` (`modules/s3dgraphy/sync/yed_import_pipeline.py:911-995`) iterates every folder and runs `UPDATE us_table SET <dim>=<value> WHERE node_uuid IN (...)`. For paradata fold rows (N yEd occurrences → 1 us_table row → 1 shared node_uuid), every folder's `UPDATE` hits the same row; the last iteration wins.

**Fix**: when `dim == "attivita"`, filter out paradata node_uuids from the `_apply_yed_folder_dimensions` `UPDATE` — yE-F has already written the correct primary (first folder in document order) during the `_write_us_rows` fold INSERT. The new SELECT pre-filter uses `unita_tipo IN ('DOC','Combinar','Extractor','property')` to identify paradata rows.

**Symbols changed**:

- `modules/s3dgraphy/sync/yed_import_pipeline.py`:
  - NEW `_PARADATA_NODEDUP_UTS: frozenset[str]` promoted to module-level (was a local in `_write_us_rows`).
  - `_apply_yed_folder_dimensions(conn, folders, sito, uuid_map)` — added paradata-skip pre-filter for `dim == "attivita"` (12 new lines, ~966-978).
- `tests/sync/test_yef_fold.py`:
  - NEW test `test_apply_yed_folder_dimensions_skips_paradata_attivita` — regression: attivita stays at first-document-order folder and is not duplicated in other_locations.
- `tests/sync/conftest_pg.py`:
  - PG `us_table` DDL extended with `other_locations TEXT` column for PG-online parity with SQLite.

**Test delta**: 351 → **352 sync tests passed** / 35 skipped (PG offline) / 0 failed.

**PG portability**: fix is pure portable SQL — `text()` + named bind params, no PG-specific or SQLite-specific syntax. Works on both backends. PG conftest DDL updated for full PG-online coverage.

**AC-2 byte-identical baseline**: preserved (resolver only activates when fan-out has run; this fix only touches the import-time `UPDATE` path).

---

## [yed-f-multifolder] — 2026-05-16

### Italiano

**yE-F multi-folder paradata** (pyarchinit tag `yed-f-multifolder-5.9.0-alpha`, commit `83d82f40` su `Stratigraph_00001`). Recupero dell'identity-dedup per i tipi paradata persa in `yed-fastfix-5.8.5-alpha` (Bug R), via single us_table row + colonna `other_locations` (JSON list) + render-time fan-out N copie visive nel graphml ri-esportato. Suite pyarchinit 312 → **351 passed / 35 skipped / 0 failed** (+39 regression test). AC-2 byte-identical preservato (3 AC-2 test ancora verdi).

**Design pattern (fold ↔ fan-out)**:

- **Import (fold)**: cross-folder occurrences di un paradata label (`material`, `D.01`, `position` referenced da folder A + folder B + folder C) sono collassate in **1 sola** us_table row. Colonna `other_locations` (Text JSON) memorizza la lista dei folder secondari oltre al primary (`attivita`). Idempotency con 2-tier degradation: `paradata_primary_by_label` lookup, fallback su DB pre-migration senza `other_locations`.
- **Export (fan-out)**: `_apply_yef_fan_out(graph)` emette **N copie visive** della stessa row paradata (una per ogni folder in primary ∪ other_locations), clonando il node con `_clone_node_for_location()` e stashing `_yef_copies_by_canonical` su graph. Il graphml ri-esportato preserva la presenza cross-folder originale yEd.
- **Edge resolver per-folder**: `_resolve_target_for_folder(target_canonical_node, source_folder, graph)` in `graph_projector._enrich_into` rapporti loop. Risolve il target edge al cloned node giusto in base al folder sorgente, BEFORE family-preference fallback. AC-2 byte-identical preserved quando il graph non ha `_yef_copies_by_canonical` (path legacy).

**Tag rilasciato:**

| Milestone | Tag | Commit |
|---|---|---|
| yE-F Multi-folder paradata | `yed-f-multifolder-5.9.0-alpha` | `83d82f40` |

**Nuovi simboli pubblici (yE-F):**

In `modules/s3dgraphy/sync/yed_import_pipeline.py`:

| Simbolo | Ruolo |
|---|---|
| `_resolve_folder_for_leaf(yed_id, folders) -> str \| None` | Helper module-level: dato un `yed_id` di leaf e la lista di `FolderCandidate`, ritorna il folder dimension value associato (o `None` se non in folder). |

`_write_us_rows` modificato:
- Sostituito branch `_PARADATA_NODEDUP_UTS` no-dedup (Bug R) con yE-F fold (1 row per unique paradata label + `other_locations` JSON).
- Pre-load loop esteso per caricare `paradata_primary_by_label` ai fini idempotency su re-import, con 2-tier degradation per DB non-migrati.

In `modules/s3dgraphy/sync/graphml_writer.py`:

| Simbolo | Ruolo |
|---|---|
| `_clone_node_for_location(primary_node, location, idx, canonical_uuid) -> StratigraphicUnit` | Clona un node primary in una copia visiva per la location specifica; preserva `canonical_uuid` per traceability. |
| `_apply_yef_fan_out(graph) -> int` | Orchestrator: emette N copie visive per ogni multi-folder paradata row, stash `_yef_copies_by_canonical` su graph, ritorna count copie create. |

`export_graphml` modificato: chiama `_apply_yef_fan_out(graph)` tra `populate_graph` e `_filter_by_site`.

In `modules/s3dgraphy/sync/graph_projector.py`:

| Simbolo | Ruolo |
|---|---|
| `_resolve_target_for_folder(target_canonical_node, source_folder, graph)` | Edge resolver per-folder: data una source folder e un target canonical node, ritorna il cloned node corretto (o il canonical se non yE-F). |

`_enrich_into` rapporti loop modificato: chiama `_resolve_target_for_folder` BEFORE family-preference fallback; preserva AC-2 byte-identical quando il graph non ha `_yef_copies_by_canonical`.

In `modules/db/structures/US_table.py`:

| Cambio | Tag |
|---|---|
| Aggiunto `Column('other_locations', Text)` alla us_table declaration | `yed-f-multifolder-5.9.0-alpha` |

In `scripts/migrations/` (NUOVO):

| File | Ruolo |
|---|---|
| `_2026_05_yef_other_locations_lib.py` | `add_other_locations_column(handle) -> int` — idempotent ALTER TABLE ADD COLUMN. Supporta SQLite + PG via DbHandle. |
| `2026_05_yef_other_locations.py` | CLI argparse con `--apply`/`--dry-run` + `--db`/`--conn-str` mutex (pattern PG-A). |

In `pyarchinitPlugin.py`:

| Simbolo | Ruolo |
|---|---|
| QAction "Migrazioni → Aggiungi colonna other_locations (yE-F)" | Menu entry per migration GUI. |
| `_run_yef_migration(self)` | Handler: file picker + confirm dialog + auto-backup + chiamata `add_other_locations_column` + result dialog. |

In `tabs/US_USM.py`:

| Simbolo | Ruolo |
|---|---|
| `_populate_other_locations_logic(widget, db_rows, current_ol_json)` | Module-level: popola `listWidget_other_locations` da DISTINCT attivita query + selectiona items in current_ol_json. |
| `_save_other_locations_logic(widget, current_attivita) -> str \| None` | Module-level: raccoglie selezioni dal widget, esclude current_attivita (sta nel primary `attivita` field), serializza in JSON. |
| `_yef_widget_visible_for_unita_tipo(unita_tipo) -> bool` | Predicate: ritorna `True` per `unita_tipo ∈ _YEF_PARADATA_UTS`. |
| `_YEF_PARADATA_UTS` (frozenset) | Set dei 4 unita_tipo paradata (`DOC`, `Combinar`, `Extractor`, `property`). |

`fill_fields` modificato: popola `listWidget_other_locations` da DISTINCT attivita query + setta visibility iniziale per `unita_tipo`.
`change_label` (slot per `comboBox_unita_tipo`) modificato: toggla widget visibility ad ogni cambio di `unita_tipo`.
`update_record` modificato: side-channel UPDATE su `other_locations` column dopo il main `DB_MANAGER.update`.

In `gui/ui/US_USM.ui`:

| Widget | Ruolo |
|---|---|
| `label_other_locations` (QLabel) | Label visibile sopra il list widget (visible solo per paradata UT). |
| `listWidget_other_locations` (QListWidget) | MultiSelection mode, maxHeight 120px, in tab_2 gridLayout_15. |

In `modules/utility/pyarchinit_i18n_stratigraphic.py`:

| Simbolo | Ruolo |
|---|---|
| `_OTHER_LOCATIONS_LABEL` (dict, 10 langs) | Traduzioni label per `it/en/de/es/fr/ar/ca/ro/pt/el`. |
| `get_other_locations_label(lang)` | Getter con fallback su `en`. |

**Test delta**: 312 → 351 (+39 test). Nuovi file di test:

| File | Tests | Coverage |
|---|---|---|
| `test_yef_fold.py` | 6 | Fold cross-folder occurrences → 1 row + other_locations JSON |
| `test_yef_fanout.py` | 4 | Fan-out N visual copies in export graphml |
| `test_yef_edge_resolver.py` | 3 | Per-folder edge target resolution + AC-2 preservation no-yE-F path |
| `test_yef_ui_widget.py` | 4 | Widget visibility toggle + save/load logic |
| `test_yef_roundtrip.py` | 1 | End-to-end yEd import → DB → graphml export round-trip |
| `test_yef_migration.py` | 4 | Migration idempotency + dry-run + SQLite + PG via fixtures |

Più 1 test rinamato in `test_yed_import_pipeline.py` + DDL fixture updates. 0 regression, AC-2 byte-identical preservato.

**Predecessore**: `[yed-fastfix]` (2026-05-16, commit `a5e8502b`). Il fastfix 19 bug aveva scelto multi-folder visibility scartando identity-dedup; yE-F le riconcilia entrambe via fold-on-import + fan-out-on-export.

### English

**yE-F multi-folder paradata** (pyarchinit tag `yed-f-multifolder-5.9.0-alpha`, commit `83d82f40` on `Stratigraph_00001`). Recovers the identity-dedup for paradata kinds lost in `yed-fastfix-5.8.5-alpha` (Bug R), via single us_table row + `other_locations` (Text JSON list) column + render-time fan-out of N visual copies in the re-exported graphml. pyarchinit suite 312 → **351 passed / 35 skipped / 0 failed** (+39 regression tests). AC-2 byte-identical preserved (3 AC-2 tests still green).

**Design pattern (fold ↔ fan-out)**:

- **Import (fold)**: cross-folder occurrences of a paradata label (`material`, `D.01`, `position` referenced from folder A + folder B + folder C) are collapsed into **a single** us_table row. The `other_locations` column (Text JSON) stores the list of secondary folders beyond the primary (`attivita`). Idempotency with 2-tier degradation: `paradata_primary_by_label` lookup, fallback for pre-migration DBs lacking `other_locations`.
- **Export (fan-out)**: `_apply_yef_fan_out(graph)` emits **N visual copies** of the same paradata row (one per folder in primary ∪ other_locations), cloning the node via `_clone_node_for_location()` and stashing `_yef_copies_by_canonical` on the graph. The re-exported graphml preserves the original yEd cross-folder presence.
- **Per-folder edge resolver**: `_resolve_target_for_folder(target_canonical_node, source_folder, graph)` in `graph_projector._enrich_into` rapporti loop. Resolves the edge target to the correct cloned node based on source folder, BEFORE family-preference fallback. AC-2 byte-identical preserved when the graph has no `_yef_copies_by_canonical` (legacy path).

**Tag shipped:**

| Milestone | Tag | Commit |
|---|---|---|
| yE-F Multi-folder paradata | `yed-f-multifolder-5.9.0-alpha` | `83d82f40` |

**Public symbols added (yE-F)** (see Italian tables above for full descriptions):

- `modules/s3dgraphy/sync/yed_import_pipeline.py`: `_resolve_folder_for_leaf(yed_id, folders) -> str | None` module-level helper. `_write_us_rows` modified: Bug R no-dedup branch replaced with yE-F fold (1 row per unique label + `other_locations` JSON) + pre-load `paradata_primary_by_label` with 2-tier degradation.
- `modules/s3dgraphy/sync/graphml_writer.py`: `_clone_node_for_location(primary_node, location, idx, canonical_uuid) -> StratigraphicUnit` + `_apply_yef_fan_out(graph) -> int` orchestrator. `export_graphml` calls fan-out between `populate_graph` and `_filter_by_site`.
- `modules/s3dgraphy/sync/graph_projector.py`: `_resolve_target_for_folder(target_canonical_node, source_folder, graph)` per-folder edge resolver. `_enrich_into` rapporti loop modified to call it BEFORE family-preference fallback (AC-2 byte-identical preserved when no `_yef_copies_by_canonical`).
- `modules/db/structures/US_table.py`: `Column('other_locations', Text)` added to us_table declaration.
- `scripts/migrations/_2026_05_yef_other_locations_lib.py` (NEW): `add_other_locations_column(handle) -> int` idempotent ALTER TABLE ADD COLUMN (SQLite + PG via DbHandle).
- `scripts/migrations/2026_05_yef_other_locations.py` (NEW): argparse CLI with `--apply`/`--dry-run` + `--db`/`--conn-str` mutex (PG-A pattern).
- `pyarchinitPlugin.py`: new menu QAction "Migrazioni → Aggiungi colonna other_locations (yE-F)" + handler `_run_yef_migration` (file picker + confirm + auto-backup + result dialog).
- `tabs/US_USM.py`: `_populate_other_locations_logic`, `_save_other_locations_logic`, `_yef_widget_visible_for_unita_tipo`, `_YEF_PARADATA_UTS` frozenset (4 paradata UTs: `DOC`, `Combinar`, `Extractor`, `property`). `fill_fields` / `change_label` / `update_record` modified accordingly.
- `gui/ui/US_USM.ui`: new `label_other_locations` (QLabel) + `listWidget_other_locations` (QListWidget, MultiSelection, maxHeight 120px) in `tab_2` `gridLayout_15`.
- `modules/utility/pyarchinit_i18n_stratigraphic.py`: `_OTHER_LOCATIONS_LABEL` dict (10 langs) + `get_other_locations_label(lang)` getter with `en` fallback.

**Test delta**: 312 → 351 (+39 tests) across 6 new test files (`test_yef_fold.py` 6, `test_yef_fanout.py` 4, `test_yef_edge_resolver.py` 3, `test_yef_ui_widget.py` 4, `test_yef_roundtrip.py` 1, `test_yef_migration.py` 4) + 1 renamed in `test_yed_import_pipeline.py` + DDL fixture updates. 0 regressions, AC-2 byte-identical preserved.

**Predecessor**: `[yed-fastfix]` (2026-05-16, commit `a5e8502b`). The 19-bug fastfix had chosen multi-folder visibility over identity-dedup; yE-F reconciles both via fold-on-import + fan-out-on-export.

---

## [yed-fastfix] — 2026-05-16

### Italiano

**Hotfix progressivi yE-D batch A→T** (pyarchinit tag `yed-fastfix-5.8.5-alpha`, commit `a5e8502b` su `Stratigraph_00001`). 19 bug emergi durante manual test su `pyarchinit_test{002..010}.sqlite` — single commit cumulativo dopo iterazioni `cartella senza nome 7→14`. Suite pyarchinit 311 → 354 → **329 dopo refactor** (+18 regression test). AC-2 byte-identical preservato.

**Trade-off principale (Bug R)**: multi-folder visibility > identity-dedup per i tipi paradata. `material`, `D.01`, `position` referenced cross-folder ora producono N us_table rows (una per occurrence yEd, con `us` suffisso `_2`/`_3`, e `d_stratigrafica` = label originale) — restaurato il visual fan-out in yEd ri-esportato. Identity-dedup di varianti tipo `D.001` / `D.001-2` / `D.001bis` collassanti perso; recupero futuro via yE-F design (single row + `other_locations` + render-time fan-out).

**Tag rilasciato:**

| Milestone | Tag | Commit |
|---|---|---|
| yE-D Fastfix batch A→T | `yed-fastfix-5.8.5-alpha` | `a5e8502b` |

(Versioning: `5.8.4-alpha` riservato a dry-run interno tra Bug E e Bug R; rilascio pubblico salta a `5.8.5-alpha`.)

**19 bug fixed** (raggruppati per modulo):

In `modules/s3dgraphy/sync/yed_import_pipeline.py`:

| ID | Sintesi |
|---|---|
| **A** | `_write_rapporti`: rapporti tuple format corretto `[type, us_target, area, sito]` (posizioni 1 e 3 erano invertite, reader pyarchinit non risolveva mai i target). |
| **B** | `_write_us_rows`: nuovo arg `member_to_period`; `periodo_iniziale`/`fase_iniziale` propagati su us_table dalla `PeriodCandidate.member_yed_ids` parsata. `period_finale`/`fase_finale` settati pari (MVP single-period US). |
| **C** | `_write_periodizzazione_rows`: aggiunta colonna `cont_per` (UI "Codice periodo") all'INSERT con valore = `periodo_i` sequenza. |
| **E** | `_strip_unita_tipo_prefix(label, unita_tipo)` helper: `us_table.us` stripped del prefisso (`US100` → `'100'`, `USV200` → `'200'`, `USVs5` → `'5'`). SF/VSF/RSF aggiunti a `_SQL_US_KINDS` (was inventario-only); `_DUAL_WRITE_INV_KINDS = {SF, VSF, RSF}` nuovo set per dual-write us_table + inventario_materiali. |
| **F** | Token rapporti via `_select_rapporti_label(edge_type, src_ut, tgt_ut)` (riusato da `graph_ingestor`): verbose IT (`Copre`/`Coperto da`/`Taglia`/...) per US-USM-USM, `>>`/`<<` per altri non-canonici, `>`/`<` per Continuity. |
| **G** | `_PARADATA_KINDS = frozenset()` empty; DOC/Combinar/Extractor/property → us_table records con `unita_tipo='DOC'`/`'Combinar'`/`'Extractor'`/`'property'` (convenzione pyarchinit verificata su `pyarchinit_db.sqlite` produzione). No più scrittura `paradata_<sito>.graphml` separato per row-paradata. |
| **H** | Pre-load existing `(us, unita_tipo)` keys (e `numero_inventario`, `(periodo, fase)`) per skip-if-exists. Re-import = no-op senza UNIQUE-collision rollback. Defensive code-path preservato per integrity errors veri. |
| **M** | `_write_rapporti` default `edge_type='generic_connection'` quando uno degli endpoint è paradata (`overlies` non valido US↔Document per s3dgraphy connection rules). Cross-kind edges emettono `>>` direttamente, niente più warning + auto-demote a generic_connection. |
| **R** | `_PARADATA_NODEDUP_UTS = {DOC, Combinar, Extractor, property}`: skippa dedup-by-identity per queste 4 famiglie. Counter `paradata_seq[(unita_tipo, base_us)] → int` per suffix incrementale (`material`, `material_2`, `material_3`). Original label in `d_stratigrafica`. Idempotency tramite `existing_paradata: dict[(label, ut), list[(us, node_uuid)]]` consumato in ordine sui re-import. |
| **S** | `_write_us_rows` ritorna nuovo dict out `us_by_yed_id_out: dict[yed_id, str]` con il us value ATTUALE scritto (post-suffix). `import_yed_raw` lo usa per costruire `id_to_label` → rapporti target risolvono al per-occurrence us value (`01_2` invece di `01` condiviso). |
| **T** | `_write_rapporti` con `_INVERSE_TOKEN` map e accumulator `rapporti_by_node`: per ogni edge yEd `a→b` scrive forward su a's row E inverse (`<<`, `Coperto da`, etc.) su b's row. DOCs ora hanno rapporti entranti visibili nel form Scheda US. Reader pyarchinit (`graph_projector._enrich_into:706+`) dedupa via stable edge-id quindi reciprocità è zero-cost in rappresentazione grafica. |

In `modules/s3dgraphy/sync/yed_classifier.py`:

| ID | Sintesi |
|---|---|
| **D** | `ClassificationKind.EXTRACTOR = "extractor"` aggiunto al 14° enum value; regex `^E\.\d+` matcha pattern Extractor canonical EM. |
| **I** | `_detect_bpmn_kind(node_element, Y_NS)` helper: legge `<y:Property name="...dataObjectType" value="DATA_OBJECT_TYPE_PLAIN"/>` → DOCUMENT, `name="...type" value="ARTIFACT_TYPE_ANNOTATION"` → PROPERTY. BPMN signal vince sul label fallback. Risolve `D.NN` (BPMN data-object) → DOCUMENT vs `D.NN.MM` (plain) → EXTRACTOR. Senza, dedup pre-Bug-R collassava entrambi nella stessa riga, drop edges Extractor→Document. |

In `modules/s3dgraphy/sync/graph_projector.py`:

| ID | Sintesi |
|---|---|
| **K** | `_PARADATA_UNITA_TIPO_TO_CLASS_PATH` + `_PARADATA_CLASS_TO_UNITA_TIPO` maps. Composite-key index `nodes_by_key: dict[(name, unita_tipo), node]` con slot `__STRAT__` per placeholder bridge. Sostituisce `strat_by_name: dict[name, node]` (name-only collisions). `_create_paradata_node_for_unita_tipo()` + `_create_stratigraphic_node_for_unita_tipo()` helper per create-on-demand. Cleanup orphan `__STRAT__` solo per nomi con paradata-row matching (preserva nodi cross-sito del bridge). |
| **N** | `populate_graph` riordinato: `_propagate_node_uuid_and_us` BEFORE `_enrich_into` (era post). `_enrich_into` ora composite-key aware (`nodes_by_us_ut` + `nodes_by_name` indices). Family-preference target resolver (`_PARADATA_UTS` frozenset + `_ut_family()` helper). Risolve "edges added pre-replacement aliased to wrong class" warning chain. |
| **P** | `_create_paradata_node_for_unita_tipo` ora ritorna `StratigraphicUnit` Python class con `attributes['unita_tipo']='DOC'/'Combinar'/etc.` (era `DocumentNode`/`CombinerNode`/`ExtractorNode`/`PropertyNode`). Render dentro swimlane GraphMLExporter via dispatch by `unita_tipo` attr; isolated paradata classes (Bug O approach) reverted perché finivano OUT-of-swimlane senza edges. |

In `modules/s3dgraphy/sync/graphml_writer.py`:

| ID | Sintesi |
|---|---|
| **Q** | `_VISUAL_BY_UNITA_TIPO`: aggiunto entry `"USV"` mirroring `"USVs"` (blue parallelogram `#248FE7`, black fill `#000000`, white plain text). pyarchinit canonical `unita_tipo='USV'` ora rendered con palette EM 1.5 corretta (era rectangle red-border `#9B3333` default fallback). `_resolve_display_label`: paradata kinds preferiscono `descrizione` (= `d_stratigrafica`) → property label = `material` (non `propertymaterial`); DOC/Extractor/Combinar = label originale (non `D.<us_suffix>`). |

In `modules/s3dgraphy/sync/graphml_writer.py:_filter_by_site`:

Branch `isinstance(StratigraphicNode)` ora copre anche le row-paradata istanze (Bug P le ha rese StratigraphicUnit). Nessuna logica filter-side aggiuntiva richiesta dopo refactor.

In `gui/yed_import_dialog.py`:

| ID | Sintesi |
|---|---|
| **J** | `_kind_choices()` lista include `ClassificationKind.EXTRACTOR` (Bug D enum aggiunto ma dialog combobox non aggiornato → `ValueError: 'extractor' is not in list` quando wizard incontra nodo Extractor). |

**Test delta**: 311 → 329 (+18 regression test), 0 regression, AC-2 byte-identical preservato. Coverage espansa per: Bug E strip helper edge cases (paradata fallback + identity variants), Bug F token dispatch per famiglia, Bug G classification destination layout, Bug H pre-existing-skip idempotency, Bug I BPMN-aware classifier, Bug J wizard combobox, Bug K composite-key projector, Bug N row-paradata via StratigraphicNode, Bug R B1 no-dedup multi-folder, Bug S rapporti per-occurrence target.

**Predecessore**: `[yed-aware-import]` (2026-05-14, commit `cbc2a5b7`). Rollout originale del 6+3 milestone yE-A → yE-Closure. Il fastfix corregge i bug residui emersi nei manual test su 9 db reali.

### English

**Progressive yE-D fastfix batch A→T** (pyarchinit tag `yed-fastfix-5.8.5-alpha`, commit `a5e8502b` on `Stratigraph_00001`). 19 bugs surfaced through manual testing on `pyarchinit_test{002..010}.sqlite` — single cumulative commit after iterations `cartella senza nome 7→14`. pyarchinit suite 311 → 354 → **329 after refactor** (+18 regression tests). AC-2 byte-identical preserved.

**Main trade-off (Bug R)**: multi-folder visibility chosen over identity-dedup for paradata kinds. `material`, `D.01`, `position` referenced cross-folder now yield N us_table rows (one per yEd occurrence, with `us` suffix `_2`/`_3`, and `d_stratigrafica` = original label) — restores the visual fan-out in the re-exported yEd graphml. Identity-dedup of `D.001` / `D.001-2` / `D.001bis` variants collapsing into one row lost; future recovery via yE-F design (single row + `other_locations` + render-time fan-out).

**Tag released:**

| Milestone | Tag | Commit |
|---|---|---|
| yE-D Fastfix batch A→T | `yed-fastfix-5.8.5-alpha` | `a5e8502b` |

(Versioning: `5.8.4-alpha` reserved for internal dry-run between Bug E and Bug R; public release jumps to `5.8.5-alpha`.)

**19 bugs by module** (see Italian tables above for full descriptions): `yed_import_pipeline.py` (A, B, C, E, F, G, H, M, R, S, T — 11 fixes), `yed_classifier.py` (D EXTRACTOR enum + regex, I BPMN-aware classification — 2 fixes), `graph_projector.py` (K composite-key + create-on-demand helpers, N reorder + family-preference, P row-paradata via StratigraphicNode-class — 3 fixes), `graphml_writer.py` (Q USV palette + property label — 1 fix), `yed_import_dialog.py` (J wizard combobox — 1 fix). Test increment: 311 → 329.

**Predecessor**: `[yed-aware-import]` (2026-05-14, commit `cbc2a5b7`). Original 6+3 milestone rollout (yE-A → yE-Closure). The fastfix corrects residual bugs emerged in manual tests across 9 real DBs.

---

## [yed-aware-import] — 2026-05-14

### Italiano

**Rollout `yEd-aware graphml import` completo** (pyarchinit tags `yed-import-foundation-5.7.5-alpha` → `yed-import-closure-5.8.3-alpha`). 6 milestone yE + 1 dependency bump (s3dgraphy 0.1.42) + 3 PG-pottery hotfix in 3 giorni (2026-05-12 → 2026-05-14). Suite pyarchinit 298 → 354 (+56 test). AC-2 byte-identical preservato attraverso tutti i 10 tag.

**Tag rilasciati nella rollout:**

| Milestone | Tag | Commit |
|---|---|---|
| yE-A Foundation (detector) | `yed-import-foundation-5.7.5-alpha` | `eb4fba81` |
| yE-B Classifier (13 kind) | `yed-import-classifier-5.7.6-alpha` | `640b4e83` |
| yE-C Parsers (period + folder) | `yed-import-parsers-5.7.7-alpha` | `5d666c67` |
| yE-D Pipeline (orchestrator + 4 policies + paradata) | `yed-import-pipeline-5.8.0-alpha` | `bfd9c858` |
| s3dgraphy bump 0.1.42 (RSF spolia) | `s3dgraphy-bump-5.8.1-alpha` | `7f5f82a8` |
| yE-E Dialog UX (QWizard + sidecar) | `yed-import-dialog-5.8.2-alpha` | `7120dc23` |
| pg-pottery fix (bidirectional coercion) | `pg-pottery-fix-5.8.2.1-alpha` | `2788ccf7` |
| pg-pottery belt-and-braces | `pg-pottery-fix-belt-and-braces-5.8.2.2-alpha` | `3f6956d2` |
| pg-pottery typefix (decl Int→Text) | `pg-pottery-typefix-5.8.2.3-alpha` | `3f30d368` |
| yE-Closure (sign-off + tutorial 36) | `yed-import-closure-5.8.3-alpha` | `cbc2a5b7` |

**Nuovi simboli pubblici (yE-A → yE-Closure):**

In `modules/s3dgraphy/sync/`:

| Simbolo | Modulo | Ruolo |
|---|---|---|
| `detect_flavor(graphml_path) -> 'yed-raw'\|'pyarchinit-projected'` | `yed_detector.py` | Riconosce se un graphml è autorato in yEd raw o esportato da pyarchinit (presenza `pyarchinit.*` data keys). |
| `ClassificationKind` (Enum, 13 values) | `yed_classifier.py` | US_REAL / US_MASONRY / US_DOCUMENTARY / USV_VIRTUAL / USV_FORMAL / **REUSED_SPECIAL_FIND** (RSF, 5.8.1) / SPECIAL_FIND / VIRTUAL_FIND / DOCUMENT / COMBINER / PROPERTY / UNKNOWN / SKIP. |
| `ClassifiedNode` (dataclass) | `yed_classifier.py` | `yed_id` + `label` + `auto_kind` + `user_kind` + `extra_attrs`. |
| `classify_leaves(graphml_path, rules=None) -> list[ClassifiedNode]` | `yed_classifier.py` | Regex order-sensitive: USVs/USVn → USVf; USV → USV_VIRTUAL; USM → US_MASONRY; USD → US_DOCUMENTARY; US → US_REAL; VSF → VIRTUAL_FIND; **RSF → REUSED_SPECIAL_FIND**; SF → SPECIAL_FIND; D./C. → DOCUMENT/COMBINER; keyword `material/position/width/...` → PROPERTY. |
| `DEFAULT_CLASSIFIER_RULES` | `yed_classifier.py` | Lista `(regex, kind)` ordinata. |
| `PeriodCandidate` (dataclass) | `yed_table_parser.py` | TableNode row → `yed_row_id`, `auto_label`/`user_label`, `auto_periodo`/`auto_fase`/`user_periodo`/`user_fase`, `member_yed_ids`, `y_min`/`y_max`. |
| `extract_periods(graphml_path) -> list[PeriodCandidate]` | `yed_table_parser.py` | Trova `<y:TableNode>` con righe; mappa colonne → period/phase. |
| `FolderCandidate` (dataclass) | `yed_group_walker.py` | Group folder → `yed_id`, `full_label`, `auto_dimension`/`user_dimension`, `auto_value`/`user_value`, `member_yed_ids`, `nested_folder_ids`, `parent_folder_id`. |
| `walk_folders(graphml_path) -> list[FolderCandidate]` | `yed_group_walker.py` | Walka i group folder ricorsivamente. Auto-dimension da prefix label (VA→attivita / AR→area / SR→struttura / ST→settore / AM→ambient / SG→saggio / QP→quad_par). |
| `DEFAULT_FOLDER_PREFIX_MAP` | `yed_group_walker.py` | dict `prefix → dimension`. |
| `FolderEdgePolicy` (StrEnum) | `yed_rapporti_policy.py` | SKIP / FAN_OUT / REPRESENTATIVE / SYNTHETIC. |
| `FolderEdge`, `FolderEdgeReport`, `ExpandedRapporti` (dataclasses) | `yed_rapporti_policy.py` | Modello dati edges + report classificato + risultato espanso (rapporti list + synthetic_us_rows). |
| `analyze_edges(graphml, classified, folders) -> FolderEdgeReport` | `yed_rapporti_policy.py` | Splitta gli edge in leaf-to-leaf / folder-touching / folder self-loops (sempre filtrati). |
| `apply_policy(report, policy, *, all_folders, classified) -> ExpandedRapporti` | `yed_rapporti_policy.py` | Applica una delle 4 policy attive. |
| `YedOverrides` (dataclass) | `yed_import_pipeline.py` | User diffs (5.8.2-alpha): `classifier: dict[yed_id, ClassificationKind]`, `periods`, `folders`, `policy`. |
| `apply_overrides_to_drafts(drafts, overrides) -> dict` | `yed_import_pipeline.py` | Pure function: returna nuovo drafts con `user_*` valorizzati. |
| `import_yed_raw(handle, graphml_path, sito, drafts, *, policy=SKIP, dry_run=False, overrides=None) -> IngestResult` | `yed_import_pipeline.py` | **Orchestrator principale.** Atomic transaction via `engine.begin()`. DbHandle PG+SQLite. `overrides=None` = comportamento yE-D hardcoded-defaults. `dry_run=True` = `_DryRunRollback` sentinel (PG-C heritage). |
| `_classify_destination(classified)` | `yed_import_pipeline.py` | Buckets: sql_us (5 kind incl. USV* + RSF), sql_inv (SPECIAL_FIND), paradata (4 kind), skipped. |
| `_resolve_unita_tipo(c)` | `yed_import_pipeline.py` | Mappa ClassifiedNode → unita_tipo str. USV_FORMAL deriva il prefix dal label (USVs/USVn/USVc). |
| `_write_us_rows`, `_write_inventario_rows`, `_write_periodizzazione_rows`, `_apply_yed_folder_dimensions`, `_write_rapporti`, `_write_paradata_via_store` | `yed_import_pipeline.py` | 5 standalone SQL write functions. Path B per PropertyNode (no US linkage). |

In `modules/s3dgraphy/sync/graph_ingestor.py`:

| Simbolo | Note |
|---|---|
| Branch hook in `populate_list()` (lines 166-216) | yEd-raw detection + dispatch + RETURN `import_yed_raw()` result. Legacy `_run()` path UNCHANGED per pyarchinit-projected (AC-2 sacred). yE-E (5.8.2-alpha) extension: probes `QApplication.instance()` e apre `YedImportDialog` se interattivo. |
| `_S3DGRAPHY_TYPE_TO_UNITA_TIPO` | Aggiunti 2 entry: `"VirtualActivity": "VA"` (5.8.0-alpha, SYNTHETIC policy) + `"ReusedSpecialFind": "RSF"` (5.8.1-alpha, s3dgraphy 0.1.42 spolia). |

In `gui/`:

| Simbolo | Modulo | Ruolo |
|---|---|---|
| `YedImportDialog(QWizard)` | `gui/yed_import_dialog.py` | 5-page wizard: classifier / periods / folders / rapporti policy / preview. |
| `_ClassifierPage`, `_PeriodsPage`, `_FoldersPage`, `_RapportiPolicyPage`, `_PreviewPage` | `gui/yed_import_dialog.py` | QWizardPage subclasses. |
| `save_sidecar(graphml, overrides) -> Path` | `gui/yed_import_dialog.py` | Persist YedOverrides in `<graphml>.yed_overrides.json` schema versionato. |
| `load_sidecar(graphml) -> YedOverrides` | `gui/yed_import_dialog.py` | Pre-populate wizard. Forward-compat: unknown ClassificationKind values skipped. |
| `sidecar_path(graphml) -> Path` | `gui/yed_import_dialog.py` | Helper path computation. |

In `scripts/`:

| Simbolo | Modulo | Ruolo |
|---|---|---|
| `import_yed_graphml.py` (CLI) | `scripts/import_yed_graphml.py` | argparse: `<graphml>` + `--site` + `--db|--conn-str` mutex + `--policy {skip|fan_out|representative|synthetic}` + `--overrides PATH` (5.8.2-alpha) + `--dry-run` + `--auto-defaults` (no-op forward-compat) + `-v`. |

In `modules/db/pyarchinit_db_manager.py`:

| Simbolo | Note |
|---|---|
| `_normalize_query_params(params, table_class_name)` | **Bidirectional coercion** (5.8.2.1-alpha pg-pottery-fix): str + numeric col → coerce numeric; int/float + str col → coerce str. Bool escluso. Mappato BEFORE cache lookup (pg-media-fix-5.7.9.1-alpha pattern). |
| `query_bool` body (line 2996-3010) | **Belt-and-braces** (5.8.2.2-alpha): stessa coercion bidirezionale inline nel loop, in caso `_normalize_query_params` venga bypassato (stale `sys.modules`). |

In `modules/db/structures/`:

| File | Cambio | Tag |
|---|---|---|
| `Pottery_table.py:19` | `Column('us', Integer)` → `Column('us', Text)` | `pg-pottery-typefix-5.8.2.3-alpha` |
| `US_table.py:20` | `Column('us', Integer)` → `Column('us', Text)` | `pg-pottery-typefix-5.8.2.3-alpha` |

ORM declarations allineate al PG schema reale (TEXT). SQLite type-loose unaffected.

In `scripts/modules_installer.py`:

| Simbolo | Note |
|---|---|
| `_cleanup_stale_dists(package_name)` | NUOVO (5.8.1-alpha): rimuove `<pkg>-*.dist-info/` dirs in `ext_libs/` + `<pkg>/__pycache__/` prima di `pip install --upgrade`. Risolve il pile-up storico di `s3dgraphy-0.1.40.dist-info` + `0.1.41.dist-info` + `0.1.42.dist-info` side-by-side. |
| `_extract_pkg_name(requirement)` | NUOVO: estrae nome canonico da requirement line (PyPI normalisation: lowercase + underscore→dash). |

**Test count finale post-rollout**: 354 passed / 42 skipped (PG-L1 require psycopg2 — 6 tests). Up from 298 a phase3-pgcompat closure (2026-05-11), **+56 test in 3 giorni**.

**Animazioni / Tutorial**: Tutorial 36 (`docs/tutorials/<lang>/36_extended_matrix_s3dgraphy.md`) esteso in **IT + EN** con sezione "5. yEd-aware Import" coprendo l'intero rollout end-to-end. Altre 8 lingue (de/es/fr/ar/ca/ro/pt/el) deferite a batch separato via `tutorial-updater` agent.

**Dev-log master**: `docs/superpowers/dev-log/T5.4_PyArchInit_Dev_Log.md` prepended con sezioni yE-Closure + yE-E + yE-D.

### English

**`yEd-aware graphml import` rollout complete** (pyarchinit tags `yed-import-foundation-5.7.5-alpha` → `yed-import-closure-5.8.3-alpha`). 6 yE milestones + 1 dependency bump (s3dgraphy 0.1.42) + 3 PG-pottery hotfixes in 3 days (2026-05-12 → 2026-05-14). pyarchinit test suite 298 → 354 (+56 tests). AC-2 byte-identical preserved across all 10 tags.

**Public symbols added** (see Italian table above for the full list): `detect_flavor`, `ClassificationKind` enum (13 values), `classify_leaves`, `PeriodCandidate`/`extract_periods`, `FolderCandidate`/`walk_folders`, `FolderEdgePolicy` (4 active policies), `analyze_edges`, `apply_policy`, `ExpandedRapporti`, `import_yed_raw` orchestrator (with `overrides: YedOverrides | None` parameter from 5.8.2-alpha), `YedOverrides` dataclass, `apply_overrides_to_drafts`, `YedImportDialog(QWizard)` + 5 page subclasses, `save_sidecar`/`load_sidecar`/`sidecar_path`, CLI `scripts/import_yed_graphml.py` with `--overrides PATH` flag.

**Modified internals**: `_normalize_query_params` extended to bidirectional coercion (pg-media-fix-5.7.9.1-alpha + pg-pottery-fix-5.8.2.1-alpha lineage) + inline defensive coercion in `query_bool` body (belt-and-braces, 5.8.2.2-alpha). `_S3DGRAPHY_TYPE_TO_UNITA_TIPO` map extended with `VirtualActivity → VA` and `ReusedSpecialFind → RSF`. `Pottery_table.us` + `US_table.us` ORM declarations aligned to TEXT (matching PG schema; pg-pottery-typefix-5.8.2.3-alpha).

**Tooling addition**: `scripts/modules_installer.py` now auto-cleans stale `<pkg>-*.dist-info/` directories in `ext_libs/` before every `pip install --upgrade` (5.8.1-alpha, fix for user-reported side-by-side dist-info pile-up).

**Final test count**: 354 passed / 42 skipped. AC-2 preserved across all 10 tags.

**Tutorials**: Tutorial 36 (`docs/tutorials/<lang>/36_extended_matrix_s3dgraphy.md`) extended IT + EN with section "5. yEd-aware Import". Other 8 languages deferred to batch via `tutorial-updater` agent.

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
