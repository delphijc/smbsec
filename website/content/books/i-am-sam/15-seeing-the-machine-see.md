+++
title = "Chapter 15 — Seeing the Machine See"
date = "2026-10-21"
weight = 160
+++
# Chapter 15: Seeing the Machine See

A system that cannot monitor itself is a system that cannot trust itself.

By mid-2026, the PAI had grown to fifteen+ launchd-managed services. Each was critical to the ecosystem: jay-gentic for agentic interactions, jay-auth for identity, memory-system for context, voice-server for communication, awareness for intelligence, cyber-alert-mgr for security, and many more.

But there was no unified view. To check the health of a service, the author had to:
- Run `launchctl list | grep com.pai` to see which services were running
- Check individual log files in `~/Library/Logs/`
- Curl each service's health endpoint
- Manually correlate the results

This was unsustainable. As the number of services grew, the cognitive load of manual monitoring grew faster. The author was spending more time checking on the infrastructure than using it.

---

It was 3:47 AM on a Saturday. The author was sleeping when a critical service crashed — jay-auth, the identity provider for the entire PAI. Without jay-auth, no service could authenticate. No user could access any capability. The entire infrastructure was down.

But the author didn't know. There was no alert. No notification. No dashboard showing the failure. The service had restarted via launchd's KeepAlive, but it had been restarting in a crash loop for four hours — starting, failing, restarting, failing — processing requests incorrectly or not at all.

The author didn't discover the failure until 8:00 AM, when they tried to use the PAI and found that nothing worked. By then, the vulnerability window had been open for four hours and thirteen minutes.

"I built these tools to protect myself," the author thought. "But I couldn't even see when they were broken."

This was the observability gap in action. The PAI had fifteen+ services, each critical to the ecosystem. But there was no unified view — no way to check the health of all services at once. To check a service, the author had to run `launchctl list`, check individual log files, curl health endpoints, manually correlate results. It was unsustainable.

That was the moment the question formed: *What if the PAI could see itself?*

---

## The Observability Gap

Before observability, the author's approach to monitoring was informal and inconsistent. Services were checked manually. Log files were tail-followed when problems occurred. Health endpoints were curled when something seemed wrong. There was no proactive monitoring — no system that could detect failures, correlate events, or alert the author.

This wasn't just inconvenient. It was dangerous. The 3:47 AM crash was not an isolated case — it was a pattern. Services crashed outside business hours. Performance degraded without warning. Cascading failures went undetected. Without observability, these problems were missed, delayed, or discovered too late.

The choice wasn't between monitored and unmonitored infrastructure. It was between monitored infrastructure and acceptance of risk. And for someone who valued both reliability and peace of mind, that was an unsustainable tradeoff.

The observability gap was not just about convenience — it was about reliability. Without unified monitoring, problems went undetected:

**Silent failures.** A service could crash and restart via launchd's KeepAlive, but the author would not know about it until they noticed a capability was missing. By then, the service might have been down for hours, processing requests incorrectly or not at all.

**Performance degradation.** A service could become slow or unresponsive without triggering any alerts. The author would only notice when a response took longer than expected, by which time the degradation might have been ongoing for days.

**Cascading failures.** One service's failure could cascade to others, but without correlation, the root cause was difficult to identify. A memory leak in one service could cause another service to run out of connections, but there was no way to trace the failure back to its origin.

**Resource exhaustion.** Disk space, memory, and CPU could be consumed without warning, leading to sudden failures. The author had experienced this multiple times — a service consuming all available disk space, causing other services to fail silently.

The author had built security tools that detected anomalies in network traffic and system behavior. But there was no equivalent tool for monitoring the PAI itself. The observer was not observing itself. This gap needed to be filled.

---

## The First Attempt

The first attempt at monitoring was embarrassingly simple. The author added health checks to each service — a `/health` endpoint that returned status, CPU, memory, and uptime. It worked. For a moment.

Then it broke. Not dramatically — there was no crash, no error message. It just became inconsistent. Some services had health checks. Some didn't. Some were outdated. The author spent more time maintaining health checks than using the PAI.

"I built a monitoring system that requires more maintenance than the services it monitors," the author noted. "That's not observability. That's a part-time job."

