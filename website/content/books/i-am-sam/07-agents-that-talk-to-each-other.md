+++
title = "Chapter 7 — Agents That Talk to Each Other"
date = "2026-10-21"
weight = 80
+++
# Chapter 7: Agents That Talk to Each Other

There is a fundamental limit to what any single AI tool can do. No matter how capable, it operates in isolation — processing requests, generating responses, completing tasks. But it cannot *collaborate*. It cannot delegate. It cannot ask a colleague for help.

Until it can.

In February 2026, the author built the first system that allowed AI agents to communicate, delegate, and collaborate: jagents. It started as a simple MCP server and evolved — through failure, iteration, and changing requirements — into a modular architecture that would influence every agent system to follow.

---

## The Moment of Overwhelm

It happened during a security incident response. A critical vulnerability had been detected by smbsec, and the author needed to: query nlm for remediation guidance, generate a report via chatterbox, notify the team via agent-remote-control, and log the incident for later analysis. Each step was straightforward in isolation. But chaining them together required manual coordination — copying output from one tool to the input of another, running commands in sequence, hoping the data survived the translation.

The workflow took twenty minutes. Twenty minutes that should have taken twenty seconds if the tools could talk to each other.

"I'm the integration layer," the author thought. "But I'm also the bottleneck."

That was the moment the question formed: *What if the tools could coordinate themselves?*

---

## The Isolation Problem

By late 2025, the author's toolkit had grown to ten+ projects. Each was capable in its own right:

- **nlm** could query documents and extract insights
- **chatterbox** could speak results aloud
- **net-pack-parser** could monitor network traffic and detect anomalies
- **cyber-alert-mgr** could analyze threats and generate YARA rules
- **smbsec** could provide security documentation and guidance
- **awareness** (in development) could research and synthesize intelligence
- **jay-gentic** (in development) could reason and plan

But they couldn't *work together*. If cyber-alert-mgr detected a critical vulnerability, there was no way to automatically:
1. Query nlm for related documentation and remediation guidance
2. Generate a report via chatterbox
3. Notify the author via voice
4. Log the incident for later analysis

Each tool was an island. The author was the bridge — manually copying information between systems, running commands in sequence, acting as the integration layer. This worked for simple workflows but broke down as complexity grew. The author was spending more time coordinating tools than using them.

Before jagents, the author's workflow was a series of isolated activities connected by manual coordination. A security incident required: running smbsec to scan, copying results to nlm for documentation, copying nlm output to chatterbox for reporting, copying chatterbox output to agent-remote-control for notification. Each handoff was a chance for information to be lost, distorted, or misunderstood.

This wasn't just inefficient. It was error-prone. The author would spend hours moving information between tools, only to realize that the context had been stripped away in translation. A vulnerability score from smbsec might be interpreted differently by nlm. A remediation recommendation from nlm might be paraphrased by chatterbox, losing nuance. A notification from agent-remote-control might be truncated, omitting critical details.

The choice wasn't between integration and isolation. It was between integration and friction. And for someone who valued both time and accuracy, that was an unsustainable tradeoff.

The problem was not technical — it was architectural. There was no protocol for agent-to-agent communication, no mechanism for service discovery, no pattern for task delegation. Each tool exposed a CLI or an API, but there was no system for orchestrating them.

---

## The MCP Promise — and Its Limits

The Model Context Protocol (MCP) had been announced by Anthropic in late 2024. It promised a standard way for AI agents to expose capabilities as tools that other agents could call. An agent could theoretically say: "I need to check network traffic" and call another agent's `network_scan` tool.

The promise was elegant. The reality was nascent.

The first attempt to use MCP directly was disappointing. The author had followed the specification, registered tools, and tried to call them across agents. It should have worked. The protocol defined the syntax. The tools were exposed correctly. But something was missing.

