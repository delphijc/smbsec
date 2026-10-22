+++
title = "Chapter 21 — Tracking the Adversary"
date = "2026-10-21"
weight = 220
+++
# Chapter 21: Tracking the Adversary

Security is not just about defending against known threats. It is about understanding the adversaries who pose those threats.

By mid-2026, the PAI had robust security capabilities — threat detection via net-pack-parser, alert management via cyber-alert-mgr, vulnerability scanning via smbsec. But threat intelligence was scattered. STIX bundles arrived from multiple feeds. MITRE ATT&CK mappings existed in isolation. Adversary profiles were stored in different formats across different projects.

There was no unified view of the threat landscape. The author was consuming threat intelligence but not *understanding* it — not seeing the connections between actors, tactics, and targets.

---

## The Intelligence Fragmentation

It was 2:33 AM on a Tuesday. The author was reviewing overnight alerts when they noticed something unsettling. Three separate alerts had fired — one from cyber-alert-mgr about a suspicious IP, one from threat-actor-tracker about a known threat actor, one from awareness about a new CVE. Each alert was valuable individually. But none of them were connected.

The suspicious IP matched a known threat actor's infrastructure. The threat actor was known to exploit the CVE. The CVE affected the author's infrastructure. But there was no way to see this connection automatically — the author had to manually cross-reference three different systems.

"I built these tools to protect myself," the author thought. "But I can't see the connections between threats."

This was the intelligence fragmentation in action. Threat intelligence was scattered across multiple sources — STIX feeds, MITRE mappings, adversary profiles, vulnerability data. Each was valuable individually, but lacking the contextual understanding that comes from correlation.

Before threat-actor-tracker, the author's approach to threat intelligence was informal and inconsistent. STIX feeds were ingested but not correlated. MITRE mappings were manual and incomplete. Adversary profiles were scattered across blog posts and internal notes. There was no unified view of the threat landscape.

This wasn't just inconvenient. It was dangerous. The 2:33 AM alert incident was not an isolated case — it was a pattern. Threats were missed because their connections were invisible. Attack paths were unknown because the data was fragmented. The author had data but not understanding.

The choice wasn't between correlated and fragmented intelligence. It was between correlated intelligence and acceptance of risk. And for someone who valued both security and understanding, that was an unsustainable tradeoff.

Threat intelligence in the PAI ecosystem was fragmented across multiple sources, each valuable but each isolated:

**STIX feeds.** Multiple threat intelligence providers published STIX 2.1 bundles containing indicators of compromise, attack patterns, and adversary information. These were ingested by cyber-alert-mgr but not correlated. The same threat actor might appear in multiple feeds, but there was no way to link them.

**MITRE ATT&CK.** The author had mapped some detection rules to MITRE ATT&CK techniques, but the mappings were manual and incomplete. There was no automated mapping or visualization. Understanding which tactics an adversary used required manual cross-referencing.

**Adversary profiles.** Information about specific threat actors was scattered across blog posts, research papers, and internal notes. There was no centralized adversary database. Finding information about a specific actor required searching multiple sources.

**Vulnerability data.** CVEs and vulnerability findings were tracked by smbsec but not linked to adversary exploitation patterns. Knowing that a vulnerability existed was different from knowing that an adversary was actively exploiting it.

The result was a collection of isolated intelligence artifacts — useful individually, but lacking the contextual understanding that comes from correlation. The author had data but not understanding.

---

## The Graph Insight

The breakthrough came from recognizing that threats are relational. Adversaries use tactics. Tactics target vulnerabilities. Vulnerabilities affect assets. Assets belong to organizations. These are not isolated facts — they are a *graph*.

A graph model is the natural representation for threat intelligence:
- **Nodes** represent entities: adversaries, tactics, techniques, tools, vulnerabilities, assets
- **Edges** represent relationships: uses, targets, exploits, affects, belongs

