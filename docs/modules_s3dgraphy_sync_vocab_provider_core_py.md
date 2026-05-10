# modules/s3dgraphy/sync/vocab_provider_core.py

## Overview

This file contains 15 documented elements.

## Functions

### _vtuple(s)

Parse a 'M.N.P' version string into a comparable tuple.

**Parameters:**
- `s`

### _first_existing(directory, names)

## `_first_existing`

**Parameters:**
- `directory`
- `names`

### _merge_dicts(base, override)

Per top-level key merge. override beats base; missing keys preserved.

**Parameters:**
- `base`
- `override`

## Classes

### VocabProviderCore

Parses the s3dgraphy JSON pillars; query API for client tools.

#### Methods

##### __init__(self, bundled_dir, overrides_dir, min_versions)

## `__init__`

##### _enforce_minimum_versions(self)

## `_enforce_minimum_versions`

##### versions(self)

## `versions`

##### reload(self)

## `reload`

##### _load_with_override(self, names)

## `_load_with_override`

##### get_unit_types(self, family)

## `get_unit_types`

##### get_edge_types(self)

## `get_edge_types`

##### get_legal_targets_for(self, source_type, edge_name)

## `get_legal_targets_for`

##### get_paradata_types(self)

## `get_paradata_types`

##### get_visual_rule(self, node_type)

## `get_visual_rule`

##### get_cidoc_mapping(self, type_abbreviation)

## `get_cidoc_mapping`