When Agent A called Agent B's `network_scan` tool, the call succeeded — but there was no way to know if Agent B was available, what its current load was, or whether it had failed silently. The protocol defined *how* to call a tool, but not *whether* the tool was ready, *how* to handle failures, or *what* to do if multiple agents exposed the same capability.

"MCP is a communication protocol," the author noted. "But it's not an architecture. It tells agents how to talk, but not how to work together."

In practice, MCP implementations were fragmented. There was no clear pattern for how agents should discover each other, negotiate capabilities, or handle failures. The protocol defined *how* agents could communicate — but not *how they should organize*. It was a communication protocol without an architecture.

The author recognized that MCP was necessary but not sufficient. What was needed wasn't just a communication protocol, but a **modular architecture** that defined:
1. How agents discover each other — Service discovery
2. How capabilities are advertised — Capability negotiation
3. How failures are handled — Resilience patterns
4. How context is shared — State management

This realization led to jagents 2.0 — a modular MCP architecture that went beyond simple tool calling to create a true agent ecosystem.

---

## What Emerged

The components that emerged from the jagents project didn't arrive all at once. They developed gradually, each one responding to a specific limitation that became visible only after the previous component was in place.

**Agent Registry** came first — the basic ability for agents to discover each other. This was the threshold feature: without it, there was no collaboration. But early registration was static — agents had to be manually configured, and there was no way to detect when an agent became unavailable. The first registry was a simple Map, and finding an agent by capability required scanning every entry. It worked for two agents. It broke at three.

**Capability Negotiation** emerged from the realization that not all agents were equal. When multiple agents could fulfill a request, the system needed a way to choose. Simple round-robin worked initially, but failed when some agents were faster, more reliable, or better suited to the specific task. The author added a scoring system that considered capability match, historical success rate, and current load — but it was fragile, and the scoring weights were essentially guesswork.

**Delegation Engine** was the core orchestration logic. It received a high-level task, decomposed it into sub-tasks, queried the registry for specialists, delegated each sub-task, collected results, and assembled the final response. The delegation engine was the brain of the system — it decided how to break down complex requests and who should handle each piece. But early versions had no failure handling. If a specialist agent failed, the entire delegation failed with it. There was no retry logic, no fallback agents, no circuit breaker.

**Context Broker** was the shared state layer that persisted context across agent interactions. It implemented an ACT-R-inspired activation scoring system — frequently accessed context was prioritized, stale context decayed over time. The context broker enabled agents to share information without tight coupling. But it was in-memory only. Context was lost between sessions, requiring re-establishment of state for each new interaction.

```typescript
// jagents — First agent registry (February 2026)
// Note: This was NOT the final architecture.
// The capability-based indexing came later.
// The heartbeat mechanism was added after the first disconnect incident.

class AgentRegistry {
  private agents: Map<string, AgentDefinition> = new Map();
  
  // First attempt: Simple registration
  // Failed: No way to discover agents by capability
  // Required scanning all agents to find matches
  register(agent: AgentDefinition): void {
    this.agents.set(agent.id, agent);
  }
  
  findAgent(id: string): AgentDefinition | null {
    return this.agents.get(id) || null;
  }

  // Second attempt: Manual capability indexing
  // Worked: Faster lookups
  // Problem: Required manual updates when capabilities changed
  private capabilityIndex: Map<string, string[]> = new Map();
  
  registerWithIndex(agent: AgentDefinition): void {
    this.agents.set(agent.id, agent);
    for (const cap of agent.capabilities) {
      if (!this.capabilityIndex.has(cap)) {
        this.capabilityIndex.set(cap, []);
      }
      this.capabilityIndex.get(cap)!.push(agent.id);
    }
  }
  
  // Final approach: Automatic capability indexing with heartbeat
  // Required: Periodic heartbeat to detect dead agents
  // Result: Dynamic, self-healing registry
  register(agent: AgentDefinition): void {
    this.agents.set(agent.id, agent);
    this.broadcastCapability(agent.capabilities);
    this.startHeartbeat(agent.id);
  }
  
  private startHeartbeat(agentId: string): void {
    // Agents must heartbeat every 30 seconds
    // Missing heartbeats trigger removal
    setInterval(() => {
      const agent = this.agents.get(agentId);
      if (agent && Date.now() - agent.lastHeartbeat > 30000) {
        this.agents.delete(agentId);
      }
    }, 10000);
  }
}
```

