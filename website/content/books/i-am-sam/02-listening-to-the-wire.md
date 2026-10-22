+++
title = "Chapter 2 — Listening to the Wire"
date = "2026-10-21"
weight = 30
+++
# Chapter 2: Listening to the Wire

There is a moment every network engineer remembers: the first time they opened a packet capture and realized the network was *alive*.

Not abstract. Not theoretical. Alive. Every handshake, every encrypted exchange, every dropped packet — visible, analysable, *understandable*. For the security architect, this moment is both exhilarating and terrifying. The network reveals everything — if you know how to listen.

It happened on a Tuesday evening. The author was running a routine query through the NotebookLM CLI — a document review task involving sensitive legal briefs. The terminal returned results in seconds, clean and structured. But something in the network monitor caught their eye: a connection to a Google analytics endpoint, firing immediately after the query completed. Not during authentication. Not during token refresh. *After* the query, sending metadata about what had just been asked.

The documents stayed local. The tokens stayed local. But the *questions* — the semantic content of what was being asked — were leaving the machine. And the author had no visibility into what Google was doing with that information.

That was the moment the question formed: *What is my machine actually talking to?*

In May 2025, the author built a tool to listen.

---

## The Question No One Asked

By mid-2025, the author's local AI infrastructure was growing. The NotebookLM CLI was querying documents. Early experiments with local LLMs were running through Ollama. Various Python scripts were orchestrating tasks between them. Each tool was useful. Each was local. Each was *yours*.

But there was a blind spot — one that most people never notice until something goes wrong. **No one knew what the machine was actually doing on the network.**

Was data leaving the machine? Which services were phoning home? What was the actual network footprint of a "local-first" AI setup?

These weren't just technical questions. They were philosophical questions for someone building a sovereignty-focused AI infrastructure. If you claim to run AI locally, but your machine is exfiltrating data to three different cloud providers, what does "local" actually mean?

The author had spent years in security — building systems where every connection was authenticated, every data flow was encrypted, every access was logged and auditable. And here was their own infrastructure, running on their own machine, with less visibility than most production systems. The irony was not lost on them.

So they asked the question that should have been asked from the beginning: *What is my machine actually talking to?*

---

## The Investigation

The investigation began the way all good security investigations begin: by following the traffic.

The first approach was naive. The author ran `lsof -i` and watched the output scroll by — hundreds of connections, most of them to familiar destinations. But the output was static, a snapshot in time, not a continuous stream. It told you what was connected *now*, but not what had connected *before* or what would connect *next*.

Next came Wireshark. The author captured traffic for ten minutes, producing a 400 MB PCAP file. Opening it in Wireshark was like staring into a waterfall — beautiful, overwhelming, and ultimately unhelpful. The relevant connections were buried under thousands of DNS queries, keep-alive pings, and background telemetry. The signal was there, but finding it required sifting through noise that was itself noisy.

The breakthrough came only after stepping back. Instead of trying to capture *everything*, what if the tool captured only what mattered — application-layer traffic, correlated with processes, presented in a way that answered specific questions?

The NotebookLM CLI, which they had proudly built as a "local-first" solution, was making repeated connections to Google's infrastructure. Not just during authentication — *during every query*. The OAuth tokens were local, yes. The documents stayed on disk. But the *intelligence* — the processed responses, the extracted insights — was flowing back through Google's servers.

Ollama was truly local for inference. But the model downloads, the update checks, the telemetry — all of it reached outward. The "local LLM" was local only in the sense that the inference happened on your machine. The ecosystem around it was firmly cloud-based.

And then there were the other services — the ones the author had forgotten about. A Python script that checked for weather updates. A Node.js process that pulled GitHub notifications. A background task that synced configuration to a cloud storage bucket. Each was small. Each was harmless in isolation. But together, they painted a picture of a machine that was far less "local" than its owner believed.

This was the first lesson in network visibility: **what you think is local is not always what is actually local.**

---

## The Tooling Gap

