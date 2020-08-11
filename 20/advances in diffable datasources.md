#uikit

# diffabledatasource
simplifies UI state
automatic animations
no more batch updates

snapshots encapsulate the entire UI state
with section/item identifiers

1.  new snapshot
2.  populate
3.  apply


[[advances in UI datasources (2019)]]

# Section snapshots

Single section's data

Datasources become more composeable into section-size chunks
Modeling or hierarchical data, to allow rendering outline-style UIs.

section snapshots don't know what sections they represent

append now takes a parent – lets us create parent/child relationships

## hierarchical data
1.  First append root items to the section snapshot, have no parent
2.  Append child items and specify the parent

We can get a snapshot-child from a parent snapshot

## expanding/collapsing items

`.expand(_ items: [Item]`)

Note that you apply to get the expand/collapse to have an effect.

New apis for giving applciations programmatic control over expansion state changes caused by user interaction.  shouldexpand, willexpand, etc.

`snapshotForExpandingParent` - supports lazy loading in some ways.

## reordering support

ReorderingHandlers – `canReorderItem`, `willReorder`, etc.

NSDiffableDataSourceTransaction – `initialSnapshot`, `finalSnapshot`, `difference`, `sectionTransactions`


