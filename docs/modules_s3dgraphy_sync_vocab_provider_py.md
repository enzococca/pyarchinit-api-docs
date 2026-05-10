# modules/s3dgraphy/sync/vocab_provider.py

## Overview

This file contains 13 documented elements.

## Functions

### _default_bundled_dir()

Locate the bundled JSON_config/ inside ext_libs/s3dgraphy/.

### _default_overrides_dir()

User-writable overrides location.

### get_default_provider()

## `get_default_provider`

## Classes

### VocabProvider

Process-wide vocabulary provider with hot-reload.

**Inherits from**: QObject if _HAS_QT else object

#### Methods

##### __init__(self, bundled_dir, overrides_dir, parent)

## `__init__`

##### _on_directory_changed(self, path)

## `_on_directory_changed`

##### get_unit_types(self, *args, **kwargs)

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

##### versions(self)

## `versions`