The author knew the existing tools. Wireshark was the gold standard for packet analysis — powerful, comprehensive, and completely overwhelming for routine monitoring. tcpdump was lighter but produced raw output that required significant post-processing. Little Snitch showed connections with process correlation, but it was a commercial product designed for general use, not for the specific questions a security architect needed to answer.

What was needed was something in between — a tool that could:

- Capture network traffic in real-time without requiring root privileges
- Parse application-layer protocols (HTTP, HTTPS handshakes, TLS extensions)
- Correlate connections with local processes
- Present findings in a way that was immediately actionable
- Run as a background service, logging everything for later analysis

The existing tools were either too low-level (raw packet capture) or too high-level (dashboard summaries). What was needed was a tool that operated at the *meaning* layer — understanding what the traffic *meant*, not just what it *looked like*.

The breakthrough came from an unexpected direction.

---

## The "Aha!" Moment

The author had been experimenting with browser-based network observability — the Performance API that powers modern web analytics. These APIs expose detailed information about every network request a browser makes: timing, size, protocol, cache status, even the exact bytes transferred.

What if those same techniques could be adapted to monitor *system-level* network activity?

The insight was elegant. Instead of trying to capture raw packets — which requires root privileges, produces overwhelming data volumes, and demands deep protocol knowledge — build a *proxy-based* observer that intercepts and analyses traffic at the application layer.

The approach was to create a transparent proxy that sat between applications and the network. Every request would pass through the proxy, which would log the destination, the protocol, the size, the timing, and the originating process. No root privileges required. No packet-level complexity. Just clean, structured visibility into what each application was doing.

This was the same principle that had driven the nlm CLI: *don't fight the system — work with it.* Instead of trying to capture packets at the kernel level, work at the application level where the meaning lives.

---

## The Build

The project that emerged was called **net-pack-parser** — a name that reflected its dual nature: it parsed network *packets* at the application layer, but also parsed the *meaning* out of each connection.

The architecture was deliberately hybrid. A Node.js server handled the capture and analysis logic, using the Performance API as its primary data source. A React frontend provided real-time visualization, allowing the author to watch traffic flow as it happened. A PostgreSQL database stored historical data for trend analysis. And Elasticsearch provided full-text search across all captured traffic — because when you're looking for a specific connection hours after it happened, you need more than a timeline.

```typescript
// net-pack-parser — First working version (May 2025)
// Note: This was NOT the final architecture.
// The proxy approach came later after the Performance API failed
// for non-browser processes.

class NetworkCapture {
  // First attempt: Performance API
  // Failed: Only works for browser processes, not system-wide
  async startWithPerformanceAPI() {
    // This only captured browser traffic
    // Missed: Python scripts, Node.js services, background daemons
    this.observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        console.log(entry.name); // Too much noise
      }
    });
  }

  // Second attempt: /proc/net/tcp parsing
  // Worked on Linux, broke on macOS
  async startWithProcParsing() {
    // macOS doesn't have /proc/net/tcp
    // Had to use netstat -p instead
    // Still missed: connections established before capture started
  }

  // Final approach: Transparent proxy
  // Required: Root privileges (or launchd helper)
  // Worked: System-wide, all protocols
  async startWithProxy() {
    // This is what actually shipped
    // See proxy/ directory for implementation
  }
}
```

The process correlation was the trickiest part. The Performance API tells you *what* was requested, but not *which process* made the request. To solve this, the parser maintained a mapping between network ports and local processes, updating it in real-time as connections were established and torn down.

```typescript
// Correlate network activity with local processes
// Note: This was the hardest part to get right.
// The port-to-PID mapping is ephemeral — connections
// appear and disappear faster than any polling interval.
// We ended up using a combination of lsof polling and
// launchd job monitoring to keep the mapping current.
async function correlateProcess(networkEntry) {
  const pid = await findProcessForPort(networkEntry.port);
  if (pid) {
    const { commandLine, cwd } = await getProcessDetails(pid);
    return { pid, commandLine, cwd };
  }
  return null;
}
```

This mapping was built by watching the system's network table — `/proc/net/tcp` on Linux, `netstat -p` on macOS — and correlating local ports with process IDs. When a new connection appeared, the parser could immediately identify which application had initiated it.