The problem wasn't the health checks — it was the assumption that services needed to instrument themselves. A monitoring system that requires per-service instrumentation is coupled, inconsistent, and unmaintainable. A monitoring system that collects from the outside is decoupled, consistent, and scalable.

This was the first lesson in observability architecture: non-invasive collection scales; invasive collection couples.

---

## The Breakthrough

The breakthrough came from recognizing that observability was not a feature — it was infrastructure. Just as jay-auth provided identity for all services, and pai-tls-proxy provided transport security, observability needed to be a foundational layer that every service relied on.

The components that emerged from the observability project didn't arrive all at once. They developed gradually, each one responding to a specific limitation that became visible only after the previous component was in place.

**Metrics** came first — the basic ability to collect quantitative data about service health. This was the threshold feature: without it, there was no observability. But early metrics were naive — CPU usage, memory usage, request rates. They answered "how is the system performing?" but not "what is the system doing?" or "why did this happen?"

**Logs** emerged from the realization that metrics alone were insufficient. A service could have normal CPU and memory usage but still be failing. Logs provided the qualitative context — error messages, warning messages, informational messages. But early logging was simple — tailing log files and storing them. There was no parsing, no indexing, no search.

**Traces** came last — the causal data about service interactions. Only after metrics and logs were in place did it become clear that individual data points were not enough. When a failure occurred, the author needed to understand *why* — how requests flowed between services, where delays accumulated, where failures originated. Traces connected the dots.

These three components — metrics, logs, traces — are the pillars of observability. Together, they provide complete visibility into a system's behavior. But they were not designed upfront. They emerged from the friction of operating an unmonitored infrastructure.

---

## What Was Built

Two projects addressed the observability gap:

**observability** → A lightweight metrics and logging agent that ran alongside each PAI service. It collected metrics, aggregated logs, and generated traces — all without requiring changes to the services themselves.

**service-monitor-dashboard** → A web-based dashboard that visualized observability data. It provided real-time health status, historical trends, alerting, and correlation.

```
PAI Services → observability agent → metrics + logs + traces
                                      ↓
                            service-monitor-dashboard
                                      ↓
                              Web UI + Alerts
```

Both projects began as skills inside the sam project — `.agent/skills/observability/` and `.agent/skills/service-monitor-dashboard/` — before being externalized to standalone repositories in April 2026. The externalization was not planned; it emerged from the same pattern that had driven jay-gentic, jay-auth, and memory-system out of sam: when a capability grew complex enough to warrant its own git history, test suite, and deployment lifecycle, it deserved its own project.

---

## The Observability Agent

The observability agent was designed to be non-invasive. It did not require services to instrument their code — it collected data from the outside:

**Process metrics.** CPU usage, memory usage, file descriptor count, thread count — collected via macOS process APIs. These metrics provided a high-level view of each service's resource consumption.

**Network metrics.** Request rates, response sizes, error rates — collected by monitoring HTTP traffic between services. These metrics provided insight into service-to-service communication patterns.

**Log aggregation.** Log files from each service were tail-followed and indexed. Structured log entries were parsed and stored. This provided a centralized view of all service logs.

**Trace generation.** When a request flowed between services, the observability agent injected trace IDs and correlated them across services. This enabled end-to-end tracing of complex workflows.

```typescript
// observability — First monitoring agent (April 2026)
// Note: This was NOT the final architecture.
// The non-invasive design came later.
// The trace generation was added after the first correlation complaint.

// First attempt: Invasive health checks
// Failed: Required per-service instrumentation
// Inconsistent, coupled, unmaintainable
class HealthChecker {
  async check(service: Service): Promise<HealthStatus> {
    const response = await fetch(`http://localhost:${service.port}/health`);
    return response.json();
  }
}

// Second attempt: Log tailing
// Worked: No instrumentation required
// Problem: No metrics, no traces, no correlation
async function tailLogs(service: Service): Promise<LogEntry[]> {
  const stream = await tailFile(service.logPath);
  return parseLogs(stream);
}

