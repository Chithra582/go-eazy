---
name: property-search-orchestrator
description: Sub-100ms debounced property search indexing, multi-parameter filtering, rolling-window cache pruning, and Zero Layout Shift (ZLS) synchronization.
---

# Property Search Orchestrator Skill

## Overview
The `property-search-orchestrator` skill manages the discovery engine for student renters, supporting real-time debounced queries, multi-criteria filtering (rent, distance, room type), rolling-window data pruning, and Zero Layout Shift (ZLS) client synchronization.

## Core Capabilities
- **Debounced Query Execution**: Optimizes real-time client search input to reduce API overhead while maintaining instant sub-100ms response latency.
- **Multi-Vector Filtering**: Filters listings by geographic proximity to campus, budget range, gender preferences (Boys/Girls PG), and included amenities.
- **72-Hour Cache Pruning**: Orchestrates automated PostgreSQL intervals to prune "Recently Viewed" records after 72 hours, preventing database bloat.
- **Zero Layout Shift (ZLS) Sync**: Delivers pre-calculated image aspect ratios and skeleton layout metadata to eliminate UI jumps during data loading.

## Inputs
- `search_query`: Debounced keyword or geographic landmark.
- `filters`: Price range, room type, campus radius, and amenities.
- `user_id`: Optional identifier to retrieve recent search history.

## Outputs
- `matching_properties`: Array of validated active property cards.
- `result_count`: Total matches discovered.
- `pagination_metadata`: Cursor tokens for infinite scroll loading.
