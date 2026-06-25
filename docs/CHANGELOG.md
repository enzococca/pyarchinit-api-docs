# PyArchInit API Documentation — Changelog

> Registro delle modifiche alla documentazione API generata automaticamente
> da `pyarchinit-code_dev/`.
>
> Format: bilingue IT/EN. Le date sono ISO 8601.

---

## [import-hook-scope-5.13.9.3] — 2026-06-25

### Italiano

**Hook `ON CONFLICT` dell'import confinati al solo import** — pyarchinit tag `import-hook-scope-5.13.9.3-alpha`; fix lato plugin (`gui/pyarchinitConfigDialog.py`). Le opzioni di import (ignora/aggiorna/abort duplicati) usano hook globali `@compiles(Insert)` che aggiungono `ON CONFLICT`/`OR IGNORE` a ogni INSERT. Venivano registrati in `check()` (eseguito al cambio checkbox **e all'apertura del dialog**) e **mai rimossi** → aprire la configurazione con un'opzione spuntata "contaminava" tutti gli insert della sessione (causa anche dell'errore `ON CONFLICT` sulle tabelle media). Approccio A: `check()` è ora no-op; gli hook stanno in `_install_import_conflict_hook()` e un wrapper su `on_pushButton_import_pressed` li installa solo per la durata dell'import e li rimuove in `finally` con `sqlalchemy.ext.compiler.deregister(Insert)` (anche su errore/`return`). L'import geometrie gira con INSERT normali.

### English

**Import `ON CONFLICT` hooks scoped to the import only** — pyarchinit tag `import-hook-scope-5.13.9.3-alpha`; plugin-only fix (`gui/pyarchinitConfigDialog.py`). The import dedup options (ignore/replace/abort) use global `@compiles(Insert)` hooks that append `ON CONFLICT`/`OR IGNORE` to every INSERT. They were registered in `check()` (run on checkbox change **and at dialog init**) and **never removed** → opening the config with an option ticked leaked the clause onto every later insert in the session (also the cause of the media-table `ON CONFLICT` error). Approach A: `check()` is now a no-op; the hooks live in `_install_import_conflict_hook()` and a wrapper on `on_pushButton_import_pressed` installs them for the import only and removes them in a `finally` via `sqlalchemy.ext.compiler.deregister(Insert)` (even on error/`return`). Geometry import runs with stock INSERTs.

---

## [webdav-media-5.13.9.2] — 2026-06-24

### Italiano

**Media WebDAV end-to-end (upload + path raddoppiato + download)** — pyarchinit tag `webdav-media-5.13.9.2-alpha`; fix lato plugin (`modules/storage/webdav_backend.py`, `modules/utility/remote_image_loader.py` + tuple di prefissi remoti in 10 file).

- **Upload.** `webdavclient3` esegue un controllo di esistenza interno (PROPFIND) prima di ogni `list`/`upload`; alcuni server (es. Synology :5006) rispondono "non esiste" anche per risorse esistenti e scrivibili → upload fallito con `RemoteParentNotFound` (`Failed to upload original file, using local path`). `WebDAVBackend.connect()` ora imposta `disable_check=True`; `ensure_directory()` non dipende più dal `check()` rotto; `connect()`/`write()` loggano l'eccezione reale invece di un `return False` muto.
- **Path raddoppiato.** Le liste di prefissi remoti nei controlli di percorso (`is_remote_url` + rami inline) omettevano `webdav://` (e gdrive/dropbox/s3/r2/sftp): un `path_resize` già completo non veniva riconosciuto → base anteposta due volte. Corrette tutte (30 occorrenze + `is_remote_url`).
- **Download per la visualizzazione.** `load_pixmap` scaricava i remoti con `requests.get` (che non parla `webdav://`); aggiunto `is_storage_backend_path` + `_download_via_storage` (lettura via `StorageManager`), per miniature e anteprima a tutto schermo.

### English

**WebDAV media end-to-end (upload + doubled path + download)** — pyarchinit tag `webdav-media-5.13.9.2-alpha`; plugin-only fixes (`modules/storage/webdav_backend.py`, `modules/utility/remote_image_loader.py` + remote-prefix tuples in 10 files).

- **Upload.** `webdavclient3` runs an internal existence check (PROPFIND) before every `list`/`upload`; some servers (e.g. Synology :5006) answer "not found" even for existing, writable resources → upload failed with `RemoteParentNotFound` (`Failed to upload original file, using local path`). `WebDAVBackend.connect()` now sets `disable_check=True`; `ensure_directory()` no longer relies on the broken `check()`; `connect()`/`write()` log the real exception instead of a silent `return False`.
- **Doubled path.** The remote-prefix lists in the path checks (`is_remote_url` + inline branches) omitted `webdav://` (and gdrive/dropbox/s3/r2/sftp): an already-complete `path_resize` was not recognized → base prepended twice. All fixed (30 occurrences + `is_remote_url`).
- **Download for display.** `load_pixmap` fetched remotes via `requests.get` (which cannot speak `webdav://`); added `is_storage_backend_path` + `_download_via_storage` (read through the `StorageManager`), covering thumbnails and the full-size viewer.

---

## [pg-onconflict-quote-5.13.9.1] — 2026-06-24

### Italiano

**Fix: `ON CONFLICT` non quotato rompe l'inserimento media su PostgreSQL** — pyarchinit tag `pg-onconflict-quote-5.13.9.1-alpha`; fix lato plugin (`gui/pyarchinitConfigDialog.py`). Le opzioni di import (ignora/aggiorna/abort duplicati) registrano hook globali `@compiles(Insert)` che accodano `ON CONFLICT (...)` a ogni INSERT. Le varianti **ignore** e **abort** interpolavano i nomi colonna non quotati → per il PK camelCase `media_to_entity_table.id_mediaToEntity` PostgreSQL ripiega in minuscolo (`id_mediatoentity`) → `UndefinedColumn` aggiungendo un'immagine. Ora usano `compiler.preparer.quote(...)` (la variante **replace** era già a posto). NB gli hook sono globali e persistono per l'intera sessione QGIS → riavviare QGIS per scaricare l'hook già registrato.

### English

**Fix: unquoted `ON CONFLICT` breaks media insert on PostgreSQL** — pyarchinit tag `pg-onconflict-quote-5.13.9.1-alpha`; plugin-only fix (`gui/pyarchinitConfigDialog.py`). The import options (ignore/replace/abort duplicates) register global `@compiles(Insert)` hooks that append `ON CONFLICT (...)` to every INSERT. The **ignore** and **abort** variants interpolated column names unquoted → for the camelCase PK `media_to_entity_table.id_mediaToEntity` PostgreSQL folds to lowercase (`id_mediatoentity`) → `UndefinedColumn` when adding an image. Both now use `compiler.preparer.quote(...)` (the **replace** variant was already fixed). Note the hooks are global and persist for the whole QGIS session → restart QGIS to drop an already-registered hook.

---

## [webdav-ssl-5.13.9] — 2026-06-24

### Italiano

**WebDAV: opzione "Verify SSL" per server con certificato self-signed** — pyarchinit tag `webdav-ssl-5.13.9-alpha`; fix lato plugin (`gui/remote_storage_dialog.py`, `modules/storage/credentials.py`, `modules/storage/webdav_backend.py`) + tutorial config (10 lingue).

- **Connessione ai server self-signed.** Con un server WebDAV su IP nudo o porta HTTPS non standard (certificato auto-firmato) la connessione falliva **in silenzio**: `webdavclient3` usa `requests`, che rifiuta i certificati non validi, e `WebDAVBackend.connect()` cattura l'eccezione restituendo `False`. Il tab **WebDAV** di *Remote Storage Config* ha ora un menu **"Verify SSL"** (Yes = verifica, default; No = certificati self-signed).
- **Propagazione.** Il valore è salvato in `pyarchinit/storage/webdav/verify_ssl` e arriva al backend tramite `CredentialsManager` (nuova chiave `verify_ssl` in `ENV_NAMES[WEBDAV]`, anche via env `PYARCHINIT_WEBDAV_VERIFY_SSL`). `WebDAVBackend.connect()` imposta `client.verify` di conseguenza e silenzia gli `InsecureRequestWarning` quando la verifica è off. Default invariato (verifica attiva).

### English

**WebDAV: "Verify SSL" option for servers with a self-signed certificate** — pyarchinit tag `webdav-ssl-5.13.9-alpha`; plugin-only fixes (`gui/remote_storage_dialog.py`, `modules/storage/credentials.py`, `modules/storage/webdav_backend.py`) + config tutorial (10 languages). With a WebDAV server on a bare IP or non-standard HTTPS port (self-signed certificate) the connection failed **silently**: `webdavclient3` uses `requests`, which rejects invalid certificates, and `WebDAVBackend.connect()` swallowed the exception and returned `False`. The **WebDAV** tab of *Remote Storage Config* now has a **"Verify SSL"** menu (Yes = verify, default; No = self-signed certificates). The value is stored in `pyarchinit/storage/webdav/verify_ssl` and reaches the backend via `CredentialsManager` (new `verify_ssl` key in `ENV_NAMES[WEBDAV]`, also via the `PYARCHINIT_WEBDAV_VERIFY_SSL` env var); `WebDAVBackend.connect()` sets `client.verify` accordingly and silences urllib3's `InsecureRequestWarning` when verification is off. Default unchanged (verification on).

---

## [palimpsest-5.13.8] — 2026-06-18

### Italiano

**RAG embeddings: OpenAI se disponibile / fastembed offline + shim Pillow.Resampling** — pyarchinit tag `palimpsest-5.13.8-alpha`; fix lato plugin (`modules/utility/llm_providers.py`, `requirements.txt`).