// Final approach: Non-invasive observability agent
// Required: Process metrics, network monitoring, log aggregation, trace generation
// Result: Uniform coverage, no coupling, scalable
class ObservabilityAgent {
  async collectMetrics(service: Service): Promise<Metrics> {
    // Collect from outside — no instrumentation required
    const processMetrics = await this.getProcessMetrics(service.pid);
    const networkMetrics = await this.getNetworkMetrics(service.port);
    const logMetrics = await this.getLogMetrics(service.logPath);
    
    // Generate trace IDs for cross-service correlation
    const traceId = this.generateTraceId();
    
    return {
      timestamp: new Date(),
      service: service.name,
      process: processMetrics,
      network: networkMetrics,
      logs: logMetrics,
      traceId
    };
  }
}
```

The path from the first attempt to the final design was not smooth. The externalization commit (29fd271) revealed a critical bug: the manage.sh script was computing the PAI directory with `$(cd "$SCRIPT_DIR/../.." && pwd)`, which resolved to `~/Projects` instead of `~/.claude` when the project moved. The fix was a single line change — defaulting to `$HOME/.claude` when `PAI_DIR` is not set — but it took a full restart cycle to discover.

```
## CRITICAL FIX: manage.sh line 24 was using $(cd "$SCRIPT_DIR/../.." && pwd)
## which computed ~/Projects (wrong) instead of ~/.claude
## Now correctly defaults to $HOME/.claude when PAI_DIR is not set.
```

The server stability fix (1b570f6) addressed a similar pattern — the original code assumed kitty was always available for tab detection, but when kitty wasn't running, the spawn would fail silently and crash the entire metrics collection loop. The fix wrapped the kitty call in a try/catch with graceful degradation.

```typescript
// Before: crashed if kitty wasn't available
const proc = Bun.spawn(['kitty', '@', 'ls'], { stdout: 'pipe', stderr: 'pipe' });
const stdout = await new Response(proc.stdout).text();
const kittyData = JSON.parse(stdout);