---

## The First Capture

The first time net-pack-parser ran, the results were immediate and sobering.

Within the first hour, it logged 847 distinct network connections from 23 different processes. Of those, 612 were to external destinations — services running on other machines, in other countries, under other organizations' control. Only 235 were truly local: loopback traffic between the author's own services.

The breakdown was revealing:

- **Google services:** 312 connections (NotebookLM queries, OAuth token refreshes, Google API calls)
- **GitHub:** 89 connections (API calls, git operations, notification polls)
- **Ollama ecosystem:** 67 connections (model downloads, update checks, telemetry)
- **Various SaaS tools:** 144 connections (slack, notepad, calendar syncs)
- **Truly local:** 235 connections (between PAI services, local databases, local LLM inference)

The "local-first" AI infrastructure was, in reality, a hybrid system — local where it counted (inference, document storage, processing) but cloud-dependent for everything else (authentication, updates, telemetry, integration).

This was not a failure. It was a *finding*. And the difference is everything.

A failure implies that something went wrong. A finding implies that something was *discovered* — something that was hidden, unknown, or misunderstood. The net-pack-parser hadn't broken anything. It had simply made the invisible visible.

Before net-pack-parser, the author operated under a set of assumptions that turned out to be wrong. They believed their "local-first" setup meant their data stayed local. They believed that because documents were stored on disk, they were private. They believed that because authentication was handled locally, the queries were private.

None of these assumptions were malicious. They were simply *unverified*. And in security, unverified assumptions are indistinguishable from vulnerabilities — because they functionally are vulnerabilities.

The parser didn't just reveal traffic patterns. It revealed the gap between intention and reality. That gap is where breaches happen.

---

## The Security Architect's Eye

For a security architect, network visibility is not just about troubleshooting — it's about *threat modelling*. Every connection is a potential attack vector. Every outbound call is a potential data leak. Every unencrypted request is a potential interception point.

The net-pack-parser gave the author the ability to see these vectors clearly for the first time. And what they saw changed how they designed everything that followed.

There was a period — perhaps two weeks — after the first capture when the author found themselves hesitating before running certain commands. A query to NotebookLM. An update check for Ollama. A sync to cloud storage. Each action was followed by an instinctive glance at the net-pack-parser dashboard, watching for connections to appear.

This was not paranoia. It was the natural response of someone who had been handed a new sense — like a blind person suddenly gaining sight. The world hadn't changed. The tools hadn't changed. But the *awareness* of what was happening had shifted fundamentally.

The author realized that most people live with this blindness permanently. They use tools, upload documents, make API calls, and have no idea what leaves their machine. Not because they're careless, but because the tools are designed to hide that information. Net-pack-parser was an act of reclamation — taking back the right to know.

The findings from that first capture session didn't arrive as three clean insights. They came as a series of observations that required time to process, some of which were later revised as more data accumulated.

**On authentication.** The initial reading was straightforward: NotebookLM was sending query metadata to Google. But further investigation revealed nuance — not all queries were equal. Some documents were clearly safe to process through the cloud (public research papers). Others were not (legal briefs, financial models). The insight wasn't "authentication is bad" but "authentication choices require conscious evaluation."

**On telemetry.** Every tool that "phoned home" did so for a reason — crash reporting, usage analytics, update checks. None of this was malicious. But none of it was necessary for the core functionality either. The author began to distinguish between tools that *required* cloud connectivity and those that *preferred* it. The former were accepted. The latter were modified or replaced.

**On the proxy pattern.** If a transparent proxy could provide visibility into application-layer traffic, what else could it do? The same proxy could enforce TLS, filter malicious requests, cache responses, and log everything for audit. The net-pack-parser was not just a monitoring tool — it was the prototype for a broader pattern of *controlled visibility*.

These insights weren't arrived at in a single epiphany. They emerged through iteration — each new capture session refining, sometimes contradicting, the conclusions drawn from the last. Security thinking is not a destination. It's a practice.

---

## The Integration That Followed

