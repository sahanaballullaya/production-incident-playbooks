# Incident - Banking Website Intermittent Downtime — Faulty DC2 Health Check

**Severity:** SEV-1  
**Environment:** Distributed system, two data centers (DC1 / DC2), load balancer, Splunk, Dynatrace  
**Status:** Resolved

---

## What Happened

Approximately 50% of users experienced intermittent errors on the banking website. The load balancer was splitting traffic evenly between DC1 and DC2, but all connections routed to DC2 were failing. The root cause was a faulty health check on DC2 that was returning stale data — the load balancer believed DC2 was healthy and continued sending traffic to it, causing roughly half of all user requests to fail.

---

## How I Found Out

An increase in error rates was detected in Splunk. Dynatrace confirmed that all connections routed to DC2 were failing while DC1 was healthy. The intermittent nature of the issue — 50% of users affected, not all — was the first clue that the problem was isolated to one data center.

---

## Investigation

Splunk showed a sustained rise in 5xx errors correlating with the start of the incident. Dynatrace pinpointed DC2 as the failing target — every connection attempt to DC2 was timing out or returning errors while DC1 remained fully operational. Investigating DC2 directly revealed the health check endpoint was returning stale cached data, incorrectly reporting the data center as healthy. The load balancer trusted this response and continued routing 50% of traffic to a effectively dead DC2.

---

## How It Was Contained

**Step 1 — Reroute traffic away from DC2**  
Manually updated the load balancer to send 100% of connections to DC1. Error rates dropped immediately, confirming DC2 was the source of all failures and DC1 could handle the full load.

**Step 2 — Validate**  
Monitored Splunk and Dynatrace for several minutes to confirm error rates returned to baseline and no new failures were occurring. DC1 remained stable under full traffic.

**Step 3 — Engage the DC2 vendor**  
Since DC2 infrastructure was managed by a third-party vendor, raised a P1 ticket and got the vendor on a bridge call. Shared Dynatrace screenshots showing all DC2 connections failing and the stale health check response as evidence. The vendor investigated their side and confirmed the health check cache had not been invalidated following a maintenance activity on their end.

**Step 4 — Fix DC2**  
Vendor cleared the stale health check data on DC2. We independently verified the health check endpoint was returning accurate, live data before proceeding.

**Step 5 — Reintroduce DC2 gradually**  
Rebalanced traffic back to 50/50 between DC1 and DC2 in a controlled manner. Monitored both data centers in Dynatrace during the reintroduction to confirm DC2 was handling traffic cleanly before declaring the incident resolved.

---

## Root Cause

The DC2 health check endpoint was serving stale cached data instead of a live system status. The load balancer relied entirely on this health check to make routing decisions and had no secondary validation. Because the health check said DC2 was healthy, the load balancer kept sending traffic to it even as all real connections were failing.

---

## Permanent Fixes Shipped

**Vendor health check requirement** — raised a formal requirement with the DC2 vendor to fix their health check to return live status rather than cached data. This was documented in the incident resolution and tracked as a vendor action item with a deadline.

**Load balancer alerting** — configured the load balancer to alert when traffic distribution between data centers deviates significantly from the expected ratio. A sudden shift to 100% on one DC now triggers an immediate investigation alert.

**Vendor SLA and communication protocol** — established a formal escalation path with the vendor for P1 incidents, including a direct bridge call contact, expected response time, and a requirement that any maintenance activity affecting health check behavior be communicated to our team in advance.

**Runbook** — documented the traffic rerouting steps in the load balancer so any on-call engineer can isolate a data center within minutes without waiting for a senior engineer. Runbook also includes the vendor P1 escalation contact and bridge call process.

---

## Lessons Learned

The health check was the single point of trust for the load balancer's routing decisions, but it was not trustworthy. A health check that returns stale data is worse than no health check — it actively misleads the system into routing traffic to a dead target. The fix was not just clearing the stale data but ensuring the health check can never serve stale data again.

The stepwise recovery — reroute, validate, engage vendor, fix, reintroduce gradually — was the right approach. Jumping straight to rebalancing without validating DC1 stability under full load first would have been a risk.

When infrastructure is vendor-managed, having evidence ready before the bridge call is critical. Sharing Dynatrace screenshots and the stale health check response upfront meant the vendor could diagnose their side immediately rather than spending time reproducing the problem. Clear data shortens vendor resolution time significantly.

---

*Based on a real incident. Some details are generalized.*
