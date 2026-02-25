# Incident — QAD ERP Slow Performance Due to Table Fragmentation

**Severity:** SEV-2  
**Environment:** QAD ERP, Progress OpenEdge, on-premises  
**Status:** Resolved

---

## What Happened

Users reported sluggish screen response times and slow batch job execution across QAD. Specific transactions that normally completed in seconds were taking significantly longer, impacting productivity across warehouse, finance, and production teams.

---

## How I Found Out

The issue was reported by users experiencing slowness. It was also independently flagged in the routine database health report, which showed a high scatter factor on several key tables. Scatter factor in Progress OpenEdge indicates table fragmentation — records that were once contiguous on disk have become scattered across extents over time, forcing the database to perform excessive I/O to read them.

---

## Root Cause

Table fragmentation had built up over time due to ongoing inserts, updates, and deletes across high-volume QAD tables. The scatter factor on several tables had reached a level where read performance was significantly degraded. Batch jobs that perform large sequential table scans were particularly affected, as fragmented records required far more disk reads to process the same data volume.

---

## What I Did to Contain It

As an immediate measure, all non-essential batch jobs were moved to after-hours to reduce table locking and CPU load during business hours. This relieved pressure on the database while users were active and bought time to plan the full fix properly.

Due to the legacy version of Progress OpenEdge in use, a partial fix was not an option — the fragmentation required a full dump/load/reindex, which involves deleting the existing database and recreating it clean. This is a high-risk, high-downtime operation that cannot be performed during live business hours.

The fix was planned for the Christmas manufacturing shutdown — a 3 to 4 day window when the plant was closed and no users needed system access. This gave a safe buffer to execute the operation without business impact and enough time to recover if any complications arose.

The process involved:

- **Full backup** — complete database backup taken before starting
- **Dump** — full data export from the existing database using the Progress dump utility
- **Delete and recreate** — existing database deleted and a fresh database structure created, eliminating all fragmentation at the storage level
- **Load** — all data reloaded into the new database, writing records cleanly and contiguously
- **Reindex** — all indexes rebuilt from scratch on the clean dataset

Users returned from the holiday break to a fully performant system. Screen response times and batch job runtimes were back to normal with no data loss.

---

## Permanent Fixes Shipped

**Scheduled periodic maintenance** — added dump/load/reindex as a recurring maintenance task on high-volume tables, scheduled during low-activity windows, to prevent scatter factor from building up to a point where it impacts performance.

**DB health report monitoring** — formalized the database health report review as a monthly SRE activity. Scatter factor thresholds were defined — any table exceeding the threshold triggers a scheduled maintenance task before users are impacted.

**Proactive alerting** — added scatter factor checks to the regular database health monitoring so degradation is caught early rather than waiting for user complaints or the monthly report cycle.

---

## Lessons Learned

The scatter factor was visible in the database health report before users felt the impact — but nobody was acting on it proactively. The fix itself was straightforward once identified, but required a maintenance window and caused disruption. Catching it earlier through regular health report reviews means the same maintenance can be scheduled quietly without users ever noticing a slowdown.

---

*Based on a real incident. Some details are generalized.*
