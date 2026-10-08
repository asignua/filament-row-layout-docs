# Changelog

All notable changes to `asignua/filament-row-layout` are documented here.

## v1.0.0 - 2026-10-08

First release.

- `Table::rowLayout(array $lines, bool|Closure $enabled = true)` and `Table::rowLayoutAside(?string $column, ?string $width = null)`: several lines per record built on the public `Table::content()` seam, so the native column manager (toggle, reorder, session persistence) keeps working inside the lines.
- `Line`: `grow()`, `wrap()`, `gap()`, `muted()`, `size()`, `alignment()`, `visibleFrom()`, `label()`, `inlineLabels()`.
- Layout rules: order inside a line follows the table's column order; unmentioned columns go to the last line; empty lines are not rendered; a column stays in the first line that names it.
- Aside column spanning all lines, with an optional fixed width.
- Header row with one label group per line, sortable labels (direction icon, `aria-pressed`) and the page selection checkbox; on small screens only the first line's labels stay.
- Stock behaviour kept: selection and bulk actions, grouping with collapsible headers, record actions, reorder mode, per-cell column and record URL / action, dark mode, RTL.
- Blocks: `MetaBlock`, `BadgeGroup`, `ProgressBlock`, `ActionsBlock`.
- `RowLayoutPlugin` links the stylesheet in the panel `<head>`; without it the records area loads it on request.
- `enabled:` closure turns the layout off per condition and renders the stock table.
- Translations: English, Ukrainian. Laravel Boost guidelines.
- Not in 1.0: column summaries, sticky header.

### Record details

- Expandable record details: `Line::details()` (ordinary columns behind a chevron, still toggled by the column manager) and `Table::rowDetails(Closure)` (a native Filament schema bound to the record).
- Details load lazily through a Livewire component hook (`$wire.rowLayoutDetails([keys])`, no trait on the page; registered in the provider's register phase and covered by a test through the real HTTP update endpoint), resolved through the table's own query; the table is not re-rendered.
- Expand all / collapse all in the header (one request per page); Filament's `expand-all-table-rows` / `collapse-all-table-rows` events are honoured.
- Opened details survive sorting, refreshes and paging back (page cache).

### Fixes before release

- Fix: Expand all sends the keys in chunks of `ServeRowDetails::MAX_KEYS` through one per-table collector, instead of one Livewire call per record (a page above 50 records hit `max_calls`).
- Fix: `MetaBlock`, `BadgeGroup` and `ProgressBlock` eager-load the relations their paths walk through (no N+1, no `LazyLoadingViolationException`).
- Fix: block eager-loading calls a path segment only when it is an argument-free public method DECLARED to return a `Relation` (and not an attribute, cast or accessor); a helper like `url(string $locale)` no longer throws, `save()`/`touch()`/`delete()` are never run. A relation without a return type is therefore not eager-loaded: declare it.
- Fix: `BadgeGroup` / `MetaBlock` paths through to-many relations (`tags.name`) list every item; `*` is no longer needed.
- Fix: cached details are dropped when the details columns or the table locale change; a record rebuilt while its details were loading reloads them; a failed load closes the record so the next click retries.
- Fix: the group checkbox honours `maxSelectableRecords()`.
- Fix: a `rowDetails()` closure returning `null` means no schema (no empty details box).
- Fix: tables of array records get no chevron and no Expand all / Collapse all buttons.
- Fix: `Line::visibleFrom()` applies to the header and no longer counts in the aside's row span; a column's `visibleFrom()` keeps the cell `inline-flex`.
- Fix: `Line::alignment(Alignment::Left|Right)` takes effect.
- Docs: the header uses `aria-pressed`, not `aria-sort`.

### Static analysis

- PHPStan / Larastan stub and `extension.neon` for the `Table::rowLayout()`, `rowLayoutAside()` and `rowDetails()` macros (picked up by `phpstan/extension-installer`; manual include documented in the README).