The net-pack-parser was initially a standalone monitoring tool. But its insights fed directly into subsequent decisions, shaping the architecture of everything that came after.

**The voice-server** would be designed with network visibility in mind. Rather than assuming that text-to-speech requests stayed local, the architecture explicitly logged every TTS call — what was spoken, when, and to whom. This was direct inheritance from the net-pack-parser philosophy: *if you can't see it, you can't secure it.*

**The pai-tls-proxy** would eventually solve the "cloud dependency" problem by terminating TLS locally. Instead of services making direct HTTPS connections to external endpoints, all traffic would flow through the proxy, which could inspect, log, and control every connection. The net-pack-parser had shown where the leaks were. The proxy would plug them.

**The jay-auth system** would provide the identity framework that made network visibility meaningful. Knowing *that* a connection was made was useful. Knowing *who* made it, *why* it was authorized, and *what* data was exchanged was essential. Identity turned visibility into accountability.

The parser wasn't just a tool — it was the first expression of the **observability-first** principle that would become central to PAI. Every service that followed would be designed with the assumption that it would be monitored, logged, and audited. Not because the author didn't trust their own code, but because *trust without verification is just optimism.*

---

## The Deep Packet Inspection

As net-pack-parser matured, it evolved from a simple connection logger into a full deep packet inspection (DPI) engine. The author added the ability to upload PCAP files — actual packet captures from network interfaces — and analyse them offline.

This capability proved invaluable during security investigations. When a suspicious connection was detected in real-time, the author could capture a PCAP of the surrounding traffic and then upload it to net-pack-parser for detailed analysis. The parser would:

- Reassemble TCP streams to reconstruct full request/response conversations
- Extract strings, URLs, and email addresses from packet payloads
- Identify protocols automatically (HTTP, DNS, SSH, FTP, SMTP, TLS)
- Run YARA rules against payloads to detect malware signatures
- Map detected threats to MITRE ATT&CK techniques

```typescript
// net-pack-parser — Deep packet inspection
interface PacketAnalysis {
  streams: ReassembledStream[];      // Full TCP conversations
  extractedStrings: ExtractedArtifact[];  // URLs, emails, IPs, keys
  protocols: ProtocolIdentification[];  // Auto-detected protocols
  threats: ThreatAlert[];            // YARA matches, signature hits
  mitreMapping: MITRETechnique[];    // ATT&CK technique associations
}
```

The YARA integration was particularly powerful. YARA is a pattern-matching tool originally designed for malware analysis — you write rules that describe byte patterns, and YARA tells you where those patterns appear in a file or stream. The author built a set of custom YARA rules to detect common attack patterns: SQL injection strings, XSS payloads, command injection attempts, known malware signatures.

When net-pack-parser encountered a packet containing one of these patterns, it generated a threat alert with the matching rule, the location in the packet, and a MITRE ATT&CK mapping. This turned the parser from a passive observer into an active threat detector.

---

## The Forensic Value

The real test of net-pack-parser came not from routine monitoring but from a security incident.

A service in the PAI ecosystem began behaving strangely — making unexpected connections to an unfamiliar domain, at unusual hours, with irregular data sizes. The net-pack-parser had been logging everything, so the author could reconstruct the entire timeline:

1. **02:14 UTC** — A background process initiated a TLS connection to `updates.example-cdn.com`
2. **02:14:03** — The TLS handshake completed, using a certificate issued by a CA the author didn't recognize
3. **02:14:05** — An HTTP POST request was sent, containing 4.2 MB of data
4. **02:14:12** — The response was 12 KB, containing a base64-encoded payload
5. **02:14:15** — The payload was decoded and executed as a shell script
6. **02:14:18** — The script established a reverse shell to `irc.example-cdn.com`

Every step was captured. Every byte was logged. The net-pack-parser had recorded the entire attack chain — from initial connection to reverse shell — in structured, searchable format.

The author was able to:

- Identify the compromised service (a forgotten Python script that checked for software updates)
- Extract the malicious payload from the Elasticsearch index
- Run YARA rules against the payload to confirm it matched a known malware signature
- Map the attack to MITRE techniques: T1071 (Application Layer Protocol), T1105 (Ingress Tool Transfer), T1059 (Command and Scripting Interpreter)
- Reconstruct the full kill chain for documentation and remediation