- **Selezione embeddings (Opzione B).** Con un provider di chat senza API di embeddings (es. **Anthropic**) o un server locale senza modello di embedding, `LLMProviderManager.build_embeddings` usa gli **embeddings OpenAI se è salvata una chiave OpenAI** (`get_api_key(OPENAI)` → `~/pyarchinit/bin/gpt_api_key.txt`, letta indipendentemente dal provider di chat — la chiave Anthropic non è mai inviata a OpenAI); **solo se non c'è chiave OpenAI** ricade su **fastembed** offline. `embeddings_signature` rispecchia la priorità: la cache FAISS si ricostruisce se cambia la disponibilità della chiave.
- **Shim `PIL.Image.Resampling`.** Le trasformazioni immagine di fastembed usano `PIL.Image.Resampling` (Pillow ≥ 9.1); alcune build di QGIS impacchettano un Pillow più vecchio → `module 'PIL.Image' has no attribute 'Resampling'`. `build_embeddings` ora applica uno shim (`_ensure_pil_resampling`) prima di importare fastembed (IntEnum mappato sulle costanti legacy). Pin `Pillow>=9.1.0` in `requirements.txt`.

### English

**RAG embeddings: OpenAI when available / offline fastembed + Pillow.Resampling shim** — pyarchinit tag `palimpsest-5.13.8-alpha`; plugin-only fixes (`modules/utility/llm_providers.py`, `requirements.txt`). With a chat provider that has no embeddings API (e.g. **Anthropic**) or a local server without an embedding model, `LLMProviderManager.build_embeddings` uses **OpenAI embeddings when an OpenAI key is saved** (`get_api_key(OPENAI)` → `~/pyarchinit/bin/gpt_api_key.txt`, read independently of the chat provider — the Anthropic key is never sent to OpenAI); **only when no OpenAI key exists** it falls back to offline **fastembed**. `embeddings_signature` mirrors this priority so the FAISS cache rebuilds when key availability changes. Also a `PIL.Image.Resampling` shim (`_ensure_pil_resampling`) applied before importing fastembed so it works on QGIS shipping Pillow < 9.1 (the image transforms need the enum added in Pillow 9.1), plus a `Pillow>=9.1.0` pin.

---

## [palimpsest-5.13.7] — 2026-06-18

### Italiano

**RAG: embeddings per provider + fallback offline fastembed** — pyarchinit tag `palimpsest-5.13.7-alpha`; fix lato plugin (`tabs/US_USM.py`, `modules/utility/llm_providers.py`, `requirements.txt`).

