---
title: "jay-gentic v0.4.03.0: Local, Multi-Provider Agentic LLM Framework"
date: 2026-05-04
tags: ["AI Security", "Local LLM", "Agent Orchestration", "Multi-Provider", "DevSecOps", "SMB Tools"]
author: "delphijc"
---

![jay-gentic Framework](jay-gentic-framework.png)

# jay-gentic v0.4.03.0: The Open-Source Agentic LLM Framework for Secure, Multi-Provider Deployments

**Version 0.4.03.0 is now available.** jay-gentic has evolved from a clever bash hack into a production-ready, modular agentic framework that runs on your infrastructure—no external API dependency required. With support for 8 new cloud providers, a 6-layer configuration precedence system, and hardened multi-provider compatibility, it's the tool for organizations that need transparent, auditable AI assistance.

---

## What Is jay-gentic?

jay-gentic is a **modular, shell-native agentic LLM framework** designed for security teams, DevOps engineers, and development organizations that need:

- **Local execution** — Run open-source models on your hardware via Ollama
- **Multi-provider flexibility** — Claude, OpenAI, Mistral, DeepSeek, Gemini, Perplexity, and more
- **Transparent agent orchestration** — Coordinate multiple specialized agents
- **Auditable tool calling** — Every command is logged and verifiable
- **No external API lock-in** — Works with cloud APIs *or* local models, your choice

At its core, jay-gentic is 21 composable bash modules + 3 Go binaries that decompose agent orchestration into testable, reusable pieces.

---

## The Evolution: v0.4.0 → v0.4.03.0

### v0.4.0 (April 4, 2026): The Modular Foundation

v0.4.0 introduced the architectural shift from a monolithic bash script to a **composable framework**:

| Component | Count | Purpose |
|-----------|-------|---------|
| **Bash modules** | 21 | Single-responsibility libraries for agents, memory, permissions, execution, logging |
| **Go binaries** | 3 | Fast SQLite memory (ACT-R), settings parsing, text sanitization |
| **Test suite** | 33 BATS files | Phase-aligned coverage across functionality + security |
| **Documentation** | 12 wiki pages | Getting started, architecture, configuration, troubleshooting |

**Key systems:**
- **Agent system** — Run, validate, and sandbox agent personas with YAML definitions
- **Memory system** — ACT-R activation scoring with episodic memory decay
- **Permissions engine** — Fail-closed enforcement with glob-style rules
- **Tool execution** — Input validation, injection prevention, 30s timeout
- **Voice synthesis bridge** — HTTP bridge to voice-server for audible responses
- **Claude Code compatibility** — 25+ bidirectional tool name mappings

### v0.4.02.2 (April 22, 2026): Provider Stability

v0.4.02.2 hardened multi-provider compatibility across OpenAI, Gemini, and Ollama:

**Fixes:**
- ✅ Duplicate tool call message accumulation (OpenAI)
- ✅ Gemini authentication with provider-specific keys
- ✅ JSON validation and error handling for tool calls
- ✅ Streaming metadata parsing across all providers
- ✅ OpenAI whitespace trimming, duplicate tool registration prevention

**Result:** Reliable tool calling across all major providers, even with streaming enabled.

### v0.4.03.0 (April 23, 2026): Multi-Provider Expansion & Config Architecture

v0.4.03.0 brings **8 new provider integrations**, **intelligent routing**, and **a 6-layer configuration precedence system**:

**8 New Providers:**

| Provider | Endpoint | Default Model |
|----------|----------|---------------|
| **Mistral** | `api.mistral.ai` | `mistral-large-latest` |
| **DeepSeek** | `api.deepseek.com` | `deepseek-chat` |
| **xAI (Grok)** | `api.x.ai` | `grok-3-beta` |
| **Fireworks** | `api.fireworks.ai` | `deepseek-v3` |
| **Perplexity** | `api.perplexity.ai` | `llama-3.1-sonar-large-128k-online` |
| **Azure OpenAI** | configurable | `gpt-4o` |
| **OpenCode Zen** | `opencode.ai/zen/v1` | `big-pickle` |
| **Ollama** | local/remote | configurable |

**Smart Routing for OpenCode Zen:**

```
claude-*  → /v1/messages              (Anthropic format)
gpt-*     → /v1/chat/completions      (OpenAI compat)
*         → /v1/chat/completions      (Fallback)
```

This means you can route different model families to their native endpoints automatically—no manual configuration needed.

**6-Layer Config Precedence:**

```
Layer 1: Defaults (compiled-in)
Layer 2: ~/.claude/config.sh
Layer 3: ~/.jay-gentic/config.sh ← ENTERPRISE_MODE locks here
Layer 4: <project>/.claude/config.sh
Layer 5: <project>/.jay-gentic/config.sh
Layer 6: CLI arguments ← Always wins
```

This enables **fine-grained control**:
- Global defaults apply everywhere
- SMBs can lock organizational policies via ENTERPRISE_MODE
- Projects override globally, preserving global rules
- Individual commands override all

