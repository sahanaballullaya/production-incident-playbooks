QAD ERP Tomcat Crash Due to CPU Exhaustion
Severity: SEV-1
Environment: QAD ERP, Apache Tomcat, on-premises Linux server
Status: Resolved

What Happened
The Tomcat application server hosting QAD crashed due to CPU exhaustion during business hours. Alerts fired, but by the time the on-call engineer could act, the CPU had already spiked to 100% and Tomcat had become unresponsive. Users lost access to the QAD web interface completely until the service was restarted.

How I Found Out
A CPU utilization alert fired on the application server. By the time I acknowledged the alert and logged in to investigate, the CPU was already maxed out and Tomcat had stopped responding to requests. The alert gave warning but not enough lead time to intervene before the crash.

What I Did to Contain It
Confirmed Tomcat was unresponsive via the service status check. Restarted the Tomcat service, which restored user access. While the service was recovering, I checked the Tomcat logs and system metrics to identify what had driven the CPU spike before the crash.

Root Cause
A combination of concurrent user sessions and a long-running background process pushed CPU to 100%. Once at full saturation, Tomcat's thread pool stalled, the JVM garbage collector began thrashing, and the process became unresponsive. The alert threshold was set correctly but the spike was too fast — CPU went from normal to 100% faster than the alert-to-action window allowed.

Permanent Fixes Shipped
Alert threshold lowered — moved the CPU alert from 90% to 70% to create more lead time between the warning and the point of no return. The goal is to have enough runway to investigate and intervene before Tomcat becomes unresponsive.
Tomcat JVM tuning — reviewed and adjusted the JVM heap settings and garbage collection configuration to reduce memory pressure under high load, which was contributing to CPU thrashing at peak utilization.
Auto-restart on failure — configured Tomcat as a systemd service with automatic restart on crash, so even if CPU exhaustion causes a crash, the service recovers without requiring manual intervention. This reduces user downtime from the time it takes an engineer to respond to under 60 seconds.
Runbook — documented the steps to safely restart Tomcat, check logs for the crash cause, and verify all users can reconnect, so any on-call engineer can handle it without escalating.

Lessons Learned
The alert was there but the threshold gave too little time to act. Fast CPU spikes are difficult to catch in time with threshold-only alerting — the bigger fix was making the system self-healing via auto-restart, so the recovery happens automatically regardless of how fast the spike occurs. Alerting alone is not enough for failure modes that move faster than human response time.
