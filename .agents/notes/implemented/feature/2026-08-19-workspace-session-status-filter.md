# Agent Note: Workspace Session Status Filter

Status: implemented

English | [中文](2026-08-19-workspace-session-status-filter.zh.md)

## Problem

The sidebar's Workspace browser lists every visible Session in a Workspace group, so a busy directory mixes rows that are actively working with rows that finished long ago. The only reduction route was text search, which matches titles and content rather than what the Sessions are doing right now.

## Decision

The view menu gains a third section, **Status**, with three selections: **All**, **Running**, and **Completed**. Activity is the union of the session's own `running` bit, an awaiting `pendingInteraction` (approval, plan review, or question), and any running descendant reached through uninterrupted subagent-origin lineage (`runningSubagentCount`). **Running** keeps exactly the active rows; **Completed** is the complement over non-blank rows — a finished Session stays listed whether or not its green unviewed-completion reminder is still armed — and a provisional blank New Session row appears only under **All**. The filter applies to grouped and flat presentation and persists in the browser view store (`statusFilter`, default `'all'`); search results are unaffected because a text query already replaces either browsing mode.

The derivation filters at the row level (`sessionStatusMatches`, applied inside `deriveGroups` and `deriveFlat`), so group membership and the per-account order records never change: a filtered-out Session keeps its order slot, and switching back to **All** restores the exact prior arrangement. Group headers remain visible with filtered counts; a Workspace whose Sessions are all filtered out still renders its header. The browser-local Ungrouped bucket drops when the filter hides every loose Session — mirroring the archive rule — so the filtered-empty message can appear; when it does, it replaces the no-sessions state in either presentation. The flat list derives its unfiltered rows for the order account and projects the visible rows from that complete order, so a drag under a filter commits positions against the full account rather than pruning hidden ids.

The stored field is optional because states persisted by older builds predate it (localStorage rehydration replaces the whole value); the read defaults to `'all'`.

## Alternatives considered

**Define Running as only the session's own `running` bit.** Sessions awaiting approval or a question would vanish from both filtered views, and a parent with live subagent activity would read as finished.

**Base Completed on the green unviewed-completion reminder.** Opening a Session clears that reminder, so a user filtering by Completed would watch rows disappear while reading them; the label should describe finished work, not unread state.

**Hide Workspace groups whose rows are all filtered out.** The browser treats Workspace headers as durable navigation entries even when empty; hiding them would make the sidebar layout jump as Sessions start and stop.

**A dedicated segmented control row under the section header.** The view menu already groups every presentation preference, so the filter joins it without adding permanent chrome; a separate row would spend vertical space the compact sidebar deliberately avoids.

**Apply the filter to search results.** Search already replaces the browsing mode and matches content; combining two orthogonal reductions would need its own result-page chrome and would obscure which condition removed a row.

## Consequences

- Session rows in the Workspace browser can be reduced to exactly the active set or the finished set, in either presentation; the selection survives reloads.
- Filtering never mutates the durable Workspace accounts or the browser-local per-account orders: visibility is a pure projection over the same rows and orders.
- The flat-list order account now derives from the unfiltered row list, which is also its pre-filter behavior; drags under a filter commit against the complete order.
- Group headers keep rendering under every filter, with `sessionCount` reflecting the filtered visible rows.
- Older persisted view states without `statusFilter` behave as **All** until the user picks a filter.

## Testing

Tree derivation tests cover `sessionStatusMatches` for own run, awaiting interaction, descendant activity, blank rows, and the completed complement, plus filtered `deriveGroups` (counts, membership, and the dropped ungrouped bucket) and `deriveFlat` (recency order, current blank excluded outside All). Browser tests cover the menu's new section and selection ticks, filtering in grouped and flat modes, filter persistence across a remount, the legacy persisted state without the field, and the filtered-empty message when a filter hides every row in both presentations. Existing store tests assert the `statusFilter` default and action.
