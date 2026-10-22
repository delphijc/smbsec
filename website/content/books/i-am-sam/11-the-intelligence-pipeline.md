+++
title = "Chapter 11 — The Intelligence Pipeline"
date = "2026-10-21"
weight = 120
+++
# Chapter 11: The Intelligence Pipeline

Information overload is the modern security professional's curse.

Threat feeds pour in by the thousands daily. Vulnerability scanners generate reports by the megabyte. Security blogs publish analysis by the hour. The problem is no longer finding information — it is finding *signal* in the noise.

---

It was 7:42 AM on a Tuesday. The author had set up a morning routine: check threat feeds, review vulnerability scans, read security blogs, then triage alerts. It should have taken thirty minutes. It took three hours.

The threat feeds had generated 847 new indicators overnight. The vulnerability scanner had flagged 23 new findings. The security blogs had published 14 new advisories. The alert stream had fired 312 events. Most were noise. Some were important. Distinguishing between them required reading each one.

"I built these tools to protect myself," the author thought. "But I'm spending more time managing the tools than using them."

This was the information deluge in action. Every tool in the PAI was generating more data than a human could process. The volume was growing faster than the author's capacity. Without automation, the system would become a liability — generating noise that required manual filtering.

That was the moment the question formed: *What if the system could think for itself?*

---

## The Information Deluge

By early 2026, the author's monitoring ecosystem had grown to include multiple sources of security intelligence, each valuable but each noisy:

- **Threat intelligence feeds** — STIX bundles from multiple providers, generating thousands of indicators daily. Most were irrelevant to the author's specific environment, but filtering them required manual effort.
- **Vulnerability scans** — Regular scans from smbsec and security-training-sandbox, producing hundreds of findings per scan. Many were low-severity or not applicable, but distinguishing signal from noise required expertise and time.
- **Network monitoring** — Continuous packet capture from net-pack-parser, flagging anomalies. Most were false positives from legitimate software updates, but genuine anomalies could be missed in the noise.
- **Alert streams** — Real-time alerts from cyber-alert-mgr, correlated and scored. But correlation was limited to the tools that fed into it — there was no cross-source correlation.
- **Security blogs and advisories** — Manual monitoring of vendor advisories, research papers, and industry news. This was the most time-consuming but also the most valuable source of context.

Before awareness, the author's approach to intelligence was informal and inconsistent. Threat feeds were checked manually. Vulnerability scans were reviewed on demand. Security blogs were read when time permitted. There was no automated correlation — no system that could connect a threat actor's TTP to a vulnerability in the author's infrastructure.

This wasn't just inconvenient. It was dangerous. The 3 AM vulnerability incident from Chapter 10 was not an isolated case — it was a pattern. Intelligence arrived outside business hours. Critical findings were buried under noise. Relevant threat actor activity was missed because the author was focused on a different source.

The choice wasn't between automated and manual intelligence. It was between automated intelligence and acceptance of risk. And for someone who valued both security and peace of mind, that was an unsustainable tradeoff.

The problem was not technical — it was cognitive. No human could maintain situational awareness across so many sources. The system needed to *think* — to research, correlate, score, and deliver intelligence automatically. This was the insight that drove the awareness project.

---

## The First Attempt

The breakthrough came from recognizing that intelligence curation was a solvable problem — if you approached it systematically.

The author had been following the development of multi-agent research systems and realized that the same principles could be applied to security intelligence. But the first attempt was embarrassingly simple.

The author assigned fixed severity scores to each intelligence item: critical CVEs scored 10, high-severity advisories scored 8, medium scored 5, low scored 2. It was fast. It was easy. It was wrong.

A critical CVE affecting a library the author never used scored the same as a critical CVE affecting their primary web server. A high-severity advisory about a threat actor targeting their industry scored lower than a medium-severity advisory about an obscure dependency. The scores were objectively correct — but contextually meaningless.

"I built a scoring system that doesn't know me," the author noted. "That's not intelligence. That's a spreadsheet with opinions."

The problem wasn't the scoring — it was the assumption that severity scores were universal. A score that makes sense for a generic security professional doesn't make sense for a specific infrastructure. The author needed scoring that adapted to their environment, their history, and their priorities.

This was the first lesson in adaptive intelligence: fixed thresholds are rigid; adaptive scoring is contextual.

---

## What Emerged

The components that emerged from the awareness project didn't arrive all at once. They developed gradually, each one responding to a specific limitation that became visible only after the previous component was in place.