With a graph model, the author could ask complex questions:
- Which adversaries use this tactic?
- Which vulnerabilities does this adversary exploit?
- Which assets are at risk from this threat actor?
- What is the attack path from initial access to impact?

The first attempt at graph-based threat intelligence was embarrassingly simple. The author stored threat data in a SQLite database — tables for actors, tactics, techniques, with foreign keys for relationships. It worked. For a moment.

Then it broke. Not dramatically — there was no crash, no error message. It just became slow. Queries that should have been simple — "which actors use this tactic?" — required complex joins across multiple tables. As the data grew, query performance degraded.

"I built a graph database that queries like a relational database," the author noted. "That's not a graph. That's a table with relationships."

The problem wasn't the database — it was the assumption that relational storage was sufficient for graph data. A graph database is optimized for traversal; a relational database is optimized for joins. For threat intelligence, traversal is more important than joins.

This was the first lesson in graph architecture: use the right data model for the right queries.

---

## What Was Built

threat-actor-tracker became a graph-based threat intelligence platform. The capabilities that emerged didn't arrive all at once. They developed gradually, each one responding to a specific limitation that became visible only after the previous capability was in place.

**STIX Ingestion** came first — the basic ability to parse and normalize threat intelligence. This was the threshold capability: without it, there was no automated threat data. But early ingestion was naive — it handled some STIX object types but not others. Missing objects were lost, and the graph was incomplete.

```typescript
// threat-actor-tracker — First STIX ingestion (June 2026)
// Note: This was NOT the final architecture.
// The graph storage was added later.
// The MITRE mapping was added after the first correlation complaint.

// First attempt: Simple database storage
// Failed: No graph traversal, no relationship modeling
// Just tables with foreign keys
async function ingestSTIX(bundle: STIXBundle): Promise<void> {
  for (const obj of bundle.objects) {
    await db.insert(obj.type, obj);
  }
}

// Second attempt: Relational graph
// Worked: Better queries
// Problem: Slow joins, no native traversal
interface GraphQuery {
  startNode: string;
  relationshipType?: string;
  depth?: number;
}

async function queryGraph(query: GraphQuery): Promise<GraphNode[]> {
  // Complex joins across multiple tables
  // Slow as data grows
}

// Final approach: Graph storage with JSONB
// Required: Native traversal, flexible schema, fast queries
// Result: Threat intelligence that connects
class ThreatActorTracker {
  async queryGraph(query: GraphQuery): Promise<GraphNode[]> {
    // SQLite with JSONB for graph edges
    // Native traversal via recursive CTE
    // Fast even with large datasets
  }
}
```

**MITRE Mapping** emerged from the realization that raw indicators were insufficient. Knowing that an IP address was malicious was different from knowing that a specific threat actor used that IP with a specific tactic. But early mapping was manual — the author had to cross-reference indicators to MITRE techniques by hand.

**Graph Storage** emerged from the realization that the relational approach was too slow. The switch to SQLite with JSONB columns enabled recursive CTE queries for graph traversal — the kind of queries that matter for threat intelligence.

**Visualization** emerged from the realization that the author couldn't spot patterns in query results. A visual graph made connections visible at a glance.

---

## The Build

threat-actor-tracker was built in TypeScript, leveraging the same Bun runtime. It ran as a launchd-managed service, ingesting STIX bundles on a schedule and updating the graph.

The first commit was `9e34ea4` on June 16, 2026 — an initial scaffold. The second commit, `f24e2f2`, came just hours later on the same day. The project grew rapidly:

- `709031a` (June 15) — switch to port 5900, handle NULL SQL strings, stack visualizations with 1000px linkage graph
- `9cbd074` (June 15) — aggregate TTPs in linkage graph, clickable threat actor modal reports with co-association lookup
- `c37e5b3` (June 15) — incremental `--from-date` ingestion, daemon start script
- `424d5af` (September 4) — TLS proxy support, improved entity extraction, TTP-IoC repopulation
- `e1c7613` (September 16) — BigBear actor ID update, Awareness feed ingestion & initial backfill
- `f6bc38f` (October 17) — PDF dossier export, awareness intelligence integration, UI enhancements

