# Architecture

## System Overview

The `bibliograph-storage-meadow` module sits between the Bibliograph record management framework and the database, using Meadow as the data access layer. This allows Bibliograph to store records in any Meadow-supported database without knowing database-specific details.

<!-- bespoke diagram: edit diagrams/system-overview.mmd or .hints.json, then: npx pict-renderer-graph build modules/meadow/bibliograph-storage-meadow/docs -->
![System Overview](diagrams/system-overview.svg)

## Class Hierarchy

```mermaid
classDiagram
	class BibliographStorageBase {
		+generateDeltaContainer(guid)
		+sourceExists(hash, cb)
		+sourceCreate(hash, cb)
		+exists(hash, guid, cb)
		+read(hash, guid, cb)
		+persistRecord(hash, guid, json, cb)
		+persistDelete(hash, guid, cb)
		+readRecordMetadata(hash, guid, cb)
		+persistRecordMetadata(hash, guid, meta, cb)
		+stampRecordTimestamp(hash, guid, cb)
		+readRecordDelta(hash, guid, cb)
		+persistRecordDelta(hash, meta, delta, cb)
		+readRecordKeys(hash, cb)
		+readRecordKeysByTimestamp(hash, from, to, cb)
	}
	class BibliographStorageMeadow {
		+Initialized: boolean
		+MeadowProvider: string
		+meadowSource: Meadow
		+meadowRecord: Meadow
		+meadowDelta: Meadow
		+initialize(fCallback)
		-_findRecord(hash, guid, cb, noDelete)
		-_findDelta(hash, guid, cb)
	}
	BibliographStorageBase <|-- BibliographStorageMeadow
```

## Initialization Flow

<!-- bespoke diagram: edit diagrams/initialization-flow.mmd or .hints.json, then: npx pict-renderer-graph build modules/meadow/bibliograph-storage-meadow/docs -->
![Initialization Flow](diagrams/initialization-flow.svg)

## Record Upsert Flow

All persist operations follow the same upsert pattern:

<!-- bespoke diagram: edit diagrams/record-upsert-flow.mmd or .hints.json, then: npx pict-renderer-graph build modules/meadow/bibliograph-storage-meadow/docs -->
![Record Upsert Flow](diagrams/record-upsert-flow.svg)

## Data Flow: Read vs Write

<!-- bespoke diagram: edit diagrams/data-flow-read-vs-write.mmd or .hints.json, then: npx pict-renderer-graph build modules/meadow/bibliograph-storage-meadow/docs -->
![Data Flow: Read vs Write](diagrams/data-flow-read-vs-write.svg)

## Three-Table Model

<!-- bespoke diagram: edit diagrams/three-table-model.mmd or .hints.json, then: npx pict-renderer-graph build modules/meadow/bibliograph-storage-meadow/docs -->
![Three-Table Model](diagrams/three-table-model.svg)

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Upsert pattern | Simplifies the API -- callers do not need to check existence before writing |
| JSON serialization | Records and metadata are schema-less; the database stores serialized JSON |
| Soft delete | Meadow manages `Deleted`, `DeleteDate`, `DeletingIDUser` fields automatically |
| Epoch ms timestamps | `RecordTimestamp` stores numeric epoch milliseconds for fast range queries |
| SQLite auto-create | SQLite is the default dev/test backend; auto-DDL reduces setup friction |
| Three Meadow entities | Source, Record, and Delta are independent entities with distinct schemas |
| SourceHash isolation | All queries include `SourceHash` to prevent cross-source data leakage |