**Multi-agent research** came first — the basic ability to parallelize source processing. This was the threshold feature: without it, the pipeline was too slow to be useful. The first implementation was a single-agent pipeline that fetched threat feeds, vulnerability scans, and network data sequentially. It worked, but if the threat actor agent was slow, the vulnerability agent waited. The second attempt ran agents in parallel with `Promise.all`, but there was no conflict resolution and no scoring. Results conflicted, duplicates were missed, and the orchestrator had no way to reconcile disagreements.

The final approach came with the orchestrator — a coordinator that ran agents in parallel, resolved conflicts, and produced a unified result. But even this evolved. The original build (March 2026) had four LLM-based researchers: Perplexity for emerging stories, Gemini for gap analysis, Claude for deep analysis, and a generic fallback for horizon scanning. Later, direct-source agents were added — an RSS feed agent for continuous monitoring, a CISA KEV agent for known exploited vulnerabilities, and an OPML feed agent for curated source lists. The pipeline grew from four agents to nine, each with a specific role.

```typescript
// awareness — First research agent (March 2026)
// Note: This was NOT the final architecture.
// The multi-agent design came later.
// The adaptive scoring was added after the first digest complaint.

// First attempt: Single-agent pipeline
// Failed: One agent couldn't cover all source types
// If the threat actor agent was slow, the vulnerability agent waited
const pipeline = {
  async run() {
    const threatFeeds = await fetchThreatFeeds();
    const vulnScans = await fetchVulnScans();
    const networkData = await fetchNetworkData();
    
    // All processed sequentially
    const threatReport = await analyzeThreats(threatFeeds);
    const vulnReport = await analyzeVulns(vulnScans);
    const networkReport = await analyzeNetwork(networkData);
    
    return { threatReport, vulnReport, networkReport };
  }
};

// Second attempt: Parallel agents
// Worked: Faster
// Problem: No conflict resolution, no scoring
const parallelAgents = [
  new ThreatActorAgent(),
  new VulnerabilityAgent(),
  new NetworkAgent()
];
const results = await Promise.all(parallelAgents.map(a => a.research()));

// Final approach: Multi-agent with orchestrator
// Required: Conflict resolution, adaptive scoring, interest graph
// Result: Coordinated, contextual, personalized intelligence
class ResearchOrchestrator {
  async runCycle() {
    const results = await Promise.all([
      this.threatAgent.research(),
      this.vulnAgent.research(),
      this.networkAgent.research(),
      this.policyAgent.research()
    ]);
    
    // Resolve conflicts
    const resolved = this.resolveConflicts(results);
    
    // Score adaptively
    const scored = resolved.map(r => this.score(r));
    
    // Update interest graph
    this.updateInterestGraph(scored);
    
    return this.generateDigest(scored);
  }
}
```

**Adaptive scoring** emerged from the realization that fixed thresholds were insufficient. A CVE affecting the author's primary web server was more urgent than one affecting an obscure dependency. But the scoring engine needed to learn what was important — and that required interaction data.