**Multi-Layer Context Aggregation:**

CLAUDE.md, AGENTS.md, and .jay-gentic.md files are aggregated across all directory levels, providing system-wide context without file duplication.

---

## Core Features: A Technical Deep Dive

### 1. Agent Orchestration

Define agents in YAML and coordinate them:

```yaml
# agents/security-analyst.yaml
name: security-analyst
model: sonnet
permissions:
  - "bash(grep*)"
  - "bash(find*)"
  - "jq(*)"
tools:
  - file-read
  - bash-execute
```

Then run them:

```bash
jay-gentic --agent security-analyst --task "Find all recent log anomalies in /var/log"
```

### 2. ACT-R Memory System

Memories score based on **recency** and **frequency**. Frequently accessed facts stay fresh; old facts decay. Backed by Go's SQLite binaries for performance.

```bash
jay-gentic memory add "fact: AWS RDS uses IAM database auth"
jay-gentic memory add "fact: Always rotate credentials quarterly"

# Retrieve top facts by activation score
jay-gentic memory list --top 10 --decay-hours 24
```

### 3. Permission Enforcement

Glob-style rules block dangerous patterns before execution:

```json
{
  "permissions": {
    "blocked": [
      "bash(rm -rf *)",
      "bash(*> /etc/*)",
      "bash(curl | bash)"
    ]
  }
}
```

### 4. Multi-Provider Execution

Run the same prompt against different providers for comparison:

```bash
# Run against local model
jay-gentic --server ollama --model llama3.2 --prompt "Explain Zero Trust"

# Run against Claude
jay-gentic --server claude --model claude-opus-4-6 --prompt "Explain Zero Trust"

# Run against OpenCode Zen
jay-gentic --server opencode --model claude-sonnet-4-5 --prompt "Explain Zero Trust"
```

### 5. Comprehensive Logging

All operations logged as structured JSONL:

```bash
# View logs
tail -f ~/.jay-gentic/logs/jay-gentic.jsonl

# Query logs
jay-gentic log query "error" --limit 100
```

### 6. Tool Execution with Input Validation

Prevents command injection and path traversal:

```bash
# This works:
echo "What files exist in /tmp?" | jay-gentic --one-shot "List files"

# This is blocked:
echo "/tmp; rm -rf /" | jay-gentic --one-shot "Execute this"
```

### 7. Rate Limiting

Prevent runaway tool loops:

```bash
jay-gentic ratelimit check --max-requests 100 --window 3600
```

### 8. Voice Synthesis Bridge

Audible responses via HTTP bridge to voice-server:

```bash
jay-gentic --voice --voice-id "en-US-Neural2-A" "Explain the Zero Trust model"
```

---

## Security for SMBs: Why This Matters

For small and medium-sized organizations, jay-gentic solves critical problems:

### 1. **No API Dependency**
Your security analysis stays on-premises. No cloud logging, no vendor lock-in. Run Llama 3.2 locally for free.

### 2. **Auditable Every Step**
Every prompt, every tool call, every model invocation is logged. Full compliance trail for regulatory requirements (SOC 2, HIPAA, GDPR).

### 3. **Cost-Effective at Scale**
- Local models (Ollama): $0 per inference
- Cloud APIs (Claude, OpenAI): Pay per token
- Mix and match: Use free local models for routine tasks, reserve paid APIs for complex reasoning

### 4. **Transparent Orchestration**
Unlike black-box AI assistants, you see exactly:
- Which agent is running
- Which tool is being called
- What permissions were checked
- Why a request was blocked

### 5. **Multi-Provider Flexibility**
No vendor lock-in. Plug in your preferred LLM:
- Local: Llama, Mistral, DeepSeek via Ollama
- Cloud: Claude, GPT-4o, Gemini, Perplexity
- Routing: Different models for different tasks

---

## Getting Started: Installation & First Run

### Prerequisites

```bash
# Required
bash --version     # 4.0+ (macOS: brew install bash)
curl --version     # Any recent version
jq --version       # 1.6+

# For Go helpers (build once)
go version         # 1.21+

# For local LLM (optional but recommended)
ollama --version
```

### Step 1: Clone jay-gentic

```bash
cd ~/Projects
git clone https://github.com/delphijc/jay-gentic.git
cd jay-gentic
chmod +x jay-gentic
```

### Step 2: Build Go Helper Binaries

```bash
# Build memory, settings, sanitize helpers
cd helpers/jg-memory && go build -o ../bin/jg-memory . && cd ../..
cd helpers/jg-settings && go build -o ../bin/jg-settings . && cd ../..
cd helpers/jg-sanitize && go build -o ../bin/jg-sanitize . && cd ../..

# Verify
ls -la helpers/bin/
```

### Step 3: Pull Local Models (Optional)