This was the moment when net-pack-parser stopped being a monitoring tool and became a *security instrument*. It wasn't just showing what was happening — it was enabling the author to *understand* what was happening, *respond* to what was happening, and *prevent* it from happening again.

The incident also revealed something deeper about the nature of modern software supply chains. The compromised service was a legitimate update checker — a small Python script that had been downloaded from a repository and forgotten. It had been sitting in a cron job for months, quietly executing code from an external source, when that source was compromised by a threat actor. The net-pack-parser hadn't prevented the initial compromise — that would have required a different kind of tool — but it had made the compromise *visible* in real-time, enabling rapid detection and response.

In the security world, this is the difference between prevention and detection. Prevention is ideal but impossible — no system is impervious to compromise. Detection is practical and essential — every system will be tested, and those that can detect and respond to attacks will survive. The net-pack-parser was a detection tool, and its value was proven not by preventing an attack but by making it impossible to hide.

---

## The Philosophy of Visibility

The net-pack-parser taught a lesson that would echo through every subsequent project: **visibility is the foundation of security.**

Before you can protect your AI infrastructure, you need to understand what it's actually doing — not what you *hope* it's doing, but what it's *actually* doing on the network. Hope is not a security strategy. Verification is.

Three principles emerged from the work:

**Assume opacity.** Don't trust that "local" means "private." Don't trust that "open source" means "safe." Don't trust that "your machine" means "your control." Verify everything. Measure everything. Log everything. The net-pack-parser didn't assume that the PAI was secure — it assumed that the PAI's network behaviour was *unknown* until proven otherwise.

**Build your own visibility.** Commercial tools are designed for general use. They show you what their developers think you need to see. But every infrastructure is unique. Every security posture is different. Every threat model is custom-built. The tools you build for yourself will always be better than the tools you buy, because they're built for *your* questions, not someone else's marketplace.

**Network awareness is security awareness.** If you can't see the traffic, you can't secure it. This is not a complex principle. It is, however, deeply counterintuitive in an age where most people interact with technology through interfaces that hide the underlying complexity. A web browser hides the HTTP requests. A mobile app hides the API calls. A cloud service hides the data flows. The net-pack-parser was an act of *un-hiding* — of pulling back the curtain to reveal what was actually happening beneath the interface.

This principle extends beyond security. Network awareness is *systems* awareness. When you can see the traffic between your services, you begin to understand their relationships — which services depend on which, which connections are critical, which are redundant, which are failures waiting to happen. The net-pack-parser didn't just show security-relevant traffic — it showed the *topology* of the PAI, the hidden map of connections that defined its architecture. This topology map became invaluable during troubleshooting, capacity planning, and incident response.

---

## The Code That Changed Everything

The git history of net-pack-parser begins with a commit that seems almost modest in retrospect:

```
feat: initial network capture and analysis framework
```

There was no grand announcement. No blog post. No social media update. Just a commit, a README, and a handful of TypeScript files in a new directory.

But that commit represented a shift in mindset. Before net-pack-parser, the author built tools and hoped they were secure. After net-pack-parser, the author built tools and *verified* that they were secure — by watching what they did, not just by examining what they contained.

The difference between hope and verification is the difference between a house with a lock and a house with a security system. Both protect. But only one *knows* when something goes wrong.

---

## What net-pack-parser Taught

The project was small by the standards of later PAI components. It had no launchd integration, no TLS termination, no identity framework. It was, in many ways, a prototype — a proof that network visibility was possible at the application layer without root privileges.

But it taught three lessons that would shape everything that followed:

**Start with the question, not the tool.** The net-pack-parser was not built because the author wanted to build a network monitor. It was built because the author had a specific question: *What is my machine actually doing on the network?* The tool was the answer, not the starting point. This pattern — question first, tool second — would be repeated across thirty-two projects.

