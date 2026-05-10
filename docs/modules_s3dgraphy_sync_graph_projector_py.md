# modules/s3dgraphy/sync/graph_projector.py

## Overview

This file contains 13 documented elements.

## Functions

### compute_primary(memberships, priority_order)

Pick exactly one primary group_uuid per us_id following priority.

memberships: list of dicts with at least these keys:
  - us_id: graph node_id of the US
  - group_uuid: target group node_id
  - group_kind: pyarchinit dimension (struttura, area, ..., toponym)

priority_order: list of group_kind names, highest priority first.
  Toponym is excluded automatically (never primary).

Returns: dict us_id → group_uuid (the primary). US without any
eligible spatial/activity membership get no entry.

**Parameters:**
- `memberships`
- `priority_order`

### _is_us_node(node)

Return True if *node* is a stratigraphic unit (US/USM/USVs/...).

**Parameters:**
- `node`

### _is_epoch_node(node)

Return True if node is an EpochNode. Defensive — avoids importing
EpochNode at module top because it forces s3dgraphy load too early.

**Parameters:**
- `node`

## Classes

### ProjectionError

Read-side failure during GraphProjector.populate_graph().

**Inherits from**: GraphSyncError

### GraphProjector

Stratigraphic-layer projection from PyArchInit DB to s3dgraphy Graph.

Usage:
    projector = GraphProjector()
    graph = projector.populate_graph(db_path, sito="Scavo archeologico")

The graph contains StratigraphicUnit / USM / USVs / USVn / USD /
SF / VSF / CON / DOC / Extractor / Combinar / property nodes plus
EpochNodes for the (periodo, fase) tuples present in the filtered
rows. Edges follow the rapporti column conventions decoded by
`_RAPPORTI_TO_EDGE_TYPE` and `_RAPPORTI_SHORTHAND` in
graphml_writer.py.

#### Methods

##### __init__(self, vocab_provider)

## `__init__`

##### populate_graph(self, db_path, sito, include_paradata, strict_schema, groups, primary_priority)

Build and return a s3dgraphy.Graph populated with the
stratigraphic rows of `sito` from the SQLite at `db_path`.

Args:
    db_path: filesystem path to the pyarchinit SQLite DB.
    sito: site identifier (`us_table.sito` value). Mandatory
        — multi-graph projections are out of scope for AI04.
    include_paradata: when True (default), merge any
        ``paradata.graphml`` produced by :class:`ParadataStore`
        for the (db, sito) pair. When False, return the pure
        stratigraphic layer (backward-compat for AI04 callers
        like ``graphml_writer.export_graphml``). On read errors
        we log a warning and fall back to strat-only — never
        fatal.
    strict_schema: when True (default), require that the
        Phase-1 migration has been applied (i.e.
        ``us_table.node_uuid`` exists) and propagate node-UUID
        attributes onto each StratigraphicUnit so AI04 can do a
        round-trip. When False, skip both the schema check and
        ``_propagate_node_uuid_and_us``: useful for the AI03
        strat-only export path (``graphml_writer.export_graphml``)
        which only needs labels/edges/swimlanes — node_uuid is
        irrelevant there and AC-2 fixtures pre-date the
        migration.
    primary_priority: optional list of dimension names ordered
        from highest to lowest priority for the AI07
        ``is_primary`` selection (compute_primary). When None,
        ``DEFAULT_PRIMARY_PRIORITY`` is used. Toponym is
        always excluded.

Returns:
    A populated `s3dgraphy.Graph`. Empty graph (zero nodes) is
    valid: it just means the site has no rows.

Raises:
    ProjectionError: on any failure reaching the DB or
        instantiating the in-memory graph.

##### _verify_node_uuid_column(self, db_path)

Ensure the Phase-1 migration that added ``us_table.node_uuid``
has been applied. Raises :class:`ProjectionError` otherwise.

Extracted from ``populate_graph`` in AI05 Group C step 2 so the
schema-check is testable in isolation and reusable by any future
method that touches strat tables.

##### _merge_paradata(self, graph, db_path, sito)

Read ``paradata.graphml`` for *sito* and add its nodes to
*graph*.

Non-fatal on read errors — logs a warning and returns. The
caller still gets the strat layer.

De-duplication: nodes whose ``node_id`` already exists on the
target graph are skipped (the strat layer wins). Edges from the
paradata graph are NOT merged here; AI05 Group C does
node-only merging because the paradata graph is currently
author/license/embargo nodes with no connecting edges.

##### _propagate_node_uuid_and_us(self, graph, db_path, sito)

Set attributes['node_uuid'], 'us' and the remaining mapped
columns on each StratigraphicUnit-family node.

Match nodes by `name` (the importer emits name=str(us_table.us))
within the requested sito. Idempotent: re-running yields the
same attribute values.

##### _enrich_into(self, graph, db_path, sito_filter)

Phase 2 / Strategy A — full-class implementation.

Body absorbed verbatim from the now-deleted standalone function
formerly named ``_enrich_pyarchinit_graph`` in ``graphml_writer``.

Bake epoch swimlanes + topological rapporti edges into *graph*.

The vendored s3dgraphy 0.1.40 PyArchInitImporter is incomplete:
it imports only US columns mapped in the JSON mapping (us_table
→ StratigraphicNode + PropertyNodes), and does NOT:

  * read periodizzazione_table → create EpochNodes
  * add ``has_first_epoch`` edges from each US to its periodo
  * parse the ``rapporti`` JSON column → create topological edges

Without those, the GraphMLExporter has no input for swimlanes
and no input for the TemporalInferenceEngine — both AI03
acceptance criteria fail. We perform the enrichment here, in
the orchestrator's filter+enrich layer, so the bridge stays a
one-call surface and so the test fixture remains pure
pyArchInit-shaped data.

Mutates the graph in place. No-op if the DB file lacks the
expected tables.

##### _merge_groups(self, graph, db_path, sito, dimensions, primary_priority)

Materialize group nodes per dimension. AI07: dispatch per
dimension — 6 spatial dims → LocationNodeGroup + is_in_location,
attivita → ActivityNodeGroup + is_in_activity (unchanged).

primary_priority: list[str] of dimension names ordered from highest
to lowest priority for is_primary selection. None = use
DEFAULT_PRIMARY_PRIORITY.

##### _emit_toponym_chain(self, graph, db_path, sito)

AI07 Stage 3: emit a recursive LocationNodeGroup(kind='toponym')
chain from site_table.{nazione,regione,provincia,comune}.

Empty levels are skipped (Q4=c). If all 4 levels are empty, no
chain is emitted.

Cross-site dedupe: each (name, "toponym") pair maps to a
deterministic group_uuid (sha1) so two sites in the same comune
share the node.

Each US in the projected graph gets one is_in_location edge to
the DEEPEST non-empty level (typically `comune`), always
is_primary=false (toponym never primary).

The chain itself is structured top-down via is_in_location edges:
nazione ← regione ← provincia ← comune (each lower level
"is_in_location" of the next level up).