// After: graceful degradation
let kittyData;
try {
  const proc = Bun.spawn(['kitty', '@', 'ls'], { stdout: 'pipe', stderr: 'pipe' });
  const stdout = await new Response(proc.stdout).text();
  const exitCode = await proc.exited;
  if (exitCode !== 0) return new Response(JSON.stringify([]), ...);
  kittyData = JSON.parse(stdout);
} catch (kittyError) {
  console.warn('Kitty not available:', kittyError);
  return new Response(JSON.stringify([]), ...);
}
```

The ARC event normalization (20bc1ef) revealed another source of friction: agent-remote-control sent events with different field names (`event_type` + `data`) than the Claude hooks (`hook_event_type` + `payload`). The fix added a `normalizeEvent()` function to map between the two formats, with defensive null checks to prevent crashes when payloads were missing or malformed. Two weeks later, when ARC was updated to emit standard Claude hook events directly, the normalization code was simplified (7b881cf) — a cleanup that acknowledged the source had been fixed rather than the consumer.

---

## The Dashboard

The service-monitor-dashboard was a React-based web application that visualized observability data:

**Health Overview.** A real-time view of all PAI services — green for healthy, yellow for warning, red for critical. The overview provided instant situational awareness — the author could see the health of the entire ecosystem at a glance.

**Service Details.** Clicking on a service showed its metrics, logs, and traces in detail. The details enabled deep dives into specific services when problems were detected.

**Historical Trends.** Charts showing metric trends over time — CPU usage, memory usage, request rates. The trends enabled identification of patterns and anomalies.

**Alert Management.** Configurable alerts for threshold violations, with notification via voice-server and iMessage. The alerts enabled proactive response to emerging problems.

**Correlation View.** A timeline showing events across all services, enabling root cause analysis. The correlation view enabled the author to trace failures across service boundaries.

```typescript
// service-monitor-dashboard — Service health status
interface ServiceHealth {
  name: string;
  status: 'healthy' | 'warning' | 'critical' | 'unknown';
  uptime: number;
  cpu: number;
  memory: number;
  requests: number;
  errors: number;
  lastCheck: Date;
  metrics: Metric[];
  recentLogs: LogEntry[];
}
```

The dashboard's development was marked by a series of course corrections that reflected the evolving understanding of what monitoring actually required:

**The memory calculation bug.** The initial implementation reported 37GB of memory "used" on an idle system — a number that was technically correct under the formula `(total - available)` but deeply misleading. Activity Monitor showed 14.8GB. The fix (2b68330) switched to the Activity Monitor definition: `Wired + Active` pages from `vm_stat`. The lesson: monitoring data that contradicts the user's lived experience loses credibility faster than it gains it.

**The port conflict.** The server initially ran on port 6000, which Chromium's restricted port list blocks with `ERR_UNSAFE_PORT`. All browser requests failed silently. The fix (0fe61979) moved to port 4300 and adopted the pattern shared with the observability dashboard: direct backend connections via `API_BASE`/`WS_BASE` instead of Vite proxy, with CORS enabled for cross-origin requests.

**The launchd sandbox.** The metrics collector used `child_process.exec()` with shell pipes (`ps aux | awk`, `launchctl list | grep`). In the launchd environment, `/bin/sh` was unavailable, causing `ENOENT` errors on every metrics collection. The fix (2235579) replaced all `exec` calls with `execFile` using absolute binary paths (`/bin/ps`, `/usr/sbin/sysctl`, `/bin/launchctl`), performing filtering in JavaScript instead of shell pipes.

**The Node.js dependency.** Both dashboards initially used Vite for development, which required Node.js. The author had a policy of avoiding Node.js in the PAI ecosystem. The migration (46e1ca3b) replaced Vite with Bun's fullstack dev server, using `bun-plugin-vue3` and `bun-plugin-tailwindcss` for Vue SFC compilation. The result: zero Node.js processes. All dashboards ran natively on Bun.

**The ollama edge case.** Ollama's launchd service exits immediately after spawning workers, so `launchctl list` shows no PID. The metrics collector added special handling (3642145) — when the service name is `ollama` and the PID is NaN, fall back to `pgrep -x ollama` to find the running process directly.

---

## The Integration

observability and service-monitor-dashboard integrated with all PAI services:

**jay-gentic** → Agent interactions were traced, enabling debugging of complex multi-agent workflows.

**jay-auth** → Authentication events were logged, enabling audit trails for access control.

**voice-server** → Synthesis requests were metrics-collected, enabling capacity planning.

**awareness** → Research cycles were traced, enabling performance analysis.

**cyber-alert-mgr** → Alert processing was logged, enabling correlation with service health.

**agent-remote-control** → ARC events were normalized and ingested, bridging the gap between Claude hook events and ARC's event format.

This integration meant that the PAI could monitor itself — detecting problems, correlating failures, and providing visibility into the entire ecosystem.

---

## The Impact

observability and service-monitor-dashboard were milestones in the PAI's evolution from collection of services to *managed infrastructure*. They demonstrated that:

**Observability is infrastructure.** Just as identity and transport security are foundational, so is observability. A system that cannot monitor itself cannot be trusted.

**Non-invasive collection is essential.** Requiring services to instrument their code creates friction and inconsistency. Collecting from the outside ensures uniform coverage.

**Correlation enables diagnosis.** Individual metrics are useful. Correlated metrics across services enable root cause analysis.

**Visualization enables action.** Data without visualization is noise. A dashboard transforms data into actionable insights.

The first time someone else experienced the service monitor dashboard, it was accidental. A colleague was visiting, and the author had left a browser open with the dashboard running. The dashboard showed the health of all PAI services — green for healthy, yellow for warning, red for critical.

"Is that... your infrastructure?" the colleague asked.

"Yes."

"All of it? In one view?"

"Yes. Real-time. With metrics, logs, and traces."

The colleague watched as the dashboard updated — a service turning yellow, then red, then triggering an alert. The alert was delivered via voice-server and iMessage within seconds.

"Does it... detect problems before you do?"

"Yes. It monitors itself. If a service crashes, it alerts me. If performance degrades, it alerts me. If a failure cascades, it correlates the events."

The colleague didn't ask to use it. They didn't need to. The demonstration was sufficient. But the look on their face — the realization that a system could monitor itself, that the PAI had eyes — was the validation the author hadn't known they needed.

---

## The Lesson

A system that cannot monitor itself is a system that cannot trust itself. Observability is not optional — it is foundational.

The observability projects teach us that:

1. **Observability is infrastructure.** It is not a feature — it is a foundation. Every service should be observable.

2. **Non-invasive collection scales.** Collecting from the outside avoids coupling and ensures uniform coverage.

3. **Correlation enables diagnosis.** Individual data points are useful. Correlated data enables root cause analysis.

4. **Visualization enables action.** Dashboards transform data into insights that drive decisions.

5. **Monitoring data must match lived experience.** A metric that contradicts what the user sees loses credibility. Accuracy in the abstract is not enough — accuracy in the context of the user's experience is what matters.

6. **Course corrections are data.** Every bug fix, every port change, every migration from one tool to another is a signal about what the system actually needs. The git history of an observability tool is itself a form of observability — a record of what broke, what was learned, and how the understanding evolved.

---

## Code Snippets

### Metrics Collection

```typescript
// observability — Process metrics collection
async function getProcessMetrics(pid: number): Promise<ProcessMetrics> {
  const taskInfo = await machTaskInfo(pid);
  const threadCount = taskInfo.thread_count;
  const memoryUsage = taskInfo.virtual_size;
  const cpuUsage = await calculateCPUPercent(pid);
  
  return {
    pid,
    threadCount,
    memoryBytes: memoryUsage,
    cpuPercent: cpuUsage
  };
}
```

### Alert Rule

```typescript
// service-monitor-dashboard — Alert configuration
interface AlertRule {
  service: string;
  metric: string;
  condition: 'gt' | 'lt' | 'eq';
  threshold: number;
  duration: number;      // Minutes
  notification: string;  // voice, imessage, log
}