**Visibility enables control.** You cannot control what you cannot see. The net-pack-parser didn't just show the author what was happening — it gave them the information needed to *change* what was happening. Once they saw that NotebookLM was phoning home, they could choose to process sensitive documents locally. Once they saw that Ollama was sending telemetry, they could configure it to stop. Visibility was not the end goal — it was the *enabler* of control.

**The observer changes the observed.** This is a principle from quantum physics that applies equally well to security. The act of monitoring network traffic changes the traffic — not because the monitor interferes with it, but because the *knowledge* of being monitored changes behaviour. After net-pack-parser was running, the author became more careful about which services they installed, which APIs they called, which data they uploaded. The monitor didn't just observe — it *influenced*. This is the desired effect of security monitoring: not just detection, but deterrence.

---

## The Thread That Connects

The net-pack-parser was born in May 2025. It was a tool for seeing the invisible — the network traffic that flows through every modern application, most of it unnoticed, most of it unexamined.

But visibility is only the first step. Once you can see what's happening, you need to understand *why* it's happening. And understanding why requires more than network logs — it requires context, intent, and meaning.

That need would lead directly to the next chapter: giving the machine a voice. If net-pack-parser was the eyes of the PAI — watching, observing, recording — then the next component would be the mouth — speaking, communicating, alerting. A system that can see but not speak is a surveillance tool. A system that can see *and* speak is an assistant.

The net-pack-parser gave the PAI its first sense of awareness. The voice system would give it a voice. And together, they would begin to form something that felt less like a collection of tools and more like a true infrastructure — one that could perceive its environment and respond to it.

But the journey from visibility to voice is longer than any single chapter can capture. It requires understanding not just *what* is happening on the network, but *what it means* — and that understanding would require a system that could remember, reason, and ultimately, *think*.

The net-pack-parser was not a philosophy. It was a tool — a modest collection of TypeScript classes, a PostgreSQL database, an Elasticsearch index. But it was the first tool that asked the right question: *What is actually happening?*

That question would echo through every project that followed. Not as a slogan, but as a habit. Before building a new capability, the author would ask: *How will I know if this is working? How will I know if it's broken? How will I know what it's doing?*

The answers to those questions would require more than a network parser. They would require identity systems, memory databases, unified dashboards. But the habit of asking — of refusing to accept opacity — began here.

And the next question was already forming: *Now that I can see what my machine is doing, how do I make it speak back?*

---

## Chapter Summary

Chapter 2 documents the construction of net-pack-parser, born from the unsettling realization that the author's "local-first" AI infrastructure was actually making hundreds of outbound connections to cloud services. The project's breakthrough was a proxy-based capture approach that avoided root privileges by operating at the application layer — using the Performance API to observe traffic rather than raw packet capture. The chapter establishes the observability-first principle that would guide every subsequent project: *if you can't see it, you can't secure it.* The deep packet inspection capabilities — TCP stream reassembly, YARA rule matching, MITRE ATT&CK mapping — transformed the parser from a passive monitor into an active threat detector. The forensic value was proven during a real security incident where the parser captured an entire attack chain from initial connection to reverse shell. Three enduring lessons emerged: start with the question not the tool, visibility enables control, and the observer changes the observed — monitoring doesn't just detect threats, it deters them by changing behavior.

---

## Key Technical Facts

| Detail | Value |
|--------|-------|
| **Project** | net-pack-parser |
| **First Commit** | May 2025 (concept), February 2026 (full implementation) |
| **Language** | TypeScript (Node.js server, React client) |
| **Key Innovation** | Application-layer proxy-based capture without root privileges |
| **Database** | PostgreSQL (metadata) + Elasticsearch (packet indexing) |
| **Threat Detection** | YARA engine + MITRE ATT&CK mapping |
| **Deployment** | Docker-compatible, launchd-integrated |
| **Runtime** | Node.js 18+ / Bun |

---

## Narrative Threads to Carry Forward

- The security architect's need for visibility
- The tension between local claims and cloud reality
- Observability as a foundation for security
- The pattern of building tools to answer personal questions
- The transition from passive monitoring to active threat detection
- The philosophy that visibility enables control
