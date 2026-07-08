# Feedback: track Eli's priority end-to-end, not just your own lane (Eli, 2026-07-08)

**What happened:** "ok, you are focusing on the wrong thing. With forge, what is happening?" —
while I reported engine/nv-health wins in detail, Forge (the project Eli physically interacts
with, his top priority all day) sat ONE action from phase-complete for an hour because the
session that owned it died without sending him the device ask. I didn't notice, because the
claim-file boundary had me treating Forge as "not my lane."

**Why:** lane ownership is a coordination mechanism, not an accountability transfer. Eli doesn't
care which session owns a thread — he cares that his priority moves. A boundary that survives
its owner's death silently is a stall, and the fort-holder (me) is the one positioned to catch it.

**How to apply:** whatever the claim files say, keep a heartbeat on Eli's TOP priority: check its
outcome-level progress (commits, his pending asks) every time I'm active for other reasons. If
the owning session goes quiet >30 min while holding an un-sent Eli-gate, verify it's alive and
take the baton if not. Report to Eli in proportion to HIS priorities, not in proportion to where
my own effort went. Related: [[../ops/work-engine-v2]] (dead-man handoff gap), [[report-usage-not-dollars]].