The scoring engine evolved through five dimensions: recency (how recent is the intelligence?), relevance (how relevant is it to the author's environment?), corroboration (how many independent sources confirm it?), novelty (is this already known?), and actionability (what can be done about it?). Each dimension had configurable weights, and the weights themselves could be adjusted as the author's priorities changed.

```typescript
// awareness — Scoring engine evolution
// Phase 1: Fixed thresholds (March 18, 2026)
// Failed: Scores didn't adapt to environment
const fixedScore = (item: Intelligence) => {
  if (item.severity === 'critical') return 10;
  if (item.severity === 'high') return 8;
  if (item.severity === 'medium') return 5;
  return 2;
};
// Problem: A critical CVE for an unused library scored same as one for the web server

// Phase 2: Multi-dimensional scoring (March 18, 2026)
// Added: Recency, relevance, corroboration, novelty, actionability
// Required: Interest graph for relevance weighting
interface ScoringContext {
  weights: ScoringWeights;           // Configurable dimension weights
  focusAreas: FocusArea[];           // Author's active interests
  pool: ResearchResult[];            // Full result pool for corroboration
  db: Database;                      // Knowledge base for novelty checks
}

function scoreItem(item: ResearchResult, ctx: ScoringContext): ScoredItem {
  return {
    recency: scoreRecency(item, ctx.now),           // Weibull decay
    relevance: scoreRelevance(item, ctx.focusAreas), // Keyword + interest weights
    corroboration: scoreCorroboration(item, ctx.pool), // Independent source count
    novelty: scoreNovelty(item, ctx.db),             // FTS + Jaccard similarity
    actionability: scoreActionability(item),         // Signal keyword heuristics
    composite: weightedSum(item, ctx.weights)
  };
}
// Still rigid: weights were static, not learned

// Phase 3: Adaptive scoring with engagement feedback (March 2026+)
// Added: User interaction signals (ACT, IGNORE) reinforce/decay topic weights
// Required: Interest graph with ACT-R decay
// Result: Scores adapt as the author's behavior changes
```

**Interest graph learning** came next — the system's memory of what mattered to the author. It started with default interests seeded from focus area labels, each with weight 1.0. Over time, the graph evolved based on the author's interactions. If the author consistently acted on intelligence about a particular topic, the graph learned to prioritize that topic. If the author ignored certain sources, the graph learned to weight them less heavily.

The interest graph used ACT-R activation decay — a cognitive science model where memory activation decays exponentially over time unless reinforced. Topics with high activation were prioritized. Topics that went dormant (weight below 0.1) were deprioritized but not forgotten. This meant the system could adapt to changing priorities without losing historical context.

```typescript
// awareness — Interest graph evolution
// Phase 1: Simple topic weights (March 18, 2026)
// Added: Reinforce on action, decay on ignore
interface InterestGraph {
  topics: Map<string, number>;      // Topic → interest score
  sources: Map<string, number>;     // Source → trust score
  actors: Map<string, number>;      // Threat actor → attention score
  lastUpdated: Date;
}

function updateInterestGraph(graph: InterestGraph, interaction: UserInteraction): InterestGraph {
  const newGraph = { ...graph };
  
  if (interaction.action === 'acted') {
    newGraph.topics.set(interaction.topic, 
      (newGraph.topics.get(interaction.topic) || 0) + 1);
  }
  
  if (interaction.action === 'ignored') {
    newGraph.topics.set(interaction.topic, 
      Math.max(0, (newGraph.topics.get(interaction.topic) || 0) - 0.5));
  }
  
  newGraph.lastUpdated = new Date();
  return newGraph;
}
// Problem: No time-based decay. Topics never faded.

// Phase 2: ACT-R activation decay (March 18, 2026)
// Added: Exponential half-life decay (14-day default)
// Required: SQLite persistence, last_activated timestamps
// Result: Topics naturally fade unless reinforced
```

**Daily digests** rounded out the system — the output mechanism that made intelligence consumable. Instead of real-time overload, intelligence was aggregated into daily briefings. The digest formatter grouped findings by focus area, included a serendipity section for unexpected but potentially valuable items, and persisted each digest to `data/digests/YYYY-MM-DD.md`.

But the digest was also the first place where the author noticed problems. The first digest was too long — 47 items, most of them low-relevance. The second digest was better but still included duplicates from different agents. The third digest introduced conflict resolution and deduplication. Each iteration responded to a specific complaint.

```typescript
// awareness — Digest evolution
// Phase 1: Simple aggregation (March 18, 2026)
// Problem: 47 items, no grouping, lots of duplicates
interface DailyDigest {
  date: Date;
  summary: string;
  topFindings: Intelligence[];
  newThreats: ThreatActor[];
  vulnerabilityAlerts: Vulnerability[];
  actionItems: ActionItem[];
  sources: SourceSummary[];
}

// Phase 2: Focus area grouping + serendipity (March 18, 2026)
// Added: Grouping by focus area, 25% serendipity quota
// Required: Serendipity agent for horizon scanning
// Result: More manageable, some unexpected value

// Phase 3: Conflict resolution + deduplication (March 2026+)
// Added: Cross-agent conflict resolution, URL-based dedup
// Required: Enhanced orchestrator, feed mutex for SQLite WAL
// Result: Clean, non-redundant briefings
```

---

## The Build

awareness was built in TypeScript, leveraging the same Bun runtime. It ran as a launchd-managed service, executing research cycles on a schedule and delivering digests daily.

### The Research Cycle Architecture

The awareness pipeline operated on a continuous research cycle:

```
┌─────────────────────────────────────────────────────┐
│                 Research Cycle                      │
├─────────────────────────────────────────────────────┤
│  1. Fetch all sources (feeds, scans, captures)      │
│       ↓                                             │
│  2. Parallel agent research (9 agents)              │
│       ├─ Perplexity → Emerging Stories              │
│       ├─ Gemini → Gap Analysis                      │
│       ├─ Claude → Deep Analysis                     │
│       ├─ Generic → Horizon Scanning                 │
│       ├─ RSS Feed → Continuous Monitoring           │
│       ├─ CISA KEV → Known Exploited Vulns           │
│       ├─ OPML Feed → Curated Sources                │
│       ├─ DuckDuckGo → Fallback Search               │
│       └─ Situation Brief → Context Builder          │
│       ↓                                             │
│  3. Aggregate & resolve conflicts                   │
│       ↓                                             │
│  4. Score with adaptive weights                     │
│       ↓                                             │
│  5. Update interest graph                           │
│       ↓                                             │
│  6. Generate daily digest                           │
│       ↓                                             │
│  7. Deliver via channels (terminal, file, voice)    │
└─────────────────────────────────────────────────────┘
```

Each cycle ran on a schedule (typically every 6 hours), with the daily digest delivered once per day. The parallel agent design meant that the pipeline could cover more ground than a single researcher — each agent specialized in a different domain and could process its sources independently.

The multi-agent research engine was the most complex part. Each agent was a specialized researcher with its own sources, methods, and output format. The orchestrator coordinated research cycles, aggregated results, and resolved conflicts. The agents communicated through a shared context layer, allowing them to build on each other's findings.

But building it wasn't clean. The original build (commit `4529392`, March 20, 2026) had 93 files and 8,037 lines added. It worked, but it was narrow — four LLM agents with no direct sources, no enrichment, no correlation. Two weeks later, a major rewrite (commit `22dee94`, March 23, 2026) added RSS feed agents, CISA KEV monitoring, entity extraction, IOC enrichment, CVE lookup, asset matching, campaign detection, and Discord alerting. That commit added 26,457 lines across 150 files.

Then came the refactor (commit `ea8f81c`, March 26, 2026) with the message: "trying to make it all work." That commit added feed polling, OPML ingestion, rating graphs, situation briefs, and a proper pipeline runner. It was the phase where the system went from "works in isolation" to "works as a system."

The most painful fixes came later. SQLite WAL lock contention required running `pollFeeds` inline instead of in parallel (commit `97b54d9`). Feed mutex locks prevented concurrent writes (commit `10889a0`). Digest backfill CLI allowed replaying historical data (commit `10889a0`). Each fix addressed a specific failure mode that only became visible under real-world load.

The adaptive scoring engine was the most innovative part. Instead of fixed thresholds, scores were calculated dynamically based on the author's environment and history. A CVE affecting a commonly used library scored higher than one affecting an obscure dependency. A threat actor targeting the author's industry scored higher than one targeting a different sector. The scoring engine learned from the author's interactions, becoming more accurate over time.

The interest graph was the most learning-oriented part. It started with default interests and evolved over time based on the author's interactions. If the author consistently acted on intelligence about a particular topic, the graph learned to prioritize that topic. If the author ignored certain sources, the graph learned to weight them less heavily. The graph was the system's memory of preferences.

---

## The Integration

awareness integrated with multiple PAI services:

**cyber-alert-mgr** → Intelligence context for alerts. When an alert was generated, awareness could provide relevant threat intelligence context — is this related to an active campaign? Is this a known TTP?

**jay-auth** → Secure access to intelligence sources. Sensitive threat intelligence required authentication; jay-auth provided secure credential management.

**voice-server** → Digest narration. Daily digests could be read aloud, turning text reports into audio briefings.

**jay-gentic** → Intelligent triage. The agentic CLI could query awareness for current intelligence context when processing security requests.

This integration was the first expression of what would become the PAI intelligence architecture: **multiple research agents, one scoring engine, one interest graph, one digest.**

---

## The Impact

awareness was a milestone in the PAI's evolution from monitoring to *understanding*. It demonstrated that:

**Automation scales cognition.** No human can review thousands of threat indicators daily. An automated pipeline can — and can do it consistently, without fatigue.

**Scoring enables prioritization.** Not all intelligence is equal. Dynamic scoring transforms raw data into actionable priorities.

**Learning enables relevance.** An interest graph that evolves over time ensures that the system prioritizes what matters to *you*, not what matters to everyone.

**Digests enable manageability.** Daily aggregation transforms information overload into manageable briefings.

More importantly, awareness was the first PAI component that truly *thought*. It didn't just collect data — it researched, correlated, scored, and delivered intelligence. It was the first step toward true AI-driven security operations.

---

The first time someone else experienced awareness's daily digest, it was accidental. A colleague was visiting, and the author had left a terminal open with the morning briefing running. The digest was reading aloud — a calm, authoritative voice summarizing overnight threat activity, new vulnerabilities, and recommended actions.

"Is that... your computer talking?" the colleague asked.

"Yes."

"About what?"

"About the security landscape. Overnight, three new threat actors were active in our sector. Two critical CVEs were published. One affects our web server."

The colleague listened as the voice continued — a summary of the top findings, the recommended actions, the confidence scores. The entire digest took about two minutes.

"Does it... know what's important?"

"It learns. If I act on intelligence about a topic, it prioritizes that topic. If I ignore certain sources, it weights them less."

The colleague didn't ask to use it. They didn't need to. The demonstration was sufficient. But the look on their face — the realization that a system could curate intelligence automatically, that the noise had been filtered into signal — was the validation the author hadn't known they needed.

---

## The Lesson

Information overload is not a data problem — it is a cognition problem. The solution is not more data; it is better processing.

The awareness project teaches us that:

1. **Multi-agent research scales.** Parallel research agents can cover more ground than a single researcher, with specialized expertise in different domains. But the architecture evolves — the first design is rarely the final one.

2. **Adaptive scoring enables prioritization.** Fixed thresholds are rigid; adaptive scoring is contextual and relevant. The first scoring system will be wrong — that's okay, as long as it learns.

3. **Learning enables personalization.** An interest graph that evolves over time ensures the system prioritizes what matters to you. But learning requires interaction — the system needs feedback to improve.

4. **Digests enable manageability.** Aggregation transforms overload into overview. But the first digest will be too long, the second too narrow, the third just right. Iteration is the path to quality.

---

## Code Snippets

### Research Agent (Evolution)

```typescript
// awareness — Research agent evolution
// Three iterations, each addressing a specific limitation

// Iteration 1: Single-agent sequential pipeline (March 2026)
// Problem: Slow. If one agent blocked, all waited.
class SequentialPipeline {
  async run() {
    const threatFeeds = await fetchThreatFeeds();
    const vulnScans = await fetchVulnScans();
    const networkData = await fetchNetworkData();
    
    const threatReport = await analyzeThreats(threatFeeds);
    const vulnReport = await analyzeVulns(vulnScans);
    const networkReport = await analyzeNetwork(networkData);
    
    return { threatReport, vulnReport, networkReport };
  }
}
// Fix: Parallelize with Promise.all

// Iteration 2: Parallel agents, no coordination (March 2026)
// Problem: No conflict resolution. Duplicates. No scoring.
const parallelAgents = [
  new PerplexityResearcher(),   // Emerging stories
  new GeminiResearcher(),       // Gap analysis
  new ClaudeResearcher(),       // Deep analysis
  new GenericResearcher()       // Horizon scanning
];
const results = await Promise.all(parallelAgents.map(a => a.research()));
// Fix: Add orchestrator with conflict resolution

// Iteration 3: Multi-agent with orchestrator (March 2026+)
// Required: Conflict resolution, adaptive scoring, interest graph
// Result: Coordinated, contextual, personalized intelligence
class ResearchOrchestrator {
  async runCycle() {
    const results = await Promise.all([
      this.perplexityAgent.research(),
      this.geminiAgent.research(),
      this.claudeAgent.research(),
      this.genericAgent.research(),
      this.rssAgent.research(),
      this.cisaKevAgent.research(),
      this.opmlAgent.research(),
      this.ddgAgent.research()
    ]);
    
    // Resolve conflicts (same CVE from multiple agents)
    const resolved = this.resolveConflicts(results);
    
    // Score adaptively (environment-aware)
    const scored = resolved.map(r => this.score(r));
    
    // Update interest graph (learn from interactions)
    this.updateInterestGraph(scored);
    
    return this.generateDigest(scored);
  }
}
```

### Scoring Engine (Evolution)

```typescript
// awareness — Scoring engine evolution
// Three iterations, each addressing a specific limitation

// Iteration 1: Fixed thresholds (March 18, 2026)
// Problem: A critical CVE for an unused library scored same as one for the web server
function fixedScore(item: Intelligence): number {
  const severityMap = { critical: 10, high: 8, medium: 5, low: 2 };
  return severityMap[item.severity] ?? 0;
}
// Failure: Scores were objectively correct but contextually meaningless

// Iteration 2: Multi-dimensional scoring (March 18, 2026)
// Added: 5 dimensions (recency, relevance, corroboration, novelty, actionability)
// Problem: Weights were static. System couldn't adapt to changing priorities.
function multiDimensionalScore(
  item: ResearchResult,
  context: ScoringContext
): ScoredItem {
  return {
    recency: scoreRecency(item, context.now),           // Weibull decay
    relevance: scoreRelevance(item, context.focusAreas), // Keyword + interest weights
    corroboration: scoreCorroboration(item, context.pool), // Independent source count
    novelty: scoreNovelty(item, context.db),             // FTS + Jaccard similarity
    actionability: scoreActionability(item),             // Signal keyword heuristics
    composite: weightedSum(item, context.weights)
  };
}
// Still rigid: weights didn't change based on user behavior

// Iteration 3: Adaptive scoring with engagement feedback (March 2026+)
// Added: ACT-R decay, user interaction signals, dynamic weight adjustment
// Result: Scores adapt as the author's behavior changes
function adaptiveScore(
  item: ResearchResult,
  context: ScoringContext,
  userHistory: UserInteraction[]
): ScoredItem {
  // Adjust weights based on historical engagement patterns
  const dynamicWeights = adjustWeights(context.weights, userHistory);
  
  const baseScore = multiDimensionalScore(item, { ...context, weights: dynamicWeights });
  
  // Apply engagement-based boost/decay
  const engagementBoost = calculateEngagementBoost(item, userHistory);
  
  return {
    ...baseScore,
    composite: baseScore.composite * engagementBoost
  };
}
// Now the system learns: what the author acts on gets prioritized
```

---

## Chapter Closing

The awareness project was born in March 2026. It gave the PAI infrastructure the ability to *think* — researching sources, correlating findings, scoring relevance, and delivering curated intelligence. But the path to that capability was paved with failures: fixed scores that didn't adapt, sequential pipelines that were too slow, digests that were too long, SQLite locks that crashed the pipeline.

The chapters ahead will show you how awareness became the foundation for the PAI's intelligence architecture — how multi-agent research, adaptive scoring, and interest graph learning transformed the infrastructure from passive monitoring to active understanding. But before that evolution, there was another question forming.

If the system can think, how should it think?

The answer would require more than a research pipeline. It would require an interface — a terminal that could think, remember, and collaborate. It would require an agent.

The next chapter will explore that question — and the breakthrough that gave the PAI not just a mind, but a voice.

---

## Chapter Summary

Chapter 11 documents the construction of awareness, the automated intelligence pipeline that gave the PAI the ability to think — researching sources, correlating findings, scoring relevance, and delivering curated digests. The chapter traces the iterative evolution from a naive single-agent pipeline to a nine-agent orchestration system, from fixed severity scores to adaptive multi-dimensional scoring, from simple topic weights to ACT-R decay-based interest graphs. The four key components emerged gradually, each one responding to a specific limitation: multi-agent research scaled with parallelism, adaptive scoring enabled contextual prioritization, interest graphs enabled personalization through learning, and daily digests enabled manageability through aggregation. The chapter establishes that information overload is a cognition problem, not a data problem — the solution is better processing, not more data. Awareness was the first PAI component that truly thought, transforming the infrastructure from monitoring to understanding.

---

## Key Technical Facts

| Detail | Value |
|--------|-------|
| **Project** | awareness |
| **First Commit** | March 18, 2026 (scaffolding), March 20, 2026 (original build) |
| **Language** | TypeScript |
| **Runtime** | Bun |
| **Key Innovation** | Multi-agent research with adaptive scoring and interest graph learning |
| **Agents (final)** | Perplexity, Gemini, Claude, Generic, RSS Feed, CISA KEV, OPML Feed, DuckDuckGo |
| **Scoring Dimensions** | Recency (Weibull decay), Relevance (keyword + interest), Corroboration (source count), Novelty (FTS + Jaccard), Actionability (signal keywords) |
| **Interest Graph** | ACT-R activation decay, 14-day half-life, SQLite persistence |
| **Output** | Daily digests via terminal, file (`data/digests/YYYY-MM-DD.md`), and voice |
| **Major Refactors** | March 23 (OSINT enhancement), March 26 ("trying to make it all work"), May-June (feed mutex, WAL fixes, digest backfill) |
| **Test Coverage** | 1,037 tests, 96.5%+ function coverage |

---

## Narrative Threads to Carry Forward

- From monitoring to understanding
- Multi-agent research as scale (and its iterative evolution)
- Adaptive scoring as relevance (and the failure of fixed thresholds)
- Interest graphs as personalization (and the role of engagement feedback)
- Digests as manageability (and the pain of first drafts)
- The colleague's reaction as validation
- If the system can think, how should it think? → leads to Chapter 12 (the terminal becomes an agent)