```bash
# Install Ollama first: https://ollama.ai

# Pull lightweight models
ollama pull llama3.2           # ~2GB, good for general tasks
ollama pull llama3.2:3b        # ~2GB, lightweight
ollama pull mistral            # ~5GB, strong reasoning
```

### Step 4: Initialize Data Directory

```bash
# Create ~/.jay-gentic/ with standard directories
./jay-gentic --version    # Triggers auto-init

# Verify
ls -la ~/.jay-gentic/
# Should contain: memory/ tools/ hooks/ agents/ skills/ config/ logs/
```

### Step 5: Test Local Execution

```bash
# Query using Ollama
./jay-gentic --server ollama --model llama3.2 --one-shot "What is Zero Trust security?"

# One-shot with Unix pipe
echo "List common cybersecurity frameworks" | ./jay-gentic --server ollama --model llama3.2 --one-shot "Answer:"
```

### Step 6: Configure a Cloud Provider (Optional)

```bash
# Set your API key
export ANTHROPIC_API_KEY="sk-ant-..."

# Test Claude
./jay-gentic --server claude --model claude-opus-4-6 --one-shot "Hello, what is your version?"
```

### Step 7: Run the Test Suite

```bash
# Install bats
brew install bats-core  # macOS
# or: sudo apt install bats  # Linux

# Run tests
cd ~/Projects/jay-gentic
bats tests/phase-1/*.bats
bats tests/security/*.bats

# Run everything
bats tests/**/*.bats
```

---

## Architecture at a Glance

```
jay-gentic (main entry)
├── jay-gentic.d/
│   ├── systems/      (subsystems: agents, memory, hooks, tools)
│   ├── tools/        (command implementations)
│   └── defaults.sh   (built-in config)
├── lib/              (21 composable bash modules)
│   ├── jg-agent.sh
│   ├── jg-memory.sh
│   ├── jg-permissions.sh
│   ├── jg-execute.sh
│   ├── jg-log.sh
│   └── ... (15 more)
├── helpers/bin/
│   ├── jg-memory     (SQLite ACT-R episodic memory)
│   ├── jg-settings   (fast JSON config parsing)
│   └── jg-sanitize   (text escape/injection filtering)
├── config/
│   └── model-routes.json  (provider routing table)
├── tests/            (33 BATS test files)
└── docs/wiki/        (12 documentation pages)
```

---

## Use Cases: What You Can Do Now

### 1. Security Incident Triage
```bash
# Analyze a suspicious log file
cat /var/log/auth.log | jay-gentic --agent security-analyst --prompt "Identify suspicious login attempts"
```

### 2. Code Review with Local Models
```bash
# Review a patch without uploading to cloud
git diff HEAD~1 | jay-gentic --server ollama --model mistral --prompt "Review this code for security issues"
```

### 3. Multi-Provider Comparison
```bash
# Compare responses across models
for model in "llama3.2" "mistral"; do
  echo "=== $model ===" 
  echo "Explain IAM roles" | jay-gentic --server ollama --model $model
done
```

### 4. Automated Compliance Checks
```bash
# Run with custom agent & logging
./jay-gentic --agent compliance-checker \
  --permissions /etc/jay-gentic/policies.json \
  --log-file /var/log/compliance-checks.jsonl
```

### 5. DevSecOps Integration
```bash
# Run in CI/CD pipeline
jay-gentic --server opencode --model big-pickle \
  --no-stream \
  --output json \
  < security-requirements.txt
```

---

## Roadmap: What's Coming

The team is actively working on:

- **Plugin marketplace** — Community-contributed agents and skills
- **Workflow engine v2** — DAG-based execution with conditional branching
- **Memory federation** — Shared memory across jay-gentic and Sam's SQLite store
- **GPU-aware routing** — Automatic model selection based on available hardware
- **Web dashboard** — Monitoring and observability UI

---

## Try It Now

```bash
git clone https://github.com/delphijc/jay-gentic.git
cd jay-gentic
chmod +x jay-gentic

# Run with local model (free)
./jay-gentic --server ollama --model llama3.2

# Or with Claude
export ANTHROPIC_API_KEY="sk-ant-..."
./jay-gentic --server claude --model claude-opus-4-6
```

No Python. No Node. No Docker. Just bash, curl, jq, and an LLM.

---

## Learn More

- **[GitHub Repository](https://github.com/delphijc/jay-gentic)** — Code, issues, contributions
- **[Provider Setup Guide](https://github.com/delphijc/jay-gentic/blob/main/docs/wiki/provider-setup.md)** — Configure cloud providers
- **[Architecture Deep Dive](https://github.com/delphijc/jay-gentic/blob/main/docs/wiki/architecture.md)** — Module organization and design
- **[Security Testing](https://github.com/delphijc/jay-gentic/blob/main/tests/security/)** — See the threat model and test coverage

---

**jay-gentic is MIT licensed. Contributions, issues, and feedback are welcome.**

*For SMBs building secure, auditable AI systems without cloud lock-in, jay-gentic is built for you.*