The first jagents commit (`51169fa`, February 26, 2026) was deceptively simple: a single MCP server named `bash-kanban-mcp-server` that exposed job management tools. It had no registry, no delegation, no context broker. It was a proof that an MCP server could talk to a task runner. The architecture emerged from there — through failure, iteration, and the growing realization that the system needed more than just tool exposure.

---

## The Delegation Pattern

One of the most impactful patterns from jagents was **delegation**. Instead of one agent trying to do everything, the system could delegate sub-tasks to specialized agents:

### The Agent Registry Schema

The agent registry used a simple but powerful schema that defined each agent's capabilities:

```typescript
interface AgentDefinition {
  id: string;                    // Unique identifier
  name: string;                  // Human-readable name
  capabilities: string[];        // List of tool names this agent exposes
  version: string;              // Semantic version
  endpoint: string;             // MCP endpoint URL
  metadata: Record<string, unknown>; // Extra context (load, health, etc.)
}
```

This schema enabled the registry to answer the fundamental question: *which agent can do this?* The capability-based indexing meant that agents could be discovered dynamically — no hard-coded dependencies, no central configuration file that needed updating when new agents were added.

### The Delegation Decision Tree

When a complex request arrived, the delegation engine followed a decision tree:

```
User Request
    ↓
Orchestrator parses intent
    ↓
Decompose into sub-tasks
    ↓
For each sub-task:
    ├─ Query registry for matching capabilities
    ├─ If multiple candidates: negotiate and select best
    ├─ If single candidate: delegate directly
    ├─ If no candidate: escalate or fail gracefully
    ↓
Collect results
    ↓
Assemble final response
```

This pattern meant that the orchestrator only needed to understand high-level task decomposition. The details of *how* each sub-task was accomplished were delegated to specialists. The orchestrator became a conductor, not a musician.

```
User: "Check for security vulnerabilities and summarize the findings"
  ↓
Orchestrator Agent
  ├── Delegates to: security-test-analyst (run vulnerability scan)
  ├── Delegates to: analyst (analyze results)
  └── Delegates to: technical-writer (summarize findings)
  ↓
Consolidated response to user
```

This pattern was revolutionary for the PAI ecosystem. It meant that any agent could coordinate complex workflows by delegating to specialists, without needing to implement every capability itself. The orchestrator agent only needed to understand the high-level task — the details were delegated.

The delegation engine handled the complexity: finding the right specialist, passing context, handling failures, and assembling results. This was the difference between a tool and an agent. A tool executes commands. An agent delegates work.

---

## The Integration

The jagents architecture became the blueprint for future agent systems:

**jay-gentic** adopted the delegation engine and context broker patterns. The agentic CLI could delegate tasks to specialized agents — security scanning, research, content creation — without implementing each capability itself. The migration from the initial MCP server to the markdown-based agent system (`8a957a9`, March 7, 2026) was a direct result of jagents' delegation pattern proving that agents could work together.

**agent-remote-control** used the agent registry for team management. Teams of agents could be created, with each member registered in the registry and capable of receiving delegated tasks.

**memory-system** provided the persistent context layer that jagents had only simulated. The context broker's in-memory store was replaced with a proper database, enabling cross-session context persistence.

The project was a bridge between the "tools" era and the "agents" era — between isolated capabilities and collaborative intelligence. It proved that agents could work together, not just alongside each other.

---

## The First Collaboration (Beyond the Author)

