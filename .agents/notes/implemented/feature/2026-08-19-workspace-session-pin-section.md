# Agent Note: Workspace Session Pin Section

Status: implemented

English | [中文](2026-08-19-workspace-session-pin-section.zh.md)

## Problem

A busy Workspace mixes sessions the user is actively watching with everything else, and no ordering preference can keep one Session in view: manual order and activity promotion move rows for other reasons, and the status filter reduces rows but does not hold a place for specific ones.

## Decision

The session row menu gains **Pin** / **Unpin** (a new `IconPinOutline16` product glyph in `dsh-client-ui-primitives`). Pinned Sessions move into a **Pinned** section rendered above every Workspace group — and above the flat list in "In one list" mode — in pin order (newest pin first); the section disappears while it has no visible rows. The section is a browser-local projection: pinning never rewrites the Host Workspace account, the browser-local per-account orders, or the flat-list order account, so unpinning returns a row to its original place. Pinning a Session whose group is folded works like archive and rename (the row menu owns the gesture), and pinned rows keep their rename, fork, and archive actions; they are not draggable inside the section.

The pinned set lives in the workspace view store (`pinnedSessionIds`, pin order, newest first; absent in states persisted by older builds, readers default to none). `derivePinnedSessions` projects the section — pin order over rows that would otherwise be visible: archived, subagent-origin, blank, and status-filtered ids never appear, so the status filter applies uniformly inside the section. `deriveGroups` and `deriveFlat` exclude pinned ids from their rows while every account keeps its slots, and the Ungrouped bucket drops when the filter or the pin set hides every loose Session (the existing archive rule). The no-sessions and filtered-empty messages yield to the Pinned section when it has rows.

## Alternatives considered

**Pin to the top of each Workspace group instead of a global section.** A pinned Session would still scroll out of view inside a long group, and the flat presentation would need a second rule; one section above everything is one mechanism for both presentations.

**Reorder the Host Workspace account on pin.** A browser presentation preference would overwrite the shared durable account, the failure mode the manual-order decision already rejects ([Workspace Sidebar Order and Folding](2026-08-11-workspace-sidebar-order-and-folding.md)).

**Store pins Host-durably (a session-log event like `session/title`).** Cross-client sync would require a new event type, projection, Host RPC, `SessionSummary` field, and both SDK surfaces; the presentation-only consumer does not justify that surface, and the browser-local store already persists across reloads.

**Drag into the Pinned section.** Drag-in would have to invent a pin-from-drop gesture and drag-out an unpin one; the row-menu toggle covers both directions with existing machinery.

**Pin marker on the original row.** The section itself shows pinning; a second indicator would restate the same fact in two places.

## Consequences

- Pinned Sessions are always at the top of the sidebar (grouped or flat), in pin order, until unpinned; unpinning restores the exact prior account position.
- The status filter and archive rules apply inside the Pinned section exactly as elsewhere; pinning is ordering, not visibility.
- The flat-list order account derives from the unpinned row list, so pinning can never prune hidden ids from the persisted order.
- The store field is optional for older persisted states; readers default to an empty set.
- `dsh-client-ui-primitives` exports one more product glyph (71 icons total).

## Testing

Tree derivation tests cover `derivePinnedSessions` pin order, the `pinned` flag, and exclusion of unknown, archived, subagent-origin, blank, and filtered ids; pinned projection out of groups and the flat list with untouched accounts; the dropped Ungrouped bucket; and empty Workspace headers under full pin. Browser tests cover pin/unpin through the row menu with the section above the groups, position restoration on unpin, remount persistence, the flat-list section and account retention, and the status filter inside the section. Row tests cover the pin menu labels and dispatch in both states. Store tests cover the pin-order prepend and removal action, and the icon suite pins the new glyph's count and palette constraints.