The STIX ingester was the most complex part. STIX 2.1 is a rich but complex format with many object types and relationship patterns. The ingester needed to:
- Parse all STIX object types
- Normalize them to a common schema
- Extract relationships between objects
- Handle duplicate detection
- Track source provenance

The MITRE mapper was the most innovative part. It maintained a local copy of the MITRE ATT&CK matrix and matched ingested indicators to known techniques using:
- Pattern matching on indicator patterns
- Semantic similarity on descriptions
- Manual override for ambiguous cases

The graph storage was the most powerful part. SQLite with JSONB columns provided a flexible graph store that supported:
- Node storage with properties
- Edge storage with relationship types
- Graph traversal queries
- Full-text search on node properties

---

## The Integration

threat-actor-tracker integrated with multiple PAI services:

**cyber-alert-mgr** → Alert correlation with threat intelligence. When an alert was generated, the tracker could provide adversary context — is this related to a known threat actor? What tactics are they using?

**jay-gentic** → Intelligent threat analysis. The agentic CLI could query the tracker for adversary information when processing security requests.

**awareness** → Threat intelligence enrichment. Awareness research could be linked to tracked adversaries and tactics.

**service-monitor-dashboard** → Threat visualization. The dashboard could display the threat graph alongside service health.

This integration meant that threat intelligence was not isolated — it was connected to detection, analysis, and response.

---

## The Impact

The first time someone else experienced threat-actor-tracker's graph visualization, it was accidental. A colleague was visiting, and the author had left a browser open with the threat graph running. The graph showed adversaries as nodes, tactics as edges, and vulnerabilities as connected nodes — a web of threat intelligence that was immediately understandable.

"Is that... all the threat actors?" the colleague asked.

"Yes. And their tactics, techniques, and targets."

"Can you... click on them?"

"Yes. Try clicking on this node."

The colleague clicked on a threat actor node. The graph expanded, showing the actor's tactics, the techniques they used, the vulnerabilities they exploited, and the assets they targeted. The entire visualization took about two seconds to render.

"Does it... update automatically?"

"Yes. New STIX feeds are ingested every six hours. New MITRE mappings are generated automatically. The graph grows as the threat landscape grows."

The colleague didn't ask to use it. They didn't need to. The demonstration was sufficient. But the look on their face — the realization that threat intelligence could be visualized as a graph, that connections could be seen at a glance — was the validation the author hadn't known they needed.

threat-actor-tracker was a milestone in the PAI's evolution toward *proactive security*. It demonstrated that:

**Graphs model threats naturally.** Adversaries, tactics, techniques, and vulnerabilities are inherently relational. Graphs capture these relationships.

**Correlation enables understanding.** Individual indicators are data. Correlated indicators are intelligence.

**Visualization enables insight.** A graph visualization reveals patterns that tables and lists conceal.

**Automation enables scale.** Manual threat intelligence analysis does not scale. Automated ingestion and mapping enable continuous updating.

---

## The Lesson

Threats are relational. Understanding them requires more than data — it requires context.

The threat-actor-tracker project teaches us that:

1. **Graphs model threats naturally.** Adversaries, tactics, and vulnerabilities are inherently relational.

2. **Correlation enables understanding.** Individual data points become intelligence when connected.

3. **Visualization enables insight.** Patterns emerge in graphs that are invisible in tables.

4. **Automation enables scale.** Manual analysis does not scale. Automated ingestion is essential.

5. **Iteration beats design.** The graph architecture emerged from failure — the first relational approach was too slow, and the final design was shaped by that limitation.

---

## Code Snippets

### Graph Query

