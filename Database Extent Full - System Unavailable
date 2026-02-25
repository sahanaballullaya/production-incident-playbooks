# Progress/Openedge Database Extent Full — System Unavailable

**Severity:** SEV-1  
**Environment:** QAD ERP on Progress OpenEdge, on-premises  
**Status:** Resolved

---

## What Happened

A database extent reached its maximum allocated size during business hours. Progress OpenEdge stops accepting any write operations when an extent is full — users immediately lost the ability to save transactions. Order entry, inventory movements, and work order confirmations all failed. The system was effectively read-only until the extent was expanded.

---

## How I Found Out

A disk utilization alert fired indicating the database extent was at capacity. By the time the alert fired, users were already experiencing errors and had begun calling the help desk.

---

## What I Did to Contain It

Logged into the database server and confirmed the extent was full using the Progress `proutil` database utilities. Added a new variable-length extent to the database structure file (`.st`) and restarted the database broker to pick up the change. Write operations resumed and users were able to log back in and continue their work.

---

## Permanent Fixes Shipped

**Earlier alerting** — lowered the disk utilization alert threshold so it fires at 75% extent capacity instead of at the point of failure. This gives a response window before users are impacted.

**Extent sizing review** — audited all database extents and pre-allocated additional extents with enough headroom to cover 6 months of growth, based on historical data volume trends.

**Runbook** — documented the exact steps to add an extent and restart the broker safely, so the fix can be executed quickly without having to look it up under pressure.

---

## Lessons Learned

The alert threshold was set too close to the limit, leaving no buffer to act before users were affected. The fix itself was straightforward — adding an extent takes only a few minutes — but detection came too late. Moving the alert threshold earlier is the single most impactful change to prevent user-facing downtime from this class of issue.

---