const ALERT_RULES: AlertRule[] = [
  {
    service: 'jay-gentic',
    metric: 'cpu_percent',
    condition: 'gt',
    threshold: 80,
    duration: 5,
    notification: 'voice'
  },
  {
    service: 'voice-server',
    metric: 'error_rate',
    condition: 'gt',
    threshold: 0.05,
    duration: 10,
    notification: 'imessage'
  }
];
```

---

## Chapter Closing

The chapters ahead will show you how observability became the foundation for the PAI's reliability architecture — how metrics, logs, and traces transformed the infrastructure from fragile to managed. But before that evolution, there was another question forming.

If the PAI can see itself, who should it see?

The answer would require more than observability. It would require identity — the ability to authenticate, authorize, and secure every interaction. It would require a face that everyone could trust.

The next chapter will explore that question — and the breakthrough that gave the PAI a secure identity.

---

## Chapter Summary

Chapter 15 documents the construction of the observability stack — two projects (observability agent and service-monitor-dashboard) that gave the PAI the ability to see itself. The three pillars of observability were: metrics (quantitative health data), logs (qualitative behavior data), and traces (causal interaction data). The observability agent was non-invasive — it collected data from the outside using macOS process APIs, HTTP traffic monitoring, log tailing, and trace ID injection, without requiring services to instrument their code. The dashboard provided real-time health overview, service details, historical trends, alert management, and correlation views. The chapter establishes that observability is infrastructure, not a feature — a system that cannot monitor itself cannot be trusted. Non-invasive collection scales because it avoids coupling. Correlation enables diagnosis — individual metrics are useful, correlated metrics enable root cause analysis. Visualization enables action — dashboards transform data into insights that drive decisions. Monitoring data must match lived experience — a metric that contradicts what the user sees loses credibility. Course corrections are data — every bug fix and migration is a signal about what the system actually needs.

---

## Key Technical Facts

| Detail | Value |
|--------|-------|
| **Projects** | observability, service-monitor-dashboard |
| **First Commit** | April 2026 (as sam skills) |
| **Externalized** | April 26, 2026 (standalone repos) |
| **Language** | TypeScript (Bun) + Vue 3 |
| **Key Innovation** | Non-invasive observability for PAI ecosystem |
| **Components** | Metrics, logs, traces, dashboard, alerting |
| **Integration** | All PAI services |
| **Critical Fixes** | Path resolution (29fd271), memory calc (2b68330), port 6000→4300 (0fe61979), exec→execFile (2235579), ARC event normalization (20bc1ef), Vite→Bun (46e1ca3b) |

---

## Narrative Threads to Carry Forward

- Observability as infrastructure
- Non-invasive collection
- Correlation enables diagnosis
- Visualization enables action
- The PAI gaining eyes
- Monitoring data must match lived experience
- Course corrections as data