```typescript
// threat-actor-tracker — Adversary graph query
// Note: This evolved from the first simple database approach.
// The recursive CTE traversal was added after the join-based approach proved too slow.
async function getAdversaryPath(
  actorId: string,
  targetId: string
): Promise<GraphPath> {
  const path = await traverseGraph({
    start: actorId,
    end: targetId,
    maxDepth: 5
  });
  
  return {
    actor: path.start,
    tactics: path.edges.filter(e => e.type === 'uses_tactic'),
    techniques: path.edges.filter(e => e.type === 'uses_technique'),
    target: path.end
  };
}
```

### MITRE Mapping

```typescript
// threat-actor-tracker — MITRE ATT&CK mapping
// Note: Early mapping was manual.
// Automatic mapping was added after the first correlation complaint.
async function mapToMITRE(
  indicator: Indicator
): Promise<MITREMapping> {
  const technique = await findMatchingTechnique(
    indicator.pattern,
    indicator.description
  );
  
  return {
    indicator: indicator.id,
    technique: technique?.id,
    tactic: technique?.tactic,
    confidence: calculateConfidence(indicator, technique)
  };
}
```

---

## Chapter Closing

The threat-actor-tracker project was born in June 2026. It gave the PAI the ability to track adversaries — understanding their tactics, mapping their techniques, and visualizing their paths.

But tracking adversaries is only half the picture. The next chapter would explore how all security capabilities were unified into a single command center.

If the PAI can track adversaries, how should it coordinate its response?

The answer would require more than threat intelligence. It would require orchestration — a system that could coordinate detection, analysis, response, and reporting across all security capabilities. It would require a manager.

The next chapter will explore that question — and the breakthrough that gave the PAI a unified command center.

---

## Chapter Summary

Chapter 21 documents the construction of threat-actor-tracker, the graph-based threat intelligence platform that gave the PAI the ability to track adversaries, map their tactics, and visualize their paths. The chapter opens with the 2:33 AM Tuesday incident where three unconnected alerts — a suspicious IP, a known threat actor, and a CVE — turned out to be the same threat. The intelligence fragmentation was real: STIX feeds ingested but not correlated, MITRE mappings manual and incomplete, adversary profiles scattered, vulnerability data unlinked. The first graph attempt was embarrassingly simple — a SQLite database with foreign keys that became slow as data grew. "I built a graph database that queries like a relational database. That's not a graph. That's a table with relationships." The final architecture used SQLite with JSONB for graph edges and recursive CTE traversal. The capabilities emerged gradually: STIX ingestion first (naive, incomplete), then MITRE mapping (initially manual), then graph storage (after the relational approach proved too slow), then visualization (after patterns were invisible in query results). Git history anchors the evolution: June 16 initial commits, June 15 linkage graph, September 4 TLS proxy support, September 16 Awareness feed integration, October 17 PDF dossier export. A colleague accidentally discovered the threat graph running in a browser — clicking a threat actor node expanded the graph in two seconds, revealing tactics, techniques, vulnerabilities, and targets. The chapter establishes that threats are relational, correlation enables understanding, visualization reveals patterns, automation enables scale, and iteration beats design.

---

## Key Technical Facts

| Detail | Value |
|--------|-------|
| **Project** | threat-actor-tracker |
| **First Commit** | June 16, 2026 (`9e34ea4`, `f24e2f2`) |
| **Language** | TypeScript (Bun) |
| **Key Innovation** | Graph-based threat intelligence platform |
| **Data Model** | SQLite with JSONB for graph edges, recursive CTE traversal |
| **Integration** | cyber-alert-mgr, jay-gentic, awareness, dashboard |
| **Key Commits** | `709031a` (linkage graph), `424d5af` (TLS proxy), `e1c7613` (Awareness feed), `f6bc38f` (PDF export) |

---

## Narrative Threads to Carry Forward

- Graphs model threats naturally
- Correlation enables understanding
- Visualization enables insight
- Automation enables scale
- Iteration beats design
- The PAI gaining threat awareness