The first time someone else experienced jagents, it was accidental. A colleague was was showing the author how Grok bot worked. The author relized that quite by accident the agent-remote-control could manage multiple independent named conversation theeads. He also realized that the orchestrator agent had received a request to "analyze this security incident," broken it into sub-tasks, delegated each to the appropriate specialist, and assembled the final response — all without human intervention. And more importantly that agent conversationa could be added to group conversation, capabilities that had been built hut not used until then.

"Is that... running by itself?" the colleague asked.

"Yes."

"Without you typing anything?"

"It's delegating to specialists. Each one handles its domain."

The colleague watched as the orchestrator collected results from three different agents — a security scan, a documentation query, and a report generation — and assembled them into a coherent response. The entire workflow took about fifteen seconds.

"Does it... know what it's doing?"

"Yes in a manner of speaking. It's following rules. But the rules are mine. The agents are mine. The output is mine."

The colleague didn't ask to use it. They didn't need to. The demonstration was sufficient. But the look on their face — the realization that this level of coordination was possible without cloud dependency — was the validation the author hadn't known they needed.

---

## The Limitations

jagents was a proof of concept — and a powerful one, but a proof of concept nonetheless. It had limitations that became apparent as the ecosystem grew:

**No persistent memory.** The context broker was in-memory only. Context was lost between sessions, requiring re-establishment of state for each new interaction. This was a significant limitation for long-running workflows.

**No security model.** Agents could call any other agent's capabilities without authentication or authorization. In a multi-user environment, this was a serious vulnerability.

**No standard interface.** Each agent defined its own tool schema, making integration fragile. There was no enforced contract for agent-to-agent communication.

**No fault tolerance.** If a specialist agent failed, the delegation failed with it. There was no retry logic, no fallback agents, no circuit breaker pattern.

These limitations were not failures — they were learning opportunities. Each one informed the design of subsequent systems: jay-gentic added persistent memory and security. agent-remote-control added authentication and team management. The memory system solved the persistence problem.

---

## The Lesson

The hardest part of building AI systems isn't making individual agents capable — it's making them *collaborative*. A single brilliant agent is impressive. A team of specialized agents that can delegate, negotiate, and share context is transformative.

The jagents project teaches us that:

1. **Protocol ≠ Architecture.** MCP defines communication, but you need an architecture for collaboration. The protocol is the syntax; the architecture is the semantics.

2. **Delegation is power.** The ability to delegate sub-tasks is what separates agents from tools. A tool executes commands. An agent delegates work.

3. **Modularity enables growth.** A modular architecture allows new agents to be added without modifying existing ones. The registry pattern enables open extension.

4. **Proof of concept is valuable.** jagents was not production-ready, but it proved the pattern. It provided the blueprint for everything that followed.

5. **Architecture emerges.** The four components — registry, negotiation, delegation, context — didn't arrive fully formed. They emerged from failure, each one responding to a limitation that became visible only after the previous component was in place.

---

## Code Snippets

### Task Delegation with Failure Handling

```typescript
// jagents — Task delegation with failure handling
class DelegationEngine {
  async delegate(task: Task): Promise<Result> {
    const specialists = this.registry.findAllAgents(task.requiredCapability);
    
    for (const specialist of specialists) {
      try {
        const context = await this.broker.shareContext(task.context);
        const result = await specialist.execute(task, context);
        if (result.success) {
          await this.broker.store(task.id, result);
          return result;
        }
      } catch (error) {
        this.log.failure(specialist.id, error);
      }
    }
    
    return { success: false, error: "All specialists failed" };
  }
}
```

### Capability Negotiation

```typescript
// jagents — Capability negotiation
interface CapabilityNegotiation {
  request: CapabilityRequest;
  responses: CapabilityResponse[];
  selected: AgentDefinition;
  agreedProtocol: string;
}

async function negotiate(
  request: CapabilityRequest,
  candidates: AgentDefinition[]
): Promise<CapabilityNegotiation> {
  const responses = await Promise.all(
    candidates.map(a => a.propose(request))
  );
  const selected = selectBest(responses, request.priority);
  return { request, responses, selected, agreedProtocol: selected.protocol };
}
```