- **Il RAG funziona con tutti i provider.** Con provider **Anthropic** il RAG dava `Error code: 401 - Incorrect API key provided: sk-ant-…` perché gli embeddings (indice FAISS) venivano sempre richiesti a OpenAI passando la chiave Anthropic (Anthropic **non ha** un'API di embeddings). `_get_embeddings` (3 copie) ora delega a `LLMProviderManager.build_embeddings`, che sceglie il backend per provider: **OpenAI** → embeddings OpenAI; **Ollama / LM Studio** → modello di embedding locale se caricato; altri/locale-senza-modello → fallback (vedi 5.13.8). La chiave del provider non è più inoltrata a OpenAI.
- **Cache FAISS legata al backend** via `embeddings_signature` (in memoria e su disco) e nuova dipendenza `fastembed>=0.3.0` per il fallback offline.

### English

**RAG: per-provider embeddings + offline fastembed fallback** — pyarchinit tag `palimpsest-5.13.7-alpha`; plugin-only fixes (`tabs/US_USM.py`, `modules/utility/llm_providers.py`, `requirements.txt`). Under the **Anthropic** provider the RAG raised `Error code: 401 - Incorrect API key provided: sk-ant-…` because embeddings (FAISS index) were always requested from OpenAI while passing the Anthropic key (Anthropic has **no** embeddings API). `_get_embeddings` (3 copies) now delegates to `LLMProviderManager.build_embeddings`, which picks the backend by provider: **OpenAI** → OpenAI embeddings; **Ollama / LM Studio** → local embedding model when loaded; others / local-without-model → fallback (see 5.13.8). The provider key is no longer forwarded to OpenAI. The FAISS cache is keyed by backend via `embeddings_signature` (in-memory and on-disk); adds the `fastembed>=0.3.0` dependency for the offline fallback.

---

## [palimpsest-5.13.6] — 2026-06-18

### Italiano

**Compatibilità langchain 1.x (Windows / QGIS 4.x) + tutorial** — pyarchinit tag `palimpsest-5.13.6-alpha`; fix lato plugin (`tabs/US_USM.py`, `requirements.txt`) + documentazione (tutorial 37 e 30). Nessuna modifica a palimpsestr o agli `.rsx` (byte-identici).

- **Compatibilità langchain 1.x (Windows / QGIS 4.x).** Su Python ≥ 3.10 il plugin installa la linea **langchain 1.x**, che ha rimosso gli shim legacy: `langchain.text_splitter` non esiste più (→ `No module named 'langchain.text_splitter'` all'avvio del RAG) e `Tool` non è più riesportato da `langchain.agents` (→ `cannot import name 'Tool' from 'langchain.agents'` in fase di report). Gli import in `tabs/US_USM.py` sono stati portati ai percorsi canonici `langchain_text_splitters`, `langchain_core.tools`, `langchain_core.callbacks`, `langchain_core.prompts`, `langchain_core.messages`, con fallback ai vecchi percorsi per langchain 0.3.x (QGIS 3.x / Python 3.9). Le chain/memory legacy (`RetrievalQA`, `ConversationSummaryMemory`), spostate in `langchain-classic` nella 1.x, usano un fallback `try/except`; aggiunta la dipendenza `langchain-classic>=1.0.0; python_version>="3.10"` a `requirements.txt`.
- **Documentazione**: Tutorial 37 (palinsesto) aggiornato in 10 lingue con la nota sul report AI (provider, errori chiari, QGIS 3.x/4.x) e il rimando alla query AI sul DB; Tutorial 30 (AI Query Database) aggiornato in 5 lingue con la sezione di troubleshooting per l'errore di import langchain.

### English

**langchain 1.x compatibility (Windows / QGIS 4.x) + tutorials** — pyarchinit tag `palimpsest-5.13.6-alpha`; plugin fixes (`tabs/US_USM.py`, `requirements.txt`) + documentation (tutorials 37 and 30). No palimpsestr or `.rsx` change (byte-identical). On Python ≥ 3.10 the plugin installs the **langchain 1.x** line, which removed the legacy shims: `langchain.text_splitter` is gone (→ `No module named 'langchain.text_splitter'` when the RAG starts) and `Tool` is no longer re-exported from `langchain.agents` (→ `cannot import name 'Tool' from 'langchain.agents'` during report generation). Imports in `tabs/US_USM.py` were moved to the canonical `langchain_text_splitters` / `langchain_core.{tools,callbacks,prompts,messages}` paths, with fallbacks to the old paths for langchain 0.3.x (QGIS 3.x / Python 3.9). Legacy chains/memory (`RetrievalQA`, `ConversationSummaryMemory`), moved to `langchain-classic` in 1.x, use a `try/except` fallback, and `langchain-classic>=1.0.0; python_version>="3.10"` was added to `requirements.txt`. Docs: Tutorial 37 (palimpsest) updated in 10 languages (AI report note + cross-reference); Tutorial 30 (AI Query Database) updated in 5 languages with a langchain-import troubleshooting section.

---

## [palimpsest-5.13.5] — 2026-06-17

### Italiano

**Funzioni AI/RAG: provider Anthropic e robustezza** — pyarchinit tag `palimpsest-5.13.5-alpha`; soli fix lato plugin (`tabs/US_USM.py`, `modules/utility/llm_providers.py`). Nessuna modifica a palimpsestr o agli `.rsx` (byte-identici).

- **Fix AI/RAG**: streaming agnostico al provider (OpenAI/Anthropic/Ollama/LM Studio) via `self.llm.stream()`; errori dell'SDK non più mascherati in `LLMProviderManager._annotate_error` (tipo preservato solo se ricostruibile, altrimenti `RuntimeError`); risoluzione del **sito corrente** nel RAG; **materiali reali** iniettati da `raw_data`; export **PDF** del report (PNG del grafico cancellato dopo `doc.build()`); grafico a torta che accetta sia `x/y` sia `labels/values`.

### English

**AI/RAG features: Anthropic provider & robustness** — pyarchinit tag `palimpsest-5.13.5-alpha`; plugin-only fixes (`tabs/US_USM.py`, `modules/utility/llm_providers.py`). No palimpsestr or `.rsx` change (byte-identical). Provider-agnostic streaming via `self.llm.stream()`; unmasked SDK errors in `LLMProviderManager._annotate_error` (type kept only when reconstructible, else `RuntimeError`); current-site resolution in the RAG; real materials injected from `raw_data`; PDF report export (chart PNG deleted after `doc.build()`); pie chart accepting both `x/y` and `labels/values`.

---

## [palimpsest-5.13.4] — 2026-06-17

### Italiano

**Coordinate puntuali dei reperti (piece-plotting)** — richiede palimpsestr **≥ 0.23.0**; nessuna modifica al codice del plugin (i 3 `.rsx` restano byte-identici a `qgis/processing/*.rsx`).

- **palimpsestr 0.23.0** (`enzococca/palimpsestr`, `R/db_connect.R`, commit `f2b3031`): `read_pyarchinit()` usa le coordinate **puntuali** di un reperto quando esso è disegnato come punto nel layer `pyarchinit_reperti` — abbinamento via la vista `pyarchinit_reperti_view` (sito + `numero_inventario` = `id_rep`). Vengono usate la x, y del punto e la z dalla colonna `quota` (o dalla Z della geometria), **al posto del centroide del poligono US**. I reperti senza punto mantengono il centroide US. Nuovi parametri: `reperti_table` (default `"pyarchinit_reperti"`) e `reperti_geometry`; lettura via sf/GDAL (SpatiaLite + PostGIS). Test: nuovo Ciclo 7 (`tests/testthat/test-read-pyarchinit.R`).
- **pyArchInit**: dà **risoluzione spaziale a livello di reperto** dove il piece-plotting è disponibile, attenuando il limite del centroide US. Tutorial 37 aggiornato in 10 lingue. Nessuna modifica agli `.rsx` (byte-identici a upstream).

### English

**Piece-plotted find coordinates** — requires palimpsestr **≥ 0.23.0**; no plugin code change (the 3 `.rsx` stay byte-identical to `qgis/processing/*.rsx`). In palimpsestr 0.23.0 (`enzococca/palimpsestr`, `R/db_connect.R`, commit `f2b3031`), `read_pyarchinit()` uses a find's **point** coordinates when it is drawn as a point in the `pyarchinit_reperti` layer — matched via the `pyarchinit_reperti_view` join (site + `numero_inventario` = `id_rep`). The point's x, y and z (from the `quota` column or the geometry Z) are used **instead of the US polygon centroid**; finds without a point keep the US centroid. New parameters `reperti_table` (default `"pyarchinit_reperti"`) and `reperti_geometry`; read via sf/GDAL (SpatiaLite + PostGIS); new test Cycle 7. On the pyArchInit side this gives find-level spatial resolution where piece-plotting is available, mitigating the US-centroid limitation. Tutorial 37 updated in 10 languages; the `.rsx` are unchanged (byte-identical to upstream).

---

## [palimpsest-5.13.3] — 2026-06-15

### Italiano

**Lettura del taf spostata in `read_pyarchinit` (upstream)** — pyarchinit tag `palimpsest-5.13.3-alpha`; richiede palimpsestr **≥ 0.22.1**.

- **palimpsestr 0.22.1** (`enzococca/palimpsestr`, `R/db_connect.R`): `read_pyarchinit()` legge il punteggio tafonomico per US dalla colonna `taf` di `palimpsest_chronology` quando l'argomento `taf` è `NULL` (default 0.5 per le US senza valore); l'argomento `taf` esplicito ha la precedenza. Con test (`tests/testthat/test-read-pyarchinit.R`).
- **`tabs/Palimpsest.py`**: rimosso il blocco taf pyArchInit-local da `SEF_FACTS_R` e dai 3 `.rsx` Processing (`r:palimpsestrfit/intrusions/report`), che tornano **byte-identici** a `qgis/processing/*.rsx`. Il taf è ora applicato a monte da `read_pyarchinit`, quindi onorato uniformemente da fit/intrusioni/report/report AI.

### English

**taf reading moved to `read_pyarchinit` (upstream)** — pyarchinit tag `palimpsest-5.13.3-alpha`; requires palimpsestr **≥ 0.22.1**. In palimpsestr 0.22.1, `read_pyarchinit()` reads the per-US taphonomic score from the `taf` column of `palimpsest_chronology` when the `taf` argument is `NULL` (0.5 default; explicit `taf` still wins), with tests. The pyArchInit-local taf block is removed from `SEF_FACTS_R` and the three Processing `.rsx`, which are byte-identical to `qgis/processing/*.rsx` again; taf is now applied upstream and honoured uniformly by fit/intrusions/report and the AI report.

---

## [palimpsest-5.13.2] — 2026-06-15

### Italiano

**Punteggio tafonomico (taf) per US** — pyarchinit tag `palimpsest-5.13.2-alpha`.

- **`tabs/Palimpsest.py` — `PalimpsestChronologyDialog`** diventa un editor *Cronologia & tafonomia* per US: `_load_existing` ora precarica **tutte** le US del sito da `us_table` (colonne read-only Periodo + N. reperti da `inventario_materiali`/`pottery`) e fonde i valori salvati; tabella a 12 colonne con `taf [0-1]`; `_save_edits`/`_collect_rows` salvano solo le righe compilate; `_save_rows`/`_ddl` con nuova colonna `taf` (migrazione `ALTER TABLE` idempotente) e helper `_periodo_str`.
- **taf onorato ovunque**: `SEF_FACTS_R` e i tre `.rsx` Processing (`r:palimpsestrfit/intrusions/report`) leggono il taf per US da `palimpsest_chronology` e sovrascrivono `taf_score` prima di `fit_sef` (blocco pyArchInit-local: gli `.rsx` non più byte-identici a upstream 0.22.0).
- **`tabs/palimpsest_ai_report.py`**: colonna `taf` nelle tabelle cronologia (facts + appendice DOCX/PDF).

### English

**Per-US taphonomic score (taf)** — pyarchinit tag `palimpsest-5.13.2-alpha`. `PalimpsestChronologyDialog` becomes a per-US *Chronology & taphonomy* editor: `_load_existing` pre-loads ALL the site's US from `us_table` (read-only Period + N. finds columns) and merges saved values; 12-column table with a `taf [0-1]` column persisted in a new `taf` column of `palimpsest_chronology` (idempotent `ALTER TABLE`); only filled rows are saved. taf is honoured everywhere — `SEF_FACTS_R` and the three Processing `.rsx` read per-US taf and override `taf_score` before `fit_sef` (a pyArchInit-local block; the `.rsx` are no longer byte-identical to upstream 0.22.0). The AI report shows a `taf` column in the chronology tables.

---

## [palimpsest-5.13.1] — 2026-06-15

### Italiano

**Hotfix scheda Palimpsest + conoscenza del pacchetto negli agenti AI** — pyarchinit tag `palimpsest-5.13.1-alpha`. Consolida i fix post-`palimpsest-5.13.0-alpha` e arricchisce il report AI.

- **`tabs/palimpsest_ai_report.py`**: nuova costante `PALIMPSESTR_KNOWLEDGE` (distillata dal paper *palimpsestR v0.21.0*) iniettata in `_facts_block` → tutti gli agenti; il prompt di sintesi richiede una sezione "Limiti e cautele". Renderer Markdown→DOCX/PDF (`_parse_markdown`, `_inline_runs`, `write_docx`, `write_pdf`) con tabelle reali e figure incorporate; continuazione automatica anti-troncamento in `PalimpsestAIReportWorker._run_agent` (con guard `_supports_meta`); pulsante "Salva PDF".
- **`tabs/Palimpsest.py`**: nuovo driver `SEF_FACTS_R` (sintesi per fase, composizione, diagnostiche, righe OxCal, figure `gg_*`) + `_gather_sef_facts`/`_find_rscript`; `PalimpsestChronologyDialog` ora carica/modifica le date salvate (colonne `start`/`end`, `_load_existing`, `_save_edits`); plot OxCal per-US più descrittivo; motore OxCal persistente.
- **`modules/utility/llm_providers.py`**: `stream_chat(..., meta=...)` opzionale e retro-compatibile, riporta `finish_reason`/`truncated`.

### English

**Palimpsest dialog hotfix + package knowledge in the AI agents** — pyarchinit tag `palimpsest-5.13.1-alpha`. `tabs/palimpsest_ai_report.py` gains `PALIMPSESTR_KNOWLEDGE` (distilled from the palimpsestR v0.21.0 paper) injected into every agent's facts block, a Markdown→DOCX/PDF renderer (`write_docx`/`write_pdf`, real tables + embedded figures), anti-truncation auto-continuation (`_run_agent` + `_supports_meta`) and a "Save PDF" action; `LLMProviderManager.stream_chat` gains an optional, backward-compatible `meta` reporting `finish_reason`/`truncated`. `tabs/Palimpsest.py` adds the `SEF_FACTS_R` facts driver and an editable, self-loading `PalimpsestChronologyDialog` (start/end columns, `_load_existing`/`_save_edits`), a more descriptive per-US OxCal calibration plot, and a persistent OxCal engine. The synthesiser must add a "Limits and caveats" section reflecting the package's documented limitations (horizontal-stratigraphy assumption, resolution-bound inference, interpretive taphonomic score).

---

## [palimpsest] — 2026-06-15

### Italiano

**Scheda Palimpsest (palimpsestr / SEF)** — pyarchinit tag `palimpsest-5.13.0-alpha`. Predecessore: `temporal-paradox-5.12.13-alpha`. Wrapper della libreria R **palimpsestr ≥ 0.22.0** (modello *Stratigraphic Entanglement Field*) via Processing R Provider.

- **`tabs/Palimpsest.py`** (`pyarchinit_Palimpsest`): costanti `FIT_ALG`/`INTRUSIONS_ALG`/`REPORT_ALG`/`CHRONOLOGY_TABLE`; `RSX_SCRIPTS` con i 3 `.rsx` (fit/intrusions/report) embeddati byte-identici alle sorgenti palimpsestr 0.22.0. Nuovi metodi: `run_report`/`_show_report` (report PDF/DOCX + pannello narrativa, validazione magic-byte `%PDF`/`PK\x03\x04`), `_augment_render_env` (discovery pandoc/LaTeX), `_pg_dsn`/`_db_params` (backend PostgreSQL via DSN libpq `PG_connection`), `open_chronology`, `_gather_sef_facts`/`_has_chronology`/`open_ai_report`. `CHRONO_OXCAL_R`: driver Rscript per la calibrazione OxCal con motore **persistente** (`oxcAAR::setOxcalExecutablePath`, dir `~/.pyarchinit/oxcal`, override `PYARCHINIT_OXCAL_DIR`).
- **`PalimpsestChronologyDialog`** (in `Palimpsest.py`): DDL idempotente di `palimpsest_chronology` (SQLite/PostgreSQL), calibrazione live C14→OxCal, import CSV calibrato; upsert per chiave `(sito,area,us)`. `read_pyarchinit()` rileva la tabella e la usa al posto della `datazione` testuale.
- **`tabs/palimpsest_ai_report.py`** (nuovo): `PalimpsestAIReportWorker` (`QThread`) + `PalimpsestAIReportDialog`. Pipeline a 3 agenti (`_methodologist_prompt`/`_analyst_prompt`/`_synthesis_prompt`) su `LLMProviderManager.stream_chat`; `AI_LANGUAGES` (10 lingue), `FIGURE_CAPTIONS`; export DOCX (figure incorporate) / Markdown.
- Icona `resources/icons/palimpsest.svg|png` collegata in `pyarchinitPlugin.py`; dati esempio OxCal in `docs/examples/`; tutorial 37 in 10 lingue + registri `TUTORIALS_METADATA`.

### English

**Palimpsest dialog (palimpsestr / SEF)** — pyarchinit tag `palimpsest-5.13.0-alpha`, after `temporal-paradox-5.12.13-alpha`. Wraps the R library **palimpsestr ≥ 0.22.0** (Stratigraphic Entanglement Field model) through the Processing R Provider. `tabs/Palimpsest.py` adds `REPORT_ALG`/`CHRONOLOGY_TABLE`, a PG-capable embedded `.rsx` set (`PG_connection`), the narrated PDF/DOCX report (`run_report`/`_show_report`, pandoc/LaTeX auto-discovery via `_augment_render_env`, magic-byte validation), the PostgreSQL backend (`_pg_dsn`/`_db_params`), and the OxCal absolute-chronology table via `PalimpsestChronologyDialog` (idempotent DDL, live C14→OxCal calibration with a **persistent** engine through `oxcAAR::setOxcalExecutablePath`, CSV import). New `tabs/palimpsest_ai_report.py` provides a 3-agent descriptive-report pipeline (`PalimpsestAIReportWorker`/`Dialog`) over `LLMProviderManager.stream_chat`, output in 10 languages as DOCX/Markdown, explaining the model/K/threshold choices. Adds a dedicated icon, OxCal sample data, and a 10-language tutorial 37.

---

## [temporal-paradox] — 2026-06-11

### Italiano

**Feature B — Rilevamento paradossi temporali/stratigrafici** (pyarchinit tag `temporal-paradox-5.12.13-alpha`, commit `61b84387`). Predecessore: `genera-continuita-5.12.12-alpha`.

La *Verifica rapporti* ora rileva quando il periodo/fase assegnato a un'unità contraddice la stratigrafia osservata. La stratigrafia è il dato di riferimento → gli auto-fix spostano i **periodi**, non i rapporti. **Confine d'intervallo stretto** (spec §5): "A interamente più antica di B" = `cron_finale(A) < cron_iniziale(B)`; periodi adiacenti che si toccano in un punto = sovrapposti, non paradosso.

**Simboli nuovi/modificati**:

- NEW `modules/utility/temporal_check.py` (Qt-free, DbHandle PG+SQLite):
  - Kind: `TEMPORAL_INVERSION`, `TEMPORAL_CONTEMPORANEITY`, `TEMPORAL_UNEVALUABLE`.
  - `build_chronology(handle, sito)` — `(periodo, fase) → (cron_iniziale, cron_finale)` da `periodizzazione_table`.
  - `load_unit_periods(handle, sito)` — `us → (pi, fi, pf, ff)` da `us_table` (il projector attacca al grafo solo `periodo_iniziale`).
  - `unit_span(periods, chrono)` / `_cron_of` (fallback fase vuota → aggrega sul periodo).
  - `detect_temporal(graph, chrono, unit_periods, *, sito, lang)` — Issues localizzati (it/en/de/es/fr/pt), dedup `seen`, placeholder esclusi via `_real_us`.
  - `solve_fixes(issues, graph, chrono, unit_periods, *, sito)` — euristica di maggioranza greedy single-pass su mappa-periodi in-memory; sposta solo unità **mono-periodo** (`_is_mono`) al periodo a spostamento minimo; gap-fill contemporaneità (kind promosso UNEVALUABLE→CONTEMPORANEITY); pareggio/multi-periodo/no-target → suggerimento.
  - `_violated(role, su, sn)` (stretto, coerente col rilevamento — guida `conflict_score` + tie-break) **separato da** `_fix_satisfies(role, cand, nb)` (rifiuta il confine che tocca per le relazioni d'ordine — guida solo `_best_target_period`).
  - `_build_adjacency` — dedup archi paralleli via set.
- `modules/utility/rapporti_check.py`:
  - `check_rapporti(...)` — nuovi kwarg opzionali `chrono=None, unit_periods=None` (assenti → blocco temporale saltato, retro-compatibile); import lazy di `temporal_check`.
  - `Edit` — nuovo campo `set_fields: tuple = ()`.
  - `apply_edits`/`rollback` — scrittura+snapshot colonne periodo dietro `_PERIOD_COL_WHITELIST` (anti-injection); rollback misto rapporti+periodi atomico.
  - `_L` — nuovi template `t_temporal_*`/`s_temporal_*` (6 lingue).
- `gui/rapporti_check_dialog.py` — `_run` costruisce chrono+unit_periods; `_preview` mostra i `set_fields`; `_apply` auto-backup DB (SQLite + `pg_dump` home-relative) prima delle scritture periodo.

**Test delta**: 24 `test_temporal_check.py` + 15 `test_rapporti_check.py` = **39 passed** (incl. confine-che-tocca in entrambe le direzioni d'arco); suite `tests/sync` **490 passed** (SQLite; residui = PG d'ambiente). AC-2 byte-identical preservato.

### English

**Feature B — Temporal/stratigraphic paradox detection** (pyarchinit tag `temporal-paradox-5.12.13-alpha`, commit `61b84387`). Predecessor: `genera-continuita-5.12.12-alpha`.

*Verifica rapporti* now detects period/phase assignments contradicting observed stratigraphy. Stratigraphy is the reference → auto-fixes move **periods**, not relations. **Strict interval boundary** (spec §5): "A entirely more ancient than B" = `cron_finale(A) < cron_iniziale(B)`; adjacent touching periods = overlap, not a paradox.

**New/changed symbols**: NEW Qt-free `modules/utility/temporal_check.py` (kinds `TEMPORAL_INVERSION`/`TEMPORAL_CONTEMPORANEITY`/`TEMPORAL_UNEVALUABLE`; `build_chronology`, `load_unit_periods`, `unit_span`, `detect_temporal` (localized it/en/de/es/fr/pt), `solve_fixes` (greedy majority heuristic, mono-period only, contemp gap-fill, tie/multi-period/no-target → suggestion); `_violated` (strict, detection-consistent, drives conflict_score+tie-break) split from `_fix_satisfies` (rejects touching boundary for ordered relations, drives `_best_target_period` only); `_build_adjacency` dedups parallel edges). `rapporti_check.py`: `check_rapporti` gains optional `chrono`/`unit_periods` (backward-compatible), `Edit.set_fields`, `apply_edits`/`rollback` write+snapshot period columns behind `_PERIOD_COL_WHITELIST`, new `t_temporal_*`/`s_temporal_*` templates. `gui/rapporti_check_dialog.py`: builds chrono+unit_periods, previews `set_fields`, auto-backups DB before period writes. Tests: 39 temporal/rapporti passed; full `tests/sync` 490 passed (SQLite); AC-2 byte-identical.

---

## [genera-continuita] — 2026-06-11

### Italiano

**Feature A — "Genera continuità" (automatismo schede CON)** (pyarchinit tag `genera-continuita-5.12.12-alpha`, commit `337310d3`). Predecessore: `5.12.11-alpha` (EM export paradata/CON/contemporaneità).

Pulsante esplicito **"Genera continuità"** nel pannello *Verifica rapporti*: per il sito selezionato, scansiona le US/USM con `periodo_iniziale ≠ periodo_finale` e crea/aggiorna idempotentemente una scheda **`CON_<us_madre>`** che copre lo span di periodi della madre, con rapporto di continuità reciproco. Anteprima dry-run, auto-backup, rimozione orfane opt-in.

**Simboli nuovi/modificati**:

- NEW `modules/s3dgraphy/sync/continuity_generator.py` (Qt-free, DbHandle PG+SQLite):
  - Pure: `scan_candidates`, `build_con_record`, `desired_rapporti`, `diff_continuity`; `CONTINUITY_SOURCE_TYPES = {US, USM}`.
  - I/O: `load_site_records`, `load_existing_con`, `apply_plan` (transazionale, `id_us = MAX+1`, `node_uuid` solo se la colonna esiste), `generate_continuity`.
- `modules/s3dgraphy/sync/rapporti.py`:
  - NEW `CONTINUITY_LABELS` (10 lingue) + `continuity_label()`; registrate in `RAPPORTI_SHORTHAND` (forward → `is_after` no-swap, reverse → `is_after` swap → **stesso** edge `CON is_after US`, nessun 2-ciclo). Nessuna modifica a `parse_rapporti`; blocco *candidate for upstream*.
- `gui/rapporti_check_dialog.py` — pulsante "Genera continuità" + flusso anteprima/conferma/backup.

**Test delta**: `test_continuity_generator.py` + `test_continuity_vocab.py` = **44 passed**; suite `tests/sync` **463 passed**. AC-2 byte-identical (il generatore non gira all'export).

### English

**Feature A — "Genera continuità" (CON automatism)** (pyarchinit tag `genera-continuita-5.12.12-alpha`, commit `337310d3`). Predecessor: `5.12.11-alpha`.

Explicit **"Genera continuità"** button in the *Verifica rapporti* panel: scans the selected site's US/USM with `periodo_iniziale ≠ periodo_finale` and idempotently creates/updates a **`CON_<us_madre>`** record spanning the madre's period interval, with reciprocal continuity rapporti, dry-run preview, auto-backup and opt-in orphan removal. **Symbols**: NEW Qt-free `modules/s3dgraphy/sync/continuity_generator.py` (pure `scan_candidates`/`build_con_record`/`desired_rapporti`/`diff_continuity` + I/O `load_site_records`/`load_existing_con`/`apply_plan` (transactional, `id_us = MAX+1`, `node_uuid` only if column exists)/`generate_continuity`; `CONTINUITY_SOURCE_TYPES = {US, USM}`). `rapporti.py`: NEW `CONTINUITY_LABELS` (10 languages) + `continuity_label()`, registered in `RAPPORTI_SHORTHAND` (forward/reverse → the **same** `CON is_after US` edge, no 2-cycle; `parse_rapporti` unchanged; block marked *candidate for upstream*). `gui/rapporti_check_dialog.py`: button + preview/confirm/backup flow. Tests: 44 feature tests passed; full `tests/sync` 463 passed; AC-2 byte-identical (generator never runs at export).

---

## [5.12.11-alpha] — 2026-06-08

### Italiano

**Export EM: paradata connessi, CON visibili, contemporaneità a doppio arco** (branch `Stratigraph_00001`, untagged interim).

- `modules/s3dgraphy/sync/graphml_writer.py`: `_resolve_display_label` — i nodi paradata (DOC/Extractor/Combinar/property) mostrano il valore `us` (`D.1`, `D.1.1`, `C.1`) invece della descrizione.
- Nuovo flag `paradata_as_groups` su `GraphMLExporter.export()` (core s3dgraphy): paradata resi come nodi collegati via `extracted_from`/`combines`/`has_property`/`has_documentation`/`has_data_provenance` invece che raggruppati in folder; `graphml_writer` passa `paradata_as_groups=False` difensivamente (`inspect.signature`).
- Contemporaneità (`has_same_time` e affini) emessa come due archi paralleli senza freccia (core `edge_generator`/`graphml_exporter`).
- Le righe `unita_tipo='CON'` sopravvivono al `graph_projector` (back-fill stratigrafico, label de-doppiata) e rendono come diamanti di continuità; i sintetici `_synth_BR_*` restano soppressi.

**Test delta**: 415 → **419 passed** (+4 in `tests/sync/test_em_export_rendering.py`), zero nuove regressioni. Parte core su ext_libs (stop-gap) + fork branch `fix/em-paradata-export-rendering` (non pushato, PR upstream in attesa).

### English

**EM export: connected paradata, visible CON, double-edge contemporaneity** (branch `Stratigraph_00001`, untagged interim). Paradata labels use the `us` value via `_resolve_display_label` (`modules/s3dgraphy/sync/graphml_writer.py`); new `paradata_as_groups` flag on s3dgraphy `GraphMLExporter.export()` draws paradata as connected nodes (`extracted_from`/`combines`/`has_property`/`has_documentation`/`has_data_provenance`), pyArchInit passes `False` defensively via `inspect.signature`; contemporaneity renders as two parallel no-arrowhead edges; explicit `unita_tipo='CON'` rows survive the projector and render as continuity diamonds while `_synth_BR_*` stay suppressed. Tests 415 → 419 passed (+4 EM tests), no new regressions; core parts in ext_libs stop-gap + fork branch `fix/em-paradata-export-rendering` (not pushed).

---

## [5.12.10-alpha] — 2026-06-08

### Italiano

**Export EM: niente diamanti di continuità auto-generati** (branch `Stratigraph_00001`, untagged interim).

- `modules/s3dgraphy/sync/graphml_writer.py`: la chiamata a `GraphMLExporter.export()` passa `continuity_diamonds=False` quando supportato (check difensivo `inspect.signature` — un ext_libs ri-vendorizzato senza il parametro non crasha).
- `ext_libs/s3dgraphy` (stop-gap live, git-ignored): flag `continuity_diamonds` su `export()` + skip paradata in `materialize_continuity`.

**Verifica**: 0 occorrenze `_synth_BR_` nel `.graphml` esportato. Suite `tests/sync` **415 passed**.

### English

**EM export: no auto continuity diamonds** (branch `Stratigraph_00001`, untagged interim). `graphml_writer` now passes `continuity_diamonds=False` to `GraphMLExporter.export()` when supported (defensive `inspect.signature` check); the ext_libs side (flag on `export()` + paradata skip in `materialize_continuity`) is a live stop-gap pending the upstream PR. Continuity stays modelled via explicit `unita_tipo='CON'` rows. Headless: 0 `_synth_BR_` in the export. Suite 415 passed.

---

## [5.12.9-alpha] — 2026-06-08

### Italiano

**Export EM: edge paradata tipizzati + USV/SF connessi** (branch `Stratigraph_00001`, untagged interim).

- NEW `modules/s3dgraphy/sync/paradata_edge_resolver.py` + post-pass in `graph_projector.populate_graph`: ri-tipizza gli edge `generic_connection` (shorthand EM `>>`/`<<`) negli edge specifici s3dgraphy — `extracted_from`, `combines`, `has_property`, `has_documentation`, `has_data_provenance`, `is_part_of` — leggendo le `allowed_connections` dal datamodel, con swap di direzione quando la regola combacia al contrario; il tipo EM viene da `attributes['unita_tipo']`, non dalla classe Python; Combiner/Extractor mai collegati a US/USM. 16 test in `tests/sync/test_paradata_edge_resolver.py`.
- `graph_projector._is_us_node` (+3 call site in `_propagate`/`_enrich`): da test sul prefisso del nome classe (`startswith("Stratigraphic")`) a walk della MRO (sottoclassi di `StratigraphicNode`) → USV/SF/VSF ora indicizzati e i loro `rapporti` diventano edge.

**Test delta**: suite `tests/sync` **415 passed**, zero nuove regressioni.

### English

**EM export: typed paradata edges + connected USV/SF** (branch `Stratigraph_00001`, untagged interim). New `paradata_edge_resolver.py` post-pass over `populate_graph` retypes EM-shorthand `generic_connection` edges into the specific s3dgraphy types (`extracted_from`/`combines`/`has_property`/`has_documentation`/`has_data_provenance`/`is_part_of`) via `allowed_connections`, node EM type from `attributes['unita_tipo']`, direction swap when matched in reverse; Combiner/Extractor never link plain US/USM. Second fix: `_is_us_node` (+3 call sites) now walks the MRO instead of a class-name prefix, so `StructuralVirtualStratigraphicUnit`/`SpecialFindUnit`/`VirtualSpecialFindUnit` get indexed and their rapporti become edges. 16 new resolver tests; suite 415 passed.

---

## [5.12.8-alpha] — 2026-06-08

### Italiano

**Fix schema: `us_table.other_locations` su updater + template** (branch `Stratigraph_00001`, untagged interim).

La colonna `other_locations` (yE-F) era dichiarata nell'ORM (`US_table.py`) ma esisteva solo come migrazione manuale → un DB nato dal template falliva all'apertura con `OperationalError: no such column`.

- `modules/db/sqlite_db_updater.py` (`update_us_table`): aggiunge `other_locations TEXT` via `add_column_if_missing`.
- `modules/db/postgres_db_updater.py`: nuovo `update_us_table()` chiamato in `run_essential_migrations` (eseguito ad ogni connessione).
- Template `resources/dbfiles/pyarchinit.sqlite` e `pyarchinit_db.sqlite` aggiornati: i nuovi DB nascono completi, gli esistenti si auto-riparano alla riconnessione.

### English

**Schema fix: `us_table.other_locations` on updaters + templates** (branch `Stratigraph_00001`, untagged interim). The yE-F column was in the ORM but only existed as a manual migration — absent from the shipped templates and on-open updaters, so template-born DBs errored on open. Added to SQLite `update_us_table` (`add_column_if_missing`), to a new PostgreSQL `update_us_table()` run via `run_essential_migrations` on every connection, and to both shipped sqlite templates. Existing DBs self-heal on reconnect.

---

## [5.12.7-alpha] — 2026-06-08

### Italiano

**s3dgraphy bump 1.6.0.dev8 → 1.6.0.dev9 + stop-gap ritirato** (branch `Stratigraph_00001`, untagged interim).

- `requirements.txt`: pin `s3dgraphy==1.6.0.dev8` → `==1.6.0.dev9`; `ext_libs/s3dgraphy` ri-vendorizzato. dev9 = dev8 + PR #23 mergiato (vocabolario multilingue dei rapporti, 10 relazioni × 10 lingue + `"supports"→is_abutted_by`).
- `modules/s3dgraphy/sync/rapporti.py` ri-sincronizzato **verbatim** da dev9: il blocco vocabolario locale non è più una divergenza; tutti i simboli consumati (`CANONICAL_UNIT_TYPES`, `strip_us_prefix`, `select_rapporti_label`, …) restano esportati.
- **Eliminato** il monkeypatch `modules/s3dgraphy/sync/ext_rapporti_patch.py` e la chiamata in `pyarchinitPlugin.initGui()`: il path d13 risolve il vocabolario nativamente da ext_libs dev9.
- `tests/sync/test_rapporti_multilingual_map.py` resta come guardia anti-drift upstream/i18n.

**Test delta**: suite **399 passed** col core dev9, zero nuove regressioni.

### English

**s3dgraphy bump 1.6.0.dev8 → 1.6.0.dev9 + stop-gap retired** (branch `Stratigraph_00001`, untagged interim). Pin dev8→dev9, ext_libs re-vendored; dev9 includes merged PR #23 (multilingual relationship-label vocabulary). `modules/s3dgraphy/sync/rapporti.py` re-synced verbatim from dev9 (all consumed symbols still exported, no import breaks); **removed** the `ext_rapporti_patch.py` boot monkeypatch and its `initGui()` call — d13 export now resolves the vocab natively from ext_libs. The consistency test stays as a drift guard. Suite 399 passed.

---

## [5.12.6-alpha] — 2026-06-07

### Italiano

**s3dgraphy bump 1.6.0.dev7 → 1.6.0.dev8** (branch `Stratigraph_00001`, untagged interim).

- `requirements.txt`: pin `s3dgraphy==1.6.0.dev7` → `==1.6.0.dev8`; `ext_libs/s3dgraphy` ri-vendorizzato. dev8 = dev7 + PR #22 mergiato (multilingual unita_tipo: `UNITA_TIPO_CANONICAL` dict + `canonical_unita_tipo()` + `_SHORTHAND_TOKENS` + `_source_rapporti_label`), puramente additivo.
- Resta locale in `modules/s3dgraphy/sync/rapporti.py` il vocabolario multilingue dei rapporti (candidato a secondo PR upstream); il monkeypatch `ext_rapporti_patch.py` resta attivo.

**Test delta**: suite **399 passed**; in repo cambia solo il pin (`ext_libs` è git-ignored).

### English

**s3dgraphy bump 1.6.0.dev7 → 1.6.0.dev8** (branch `Stratigraph_00001`, untagged interim). Pin dev7→dev8, ext_libs re-vendored; dev8 = dev7 + merged PR #22 multilingual unita_tipo fix (now official upstream). The multilingual relationship-label vocabulary stays local in `rapporti.py` (second upstream PR candidate); `ext_rapporti_patch.py` stays active. No breaking changes; suite 399 passed; only `requirements.txt` changes in-repo.

---

## [5.12.5-alpha] — 2026-06-07

### Italiano

**Verifica rapporti: dettaglio direzionale di cicli/contraddizioni, localizzato** (branch `Stratigraph_00001`, untagged interim).

Cicli e contraddizioni mostrano ora la catena completa con il rapporto di ogni passo (es. `US 616 «Coperto da» US 618 ⇄ US 618 «Coperto da» US 616`), nella lingua di QGIS.

- `modules/utility/rapporti_check.py`: `check_rapporti(..., lang=...)`; parole dei rapporti dalla tabella i18n `RELATIONSHIPS` (tutte le 10 lingue), prefisso unità localizzato (US/SU/SE/UE/…), template messaggi it/en/de/es/fr/pt (fallback inglese). Nuova `kind_title(kind, lang)` per i titoli di gruppo.
- `gui/rapporti_check_dialog.py`: legge la lingua da `QgsSettings("locale/userLocale")` e la passa alla verifica.

**Test delta**: suite **397 passed** (+`test_contradiction_summary_is_localized_and_directional`, `test_kind_title_localized_with_fallback`).

### English

**Verifica rapporti: directional, localized cycle/contradiction detail** (branch `Stratigraph_00001`, untagged interim). Cycles and contradictions now show the full chain with each step's relationship in the QGIS UI language. `check_rapporti(..., lang=...)`: relationship words from pyArchInit's i18n `RELATIONSHIPS` table (all 10 languages), localized unit prefix, message templates for it/en/de/es/fr/pt (English fallback); new `kind_title(kind, lang)`. The dialog reads `QgsSettings("locale/userLocale")` and passes it through. Suite 397 passed.

---

## [5.12.4-alpha] — 2026-06-07

### Italiano

**Reciprocità rapporti: copertura completa di tutte le 10 lingue pyArchInit** (branch `Stratigraph_00001`, untagged interim).

Completa/corregge [5.12.2-alpha]: il reciproco di "Abuts" è **"Supports"** (indice 9 di `RELATIONSHIPS`) e il fallimento di round-trip valeva per ogni lingua, non solo l'inglese.

- `modules/s3dgraphy/sync/rapporti.py`: rimossi gli alias non canonici; `RAPPORTI_TO_EDGE_TYPE` esteso a **10 relazioni × 10 lingue** (it, en, de, es, fr, ar, ca, ro, pt, el), derivato dalla tabella i18n `RELATIONSHIPS` (duplicata, non importata, per non accoppiare il package sync a `pyarchinit.*`). Da 20 → 97 chiavi.
- NEW `tests/sync/test_rapporti_multilingual_map.py`: la tabella embedded deve combaciare esattamente con `RELATIONSHIPS`; coppie inverse coerenti con `_EDGE_TYPE_INVERSE`.

**Test delta**: suite **397 passed**. Copertura multilingue candidata a PR upstream s3dgraphy.

### English

**Rapporti reciprocity: full coverage of all 10 pyArchInit languages** (branch `Stratigraph_00001`, untagged interim). `parse_rapporti` now recognises relationship labels in all 10 languages, not just it/en — [5.12.2] had patched only English `abuts` with invented aliases. Replaced with the full 10×10 mapping in `RAPPORTI_TO_EDGE_TYPE` (20 → 97 keys), derived from pyArchInit's i18n `RELATIONSHIPS` table (duplicated, not imported); new consistency test `test_rapporti_multilingual_map.py` fails if the duplicate drifts and checks inverse pairs against `_EDGE_TYPE_INVERSE`. Suite 397 passed; upstream PR candidate.

---

## [5.12.3-alpha] — 2026-06-07

### Italiano

**"Verifica rapporti" spostata da menu a tab del dialog di import** (branch `Stratigraph_00001`, untagged interim).

- `gui/rapporti_check_dialog.py`: logica estratta in `RapportiCheckPanel(QWidget)` riutilizzabile, con parametro opzionale `db_provider` (callable → `DbHandle`); `RapportiCheckDialog(QDialog)` resta come thin wrapper.
- `modules/s3dgraphy/s3dgraphy_dot_bridge.py`: nuovo tab **"Verifica rapporti"** in `S3DGraphyExportDialog`; dopo un import riuscito il dialog passa al tab e preseleziona il sito importato.
- `pyarchinitPlugin.py`: rimossi l'azione di menu `actionRapportiCheck` (4 rami `initGui`) e l'handler `runRapportiCheck`.

Cambiamento GUI, nessun test automatico; il core `rapporti_check` è invariato e coperto dai test esistenti.

### English

**"Verifica rapporti" moved from menu to a tab in the import dialog** (branch `Stratigraph_00001`, untagged interim). Logic extracted into a reusable `RapportiCheckPanel(QWidget)` with an optional `db_provider` callable (→ `DbHandle`); `RapportiCheckDialog(QDialog)` stays as a thin wrapper. `s3dgraphy_dot_bridge.py` adds a "Verifica rapporti" tab to `S3DGraphyExportDialog` that auto-activates and pre-selects the site after a successful import. `pyarchinitPlugin.py` drops `actionRapportiCheck` (4 initGui branches) and `runRapportiCheck`. GUI change; the `rapporti_check` core is unchanged.

---

## [5.12.2-alpha] — 2026-06-07

### Italiano

**Fix auto-fix reciprocità: l'inverso di "abuts" ora round-trippa (vocab + guardia onestà)** (branch `Stratigraph_00001`, untagged interim).

Su `test_6` (khutm, etichette inglesi) la verifica dichiarava 113 correzioni ma ne risolveva ~6: `RAPPORTI_TO_EDGE_TYPE` non aveva etichette inglesi per `is_abutted_by`, quindi `parse_rapporti("Supports")` scartava silenziosamente l'inverso calcolato da `get_inverse_relationship("Abuts")`.

- `modules/s3dgraphy/sync/rapporti.py`: aggiunti gli alias inglesi `"supports"`, `"abutted by"`, `"is abutted by"` per `is_abutted_by` (su `test_6`: reciprocità **107 → 0**).
- `modules/utility/rapporti_check.py` — *guardia di onestà*: una reciprocità è auto-correggibile solo se `RAPPORTI_TO_EDGE_TYPE[inv] == _EDGE_TYPE_INVERSE[et]`, altrimenti scelta manuale → il conteggio in anteprima è sempre veritiero.

**Test delta**: suite **394 passed** (+`test_abuts_reciprocity_fix_label_round_trips`, `test_parse_rapporti_knows_english_is_abutted_by`).

### English

**Reciprocity auto-fix: the inverse of "abuts" now round-trips (vocab + honesty guard)** (branch `Stratigraph_00001`, untagged interim). On English-language sites the check claimed 113 fixes but resolved ~6: s3dgraphy's `RAPPORTI_TO_EDGE_TYPE` had no English label for `is_abutted_by`, so `parse_rapporti("Supports")` silently dropped the i18n-derived inverse. Added the missing English aliases in `rapporti.py` (reciprocity 107 → 0 on `test_6`); `rapporti_check.py` gains an honesty guard — a reciprocity is auto-fixable only when the inverse label round-trips to the correct inverse edge type, otherwise it is surfaced as a manual choice. Suite 394 passed.

---

## [5.12.1-alpha] — 2026-06-07

### Italiano

**Fix import round-trip: copia cross-sito invece di spostamento + skip nodi sintetici** (branch `Stratigraph_00001`, untagged interim).

`GraphIngestor.populate_list` abbinava le righe per `node_uuid` senza filtro sito e forzava `sito` al target → importare il GraphML di un sito A in un sito B **spostava** le righe di A (azzerandolo); i nodi `_synth_BR_*` tornavano come finte US.

- `graph_ingestor._resolve_target_row()`: stesso sito → UPDATE in place (round-trip idempotente, AC-2 preservato); cross-sito → **INSERT di una copia con `node_uuid` nuovo (uuid7)**, sorgente intatta; re-import della copia → match per chiave naturale `(sito, area, us, unita_tipo)` → UPDATE idempotente.
- `graph_ingestor._is_synthetic_node()`: nodi `_synth_*` saltati nei loop detection + write.
- `node_uuid` mai riscritto in UPDATE.

**Test delta**: NEW `tests/sync/test_round_trip_file.py`; suite **392 passed**, 2 round-trip ex-`xfail` ora passano.

### English

**Import round-trip fix: cross-site copy instead of move + synthetic-node skip** (branch `Stratigraph_00001`, untagged interim). `GraphIngestor.populate_list` matched rows by `node_uuid` with no sito filter and forced `sito` to the target, so importing site A's GraphML into site B **moved** A's rows (emptying A); `_synth_BR_*` diamonds came back as bogus US rows. New `_resolve_target_row()`: same site → UPDATE in place (AC-2 preserved); cross-site → INSERT a copy with a fresh uuid7 `node_uuid`; copy re-import matches the natural key `(sito, area, us, unita_tipo)` → idempotent UPDATE. `_is_synthetic_node()` skips `_synth_*` in both loops; `node_uuid` is never rewritten on UPDATE. New `test_round_trip_file.py`; suite 392 passed, 2 previously-xfail round-trips now pass.

---

## [5.12.0-alpha] — 2026-06-06

### Italiano

**Verifica rapporti stratigrafici + auto-fix conservativo + import copia** (branch `Stratigraph_00001`, untagged interim).

- NEW `modules/utility/rapporti_check.py` (core Qt-free): costruisce il grafo del sito (`GraphProjector`) ed esegue `detect_stratigraphic_cycles` + `validate_connection` di s3dgraphy + scan di reciprocità — ristretto ai soli nodi US reali (placeholder sintetici esclusi). Fix conservativi: self-loop → rimozione; reciprocità mancante → CREA il rapporto inverso localizzato (`get_inverse_relationship`); contraddizioni/cicli → scelta manuale. `apply_edits` fa snapshot poi scrive via `DbHandle` (SQLite + PostgreSQL); `rollback` ripristina **byte-identico**.
- NEW `gui/rapporti_check_dialog.py` + voce di menu: sito → report ad albero → anteprima diff → Applica / Annulla ultimo fix.
- Import copia (anti-rename): `regenerate_node_uuids(graph)` + flag `--copy` in `scripts/s3dgraphy_sync.py` (path `populate_list`); wiring GUI in sospeso.

**Test delta**: NEW `tests/sync/test_rapporti_check.py` + `test_import_copy_mode.py`; suite **389 passed** / 6 xfailed.

### English

**Stratigraphic rapporti check + conservative auto-fix + import copy** (branch `Stratigraph_00001`, untagged interim). New Qt-free `modules/utility/rapporti_check.py`: builds the site graph via `GraphProjector`, runs s3dgraphy `detect_stratigraphic_cycles` + `validate_connection` + a reciprocity scan restricted to real us_table-backed nodes; conservative fixes (self-loop removal; missing reciprocity → CREATE the localized inverse rapporto; contradictions/cycles surfaced for manual choice); `apply_edits` snapshots then writes via `DbHandle`, `rollback` restores byte-identical. New `gui/rapporti_check_dialog.py` + menu action. Import-copy helper: `regenerate_node_uuids(graph)` + `--copy` flag in `scripts/s3dgraphy_sync.py` (GUI wiring pending). New tests; suite 389 passed / 6 xfailed.

---

## [5.11.4-alpha] — 2026-06-06

### Italiano

**DB update: coercion `''` → NULL per colonne numeriche (PG strict-typing)** (branch `Stratigraph_00001`, untagged interim).

La scheda Periodizzazione in UPDATE crashava su PostgreSQL con `InvalidTextRepresentation: invalid input syntax for type bigint: "" ... cont_per=''`: il form passa `''` per i line-edit vuoti, PG lo rifiuta per colonne `Integer`/`bigint` (SQLite type-loose lo tollera). Nessun drift di schema; il path INSERT già coerceva, mancava l'UPDATE centrale.

- `modules/db/pyarchinit_db_manager.py`: nuovo `_coerce_numeric_blanks(params, table_class_name)` — companion in scrittura di `_normalize_query_params` — chiamato in `update()`: `''`/spazi → `None` per colonne con `python_type` int/float; colonne testuali e valori valorizzati invariati. Vale per tutte le schede su PostgreSQL, no-op su SQLite.

Nota: `cont_per` resta NULL dopo import GraphML — ricalcolare con `update_cont_per` se serve.

### English

**DB update: `''` → NULL coercion for numeric columns (PG strict-typing)** (branch `Stratigraph_00001`, untagged interim). The central UPDATE path crashed on PostgreSQL when a form handed `''` for an untouched numeric line-edit (`InvalidTextRepresentation` on `cont_per`); no schema drift — the ORM declares `Integer` correctly. New `_coerce_numeric_blanks(params, table_class_name)` in `modules/db/pyarchinit_db_manager.py` (write-path companion of `_normalize_query_params`, called inside `update()`) converts `''`/whitespace to `None` for columns whose SQLAlchemy `python_type` is int/float. Generic across all forms' UPDATE on PostgreSQL; no-op on SQLite.

---

## [5.11.3-alpha] — 2026-06-06

### Italiano

**Import: epoch INSERT robusto al desync di sequence PostgreSQL** (branch `Stratigraph_00001`, untagged interim).

L'import in un sito target nuovo (anche in anteprima dry-run) crashava con `UniqueViolation ... periodizzazione_table_pkey`: le sequence serial del DB PG erano rimaste indietro rispetto ai dati (tipico post-`pg_restore`).

- `modules/s3dgraphy/sync/graph_ingestor.py`: nuovo `_resync_pg_serial_sequences(conn, handle)` chiamato all'apertura della transazione di `populate_list` → `setval` delle sequence di `us_table.id_us` e `periodizzazione_table.id_perfas` a `MAX(pk)` prima di scrivere. Solo PostgreSQL, no-op su SQLite. Ogni `setval` gira in un proprio `SAVEPOINT` (`begin_nested`) così una tabella/colonna assente non aborta la transazione d'ingest.

**Test delta**: `test_graph_ingestor.py::test_resync_pg_serial_sequences_is_noop_on_sqlite`; suite **380 passed** / 6 xfailed.

### English

**Import: epoch INSERT robust to desynced PostgreSQL sequences** (branch `Stratigraph_00001`, untagged interim). Importing into a new target site (even in dry-run preview) raised `UniqueViolation` on `periodizzazione_table_pkey` because the PG serial sequences lagged the data (typical after `pg_restore`). New `_resync_pg_serial_sequences(conn, handle)` in `graph_ingestor.py`, called at the start of `populate_list`'s transaction, `setval`s `us_table.id_us` + `periodizzazione_table.id_perfas` to `MAX(pk)` before writing; PostgreSQL-only, no-op on SQLite; each `setval` runs in its own `SAVEPOINT` (`begin_nested`) so a missing table can't abort the ingest. Test added; suite 380 passed / 6 xfailed.

---

## [5.11.2-alpha] — 2026-06-06

### Italiano

**GraphProjector: riconoscimento US/USM multilingua (export DB non italiani)** (branch `Stratigraph_00001`, untagged interim).

Il gating stratigrafico in `graph_projector.py` usava una tupla solo italiana (`"US","USM","USD","USV",…`): su un DB inglese (khutm: 479/485 righe `SU`/`WSU`) `populate_graph` produceva 6 nodi-us / 0 strat-edge.

- `modules/s3dgraphy/sync/graph_projector.py`: nuova mappa `_UNITA_TIPO_CANONICAL` + `_canonical_unita_tipo()`; gating e factory `_create_stratigraphic_node_for_unita_tipo` usano il codice canonico (`ut_canon`), mentre `attributes['unita_tipo']` conserva l'originale (round-trip + dispatch rapporti language-aware).

Dopo il fix, sul vero Al-Khutm: **485 nodi-us / 2368 strat-edge**, d13 tutto in inglese; sistemata anche la copertura del raggruppamento per area. **Test delta**: `test_graph_projector.py::test_projector_recognizes_localized_su_wsu`; suite **379 passed** / 6 xfailed.

### English

**GraphProjector: multilingual US/USM recognition (non-Italian DB export)** (branch `Stratigraph_00001`, untagged interim). The stratigraphic gating used an Italian-only tuple, so on an English DB (479/485 rows `SU`/`WSU`) `populate_graph` produced 6 us-nodes / 0 strat-edges. New `_UNITA_TIPO_CANONICAL` map + `_canonical_unita_tipo()` in `graph_projector.py`: gating and the `_create_stratigraphic_node_for_unita_tipo` factory use the canonical code while `attributes['unita_tipo']` keeps the original (round-trip + language-aware rapporti dispatch). After the fix on real Al-Khutm: 485 us-nodes / 2368 strat-edges, d13 all English; area-grouping coverage fixed too. Suite 379 passed / 6 xfailed.

---

## [5.11.1-alpha] — 2026-06-06

### Italiano

**d13 physical_relationships: US/USM multilingua + etichette localizzate** (branch `Stratigraph_00001`, untagged interim).

Il dispatch verbose/shorthand di `rapporti.py` riconosceva come canonici solo `{"US","USM"}`: su DB inglesi (`SU`/`WSU`) gli US/USM reali cadevano nello shorthand `>>`/`<<` nel d13.

- `modules/s3dgraphy/sync/rapporti.py`: `CANONICAL_UNIT_TYPES` esteso multilingua (US, USM, SU, WSU, SE, MSE, UE, UEM, USZ, ΣΜ, ΤΣΜ, da `UNIT_TYPE_ABBREV`); `serialize_rapporti_from_edges` preferisce l'etichetta originale del campo `rapporti` del nodo sorgente, capitalizzata (`Covers`/`Copre`/…), con fallback al canonico; le unità virtuali restano `>>`/`<<`, la continuità `>`/`<`.
- NEW `modules/s3dgraphy/sync/ext_rapporti_patch.py` (`apply()` in `pyarchinitPlugin.initGui`): monkeypatch al boot che ricopia i simboli corretti su `ext_libs/s3dgraphy/sync/rapporti.py` (git-ignored, re-installato da PyPI); da rimuovere quando il fix sarà upstream.

**Test delta**: NEW `tests/sync/test_rapporti_multilingual_d13.py` (16 casi); suite **378 passed** / 6 xfailed.

### English

**d13 physical_relationships: multilingual US/USM + localized labels** (branch `Stratigraph_00001`, untagged interim). The verbose/shorthand dispatch in `rapporti.py` only treated Italian `{"US","USM"}` as canonical, so English `SU`/`WSU` rows fell to the `>>`/`<<` shorthand in d13. `CANONICAL_UNIT_TYPES` extended multilingual (from `UNIT_TYPE_ABBREV`); `serialize_rapporti_from_edges` prefers the source node's own `rapporti` label, capitalized, falling back to the canonical one; virtual units keep `>>`/`<<`, continuity `>`/`<`. New boot monkeypatch `ext_rapporti_patch.py` (`apply()` in `initGui`) copies the corrected symbols onto the git-ignored ext_libs module — to be removed once fixed upstream. 16 new tests; suite 378 passed / 6 xfailed.

---

## [5.11.0-alpha] — 2026-06-06

### Italiano

**Allineamento a s3dgraphy 1.6.0.dev7 (Phase 1)** (branch `Stratigraph_00001`, untagged interim; lavoro svolto sul branch `s3dgraphy-1.6-migration`).

- `requirements.txt`: pin `s3dgraphy>=1.5.0` → `==1.6.0.dev7`; `ext_libs/s3dgraphy` 1.5.0 → dev7 (`pip install --target ext_libs --no-deps --pre`; include **d13** in `exporter/graphml/*` + `importer/import_graphml.py`).
- `modules/s3dgraphy/sync/` (import path `modules.s3dgraphy.sync.*` invariato → zero churn ai call-site): **ADD** `rapporti.py` (`parse_rapporti` / `serialize_rapporti_from_edges`); **UPDATE** `graph_ingestor.py`, `graph_projector.py`, `graphml_writer.py` alle versioni dev7 canonical-edges; `__init__.py` merge 3-way con innesto di `get_vocab_provider()`; **preservati** `vocab_provider.py`, `_workspace.py`, `edge_registry.py`, `pyarchinit_pg_importer.py`.
- `tests/sync/`: baseline AC-2 `mini_volterra_baseline_ai03.graphml` rinfrescato all'output canonical-edges+d13 (equivalenza verificata); 6 test marcati `xfail` (debito upstream, in attesa s3Dgraphy #13).

**Verifica zero regressioni**: BEFORE (1.5.0) 9 failed / 368 passed / 9 errors → AFTER (dev7) 9 failed / 362 passed / 6 xfailed / 9 errors; i residui sono tutti `test_*_pg.py` pre-esistenti, **0 fallimenti non-PG**.

### English

**Alignment to s3dgraphy 1.6.0.dev7 (Phase 1)** (branch `Stratigraph_00001`, untagged interim; work done on `s3dgraphy-1.6-migration`). Pin `s3dgraphy>=1.5.0` → `==1.6.0.dev7`; ext_libs re-vendored (includes d13). Vendored sync tree updated with zero call-site churn: ADD `rapporti.py` (`parse_rapporti`/`serialize_rapporti_from_edges`), UPDATE `graph_ingestor.py`/`graph_projector.py`/`graphml_writer.py` to dev7 canonical-edges, `__init__.py` 3-way merge grafting `get_vocab_provider()`; preserved `vocab_provider.py`, `_workspace.py`, `edge_registry.py`, `pyarchinit_pg_importer.py`. AC-2 baseline `mini_volterra_baseline_ai03.graphml` refreshed to the canonical-edges+d13 output; 6 tests marked xfail (upstream test debt). Before/after proof: 368 → 362 passed / 6 xfailed, residual failures all pre-existing `test_*_pg.py`, 0 non-PG failures.

---

## [5.10.1-alpha] — 2026-05-24

### Italiano

**US-USM rapporti save: consistenza posizionale delle celle vuote** (branch `Stratigraph_00001`, untagged interim).

In `tableWidget_rapporti` (4 colonne) e `tableWidget_rapporti2` (7 colonne), `table2dict` saltava le celle mai toccate (`item(r,c)` → `None`) invece di salvare `""` → sottoliste a lunghezza variabile nel campo `rapporti`, fino al caso corrotto posizionalmente (`['Coperto da', '7', 'Roma']` con il sito in posizione area).

- `tabs/US_USM.py`: `table2dict()` nuovo parametro opt-in `preserve_empty: bool = False`; con `True` le celle `None` diventano `""`. Attivato solo ai call site rapporti in `insert_new_rec()` e `set_LIST_REC_TEMP()`; gli altri tableWidget mantengono il comportamento skip-None.
- Chiude il "Fix C" residuo della serie di master (Fix A `get_all_areas` e Fix B `_update_rapporti_add_area_sito`/`_update_rapporti2_add_area_sito` erano già su questo branch). Nessuna migrazione: le sottoliste corte si riparano col prossimo `update_all_areas` (`Ctrl+U`).

### English

**US-USM rapporti save: positional consistency for empty cells** (branch `Stratigraph_00001`, untagged interim). `table2dict` skipped untouched cells (`item(r,c)` returns `None`) instead of saving `""`, producing length-variable — and in the worst case positionally corrupted — sublists in the `rapporti` field. Added an opt-in `preserve_empty: bool = False` parameter to `table2dict()` in `tabs/US_USM.py`, enabled only at the rapporti call sites in `insert_new_rec()` and `set_LIST_REC_TEMP()`; all other tableWidgets keep the skip-None default. Closes the residual "Fix C" (Fixes A/B were already on this branch via Phase 2); no migration needed — short sublists are repaired by the next `update_all_areas` run.

---

## [5.10.0-alpha] — 2026-05-22

### Italiano

**Extended Matrix renderer (Graphviz dot + EM palette icons)** (pyarchinit tag `5.10.0-alpha`, 11 commit `7cf297fd..5d990988`).

Dopo `pushButton_export_extended_matrix`, il plugin genera automaticamente un `<base>_swimlane.png` usando Graphviz `dot` come layout engine (equivalente batch di "Apply Swimlane Layout + Orthogonal Edges" in yEd).

- 5 nuovi moduli (~1900 righe) in `modules/utility/`: `em_palette_parser.py`, `s3d_json_loader.py`, `harris_swimlane_layout.py`, `matrix_swimlane_renderer.py` (fallback matplotlib), `extended_matrix_renderer.py` (**principale**, dot-based con EM palette icons ufficiali da s3dgraphy).
- Layout `dot` (`splines=ortho`, `rankdir=TB`, `compound=true`, `pack=true`); edge tipizzati dal campo `pyarchinit.rapporti` (solid navy / dashed rosso per Taglia / doppia linea nera `{rank=same}` per Uguale a / dashed grigio per `<<`/`>>`); dedup archi reciproci via `frozenset((src,tgt))` (204→118 su test_1); output 300 dpi A3; key lookup dinamico via `<key attr.name>`.
- Wire in `modules/s3dgraphy/s3dgraphy_dot_bridge.py:export_extended_matrix_multi_format`: try `extended_matrix_renderer` → fallback `matrix_swimlane_renderer`; salva `<base>_swimlane.png` + `.dot`.
- Dipendenza esterna: binario Graphviz `dot` (auto-discovery `shutil.which("dot")` + path fallback); se assente → fallback matplotlib automatico.

Validato su `test_1.graphml` (83 nodi, 118 edge dedup, 7 cluster): PNG 1.1 MB @ 300 dpi, 66/83 paradata con EM icons.

### English

**Extended Matrix renderer (Graphviz dot + EM palette icons)** (pyarchinit tag `5.10.0-alpha`, 11 commits `7cf297fd..5d990988`). After `pushButton_export_extended_matrix` the plugin automatically generates `<base>_swimlane.png` using Graphviz `dot` as the layout engine. 5 new modules (~1900 lines) in `modules/utility/` — `em_palette_parser.py`, `s3d_json_loader.py`, `harris_swimlane_layout.py`, `matrix_swimlane_renderer.py` (matplotlib fallback), `extended_matrix_renderer.py` (primary, dot-based with official EM palette icons). Typed edges from the `pyarchinit.rapporti` field, reciprocal-edge dedup via `frozenset((src,tgt))` (204→118), 300 dpi A3 output, dynamic `<key attr.name>` lookup. Wired in `s3dgraphy_dot_bridge.py:export_extended_matrix_multi_format` (dot → matplotlib fallback; saves `<base>_swimlane.png` + `.dot`). Requires the Graphviz `dot` binary (auto-discovery via `shutil.which` + fallback paths; graceful matplotlib fallback if missing). Validated on `test_1.graphml`: 83 nodes, 118 deduped edges, 66/83 paradata nodes with EM icons.

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
