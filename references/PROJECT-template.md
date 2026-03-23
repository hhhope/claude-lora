---
tags: [lore/project]
project: project-name
---

# PROJECT — [project-name]

> Claude reads this in full at every task start.
> Three sections only. Each has a strict definition.
> If content doesn't fit the definition, it doesn't belong here.

---

## Engineering standards

> What Claude must always do in this project, regardless of task.
> Test: "what breaks if Claude ignores this line?"
> If nothing breaks → delete it.

<!--
- All write operations must handle concurrent access
- All external calls must define fallback behavior
- All API responses must include structured error codes
-->

---

## System boundaries

> What this service owns, calls, and must never do.
> Claude needs this to know where its responsibility ends.
> Granularity: service-level, not function-level.

Owns: [e.g. order lifecycle, payment state machine]
Calls: [e.g. payment-service /charge, notification-service /send]
Called by: [e.g. api-gateway, admin-service]
Must never: [e.g. call order-service from payment-service — circular dependency]

---

## Known debt

> Design gaps that exist right now in the codebase.
> Not historical — current state Claude will encounter.
> Test: "would a new engineer be surprised by this?"
> If yes → list it here.

<!--
- user table has no version field — use pessimistic lock for all updates
- payment_log has no index on created_at — avoid range queries
-->