### Context Broker with ACT-R Activation Scoring

```typescript
// jagents — Context broker with ACT-R activation scoring
class ContextBroker {
  private context: Map<string, ContextItem> = new Map();
  
  async shareContext(key: string, value: unknown): Promise<void> {
    const item = this.context.get(key) || { value, activation: 0 };
    item.value = value;
    item.activation += 1.0;
    item.lastAccessed = Date.now();
    this.context.set(key, item);
  }
  
  async retrieveContext(key: string): Promise<unknown> {
    const item = this.context.get(key);
    if (!item) return null;
    
    const decay = (Date.now() - item.lastAccessed) / 60000;
    item.activation = Math.max(0, item.activation - decay * 0.1);
    
    return item.value;
  }
}
```

---

## Chapter Closing

The jagents project was born in February 2026. It was not a production system — it was a proof of concept, a demonstration that the pattern was possible. It had limitations: no persistent memory, no security model, no fault tolerance. But it proved something fundamental: agents could collaborate, not just communicate.

That proof of concept would inform every agent system that followed. jay-gentic would add persistent memory and security. agent-remote-control would add authentication and team management. The memory system would solve the persistence problem. But the core insight — that delegation is what separates agents from tools — came from jagents.

The chapters ahead will show you how that insight evolved into a full agent ecosystem. But before that evolution, there was another question forming.

If agents can collaborate, what kind of worlds can they build?

The answer would take a detour — one that seemed unrelated to AI infrastructure, but taught lessons that would shape everything that followed.

---

## Chapter Summary

Chapter 7 documents the construction of jagents, the first system that allowed AI agents to truly collaborate rather than merely communicate. Built on the Model Context Protocol (MCP), jagents went beyond the spec by creating a modular architecture — but not through upfront design. The four key components (Agent Registry, Capability Negotiation, Delegation Engine, Context Broker) emerged gradually through iteration, each one responding to a limitation that became visible only after the previous component was in place. The chapter includes the specific moment of frustration that triggered the project (a 20-minute manual security incident response that should have taken 20 seconds), the failed first attempt at using MCP directly (which worked syntactically but lacked discovery, negotiation, and failure handling), and the evolution of the agent registry from a simple Map to a capability-indexed, heartbeat-enabled system. The delegation pattern was revolutionary — instead of one agent trying to do everything, the system could delegate to specialists, with the orchestrator only needing to understand high-level tasks. The chapter acknowledges jagents' limitations (no persistent memory, no security model, no fault tolerance) as learning opportunities that informed subsequent systems. A brief scene captures the first time a colleague experienced the delegated workflow — the validation that coordination without cloud dependency was possible. The key insight: protocol defines communication syntax, but architecture defines collaboration semantics. A single brilliant agent is impressive; a team of specialized agents that can delegate, negotiate, and share context is transformative.

---

## Key Technical Facts

| Detail | Value |
|--------|-------|
| **Project** | jagents |
| **First Commit** | February 26, 2026 (`51169fa`) |
| **MCP Server Name** | bash-kanban-mcp-server |
| **Key Evolution** | Feb 2026 (MCP server) → Mar 2026 (jay-gentic integration) → Mar 2026 (bash migration) → Apr 2026 (TUI, runner backends) |
| **Language** | TypeScript (MCP server), Bash (task runner) |
| **Key Innovation** | Agent registry, delegation engine, context broker — all emerged iteratively |
| **Protocol** | MCP (Model Context Protocol) |

---

## Narrative Threads to Carry Forward

- The shift from tools to agents
- Delegation as a core pattern
- The need for trust and identity
- Modularity as a growth strategy
- Proof of concept as foundation
- Architecture emerging from failure, not design
