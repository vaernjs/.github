<div align="center">

<a href="https://vaernjs.tech">
  <img src="https://raw.githubusercontent.com/vaernjs/.github/main/assets/vaernjs-logo-white.svg" alt="VaernJS" width="380">
</a>

<br/>
<br/>

### Agent-native JavaScript & TypeScript runtime.
**Guardrails in the engine, not in your app.**

<br/>

[![npm version](https://img.shields.io/npm/v/@vaernjs/vaern?style=flat-square&color=005C41&label=npm)](https://www.npmjs.com/package/@vaernjs/vaern)
[![License: MIT](https://img.shields.io/badge/License-MIT-005C41.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Website](https://img.shields.io/badge/Website-vaernjs.tech-111111?style=flat-square)](https://vaernjs.tech)
[![GitHub Repo](https://img.shields.io/badge/GitHub-vaernjs%2Fvaern-111111?style=flat-square&logo=github)](https://github.com/vaernjs/vaern)
[![Maintained by DevLogix](https://img.shields.io/badge/Maintained%20by-DevLogix-005C41?style=flat-square)](https://devlogix.com.pk)

<br/>

[Overview](#-overview) • [The Authority Problem](#-the-authority-problem) • [Three Engine Primitives](#-three-engine-primitives) • [Quick Start](#-quick-start) • [Radical Honesty](#-what-vaern-is-not) • [Ecosystem](#-repositories) • [Docs & Community](#-documentation--community)

---

</div>

## 🌐 Overview

**Vaern (VaernJS)** is an open-source, agent-native JavaScript and TypeScript runtime built in Rust. 

When autonomous agents run in standard runtimes like Node.js or Bun, every safety check, spend limit, and audit logger runs as ordinary application code. **Vaern moves the guardrails below the application into the engine** — enforcing hard spend limits, producing tamper-evident effect journals, and enabling byte-for-byte offline replays.

Maintained with pride by **DEVLOGIX TECHNOLOGY (PRIVATE) LIMITED** under the **MIT License**.

---

## ⚡ The Authority Problem

In typical agent architectures, guardrails share the same authority as the code they are supposed to constrain:

```typescript
// The conventional pattern (a library check):
if (spent + estimate > limit) throw new Error("budget exceeded");
```

**What authority does that check have over the agent? None.**
- It executes in the same process, in the same language, with the same global memory and network sockets.
- If a prompt injection triggers an unhandled branch, if an asynchronous callback skips validation, or if a tool circumvents the wrapper, **the guardrail is bypassed**.
- When an agent misbehaves at 3 AM, your audit log is merely the application's own unverified testimony.

> **A guardrail that shares authority with the process it guards is not a boundary — it is a convention.**  
> Vaern moves the boundary into the runtime engine itself.

---

## 🛡️ Three Engine Primitives

```
┌────────────────────────────────────────────────────────┐
│                   Your Agent Script                    │
│             (JavaScript / TypeScript / ESM)            │
├────────────────────────────────────────────────────────┤
│           VAERN RUNTIME ENGINE (Rust + QuickJS)        │
│                                                        │
│  💸 Budget Enforcement     🧾 Hash-Chained Journal     │
│     • Pre-dispatch reserve    • SHA-256 chained logs   │
│     • Integer nano-USD        • Protected $VAERN_HOME  │
│     • Hardware sync to disk   • Automatic redactions   │
│                                                        │
│  ⏪ Offline Replay                                     │
│     • Zero network required   • Byte-for-byte verify   │
└────────────────────────────────────────────────────────┘
```

### 1. 💸 Budgets Reserved Before Dispatch
* **Pre-flight reservation:** Before an HTTP request or model call leaves the machine, its estimated cost is checked and reserved on disk.
* **Refusal before cost:** If the budget cannot cover the call, the socket is never opened. The journal records `sent: false` and `charged_nano_usd: 0`.
* **Zero floating-point drift:** All currency calculations use integer **nano-USD** ($10^{-9}$).

### 2. 🧾 Tamper-Evident Effect Journal
* **Cryptographic chaining:** Every side-effect (network fetch, model generation, file I/O, console output) is written to an append-only, SHA-256 hash-chained log.
* **Engine-level immutability:** The running script's filesystem operations inside `$VAERN_HOME` are blocked with `EPERM`. The agent cannot alter its own history or spend ledger.
* **Credential scrubbing:** Authorization headers, bearer tokens, API keys, and secret parameters are redacted automatically before hitting disk.

### 3. ⏪ Deterministic Offline Replay
* **Reproducible execution:** `vaern replay` verifies the journal hash chain, pre-hashes all modules, restores the initial PRNG seed, and replays every effect with **the network completely disconnected**.
* **Byte-for-byte verification:** Re-executes the run and compares standard output and effects against the journal byte-for-byte.

---

## 🚀 Quick Start

### Installation

Pre-built native binaries are published on npm for Linux (x64, arm64), macOS (Apple Silicon, Intel), and Windows (x64):

```bash
# Install globally via npm
npm install -g @vaernjs/vaern

# Verify installation
vaern --version
vaern doctor
```

### Run an Agent with Enforced Budget

```typescript
// agent.ts
import { model } from "vaern:ai";

const gpt = model("gpt-4o", {
  pricePer1kInputNanoUsd: 2500000,
  pricePer1kOutputNanoUsd: 10000000,
});

const response = await gpt.chat("Summarize quarterly risk findings.");
console.log(response.content);
```

Run with a strictly enforced \$0.05 budget:

```bash
# Run with hard budget constraint
vaern run agent.ts --budget-usd 0.05

# Verify cryptographic journal integrity
vaern journal verify latest

# Replay run completely offline (zero network access)
vaern replay latest
```

---

## 🔍 What Vaern Is Not (Radical Honesty)

We believe in documenting limitations with the same rigor as features:

| Limitation | Reality |
|---|---|
| **Not a Node or Bun drop-in** | No CommonJS, no `node:*` built-ins. Pure ES modules with installed `node_modules`. |
| **No JIT (Embedded QuickJS)** | On tight, 50M-iteration arithmetic loops, Vaern is **5.4× slower than Node**. Agent workloads are bound by model & I/O latency, not raw numeric loops. |
| **Journal startup overhead** | Hash-chained journalling adds ~5 ms to startup (can be bypassed with `--no-journal`). |
| **Tamper-evident, not signed** | The hash chain verifies internal consistency. If an external process with root file access modifies both records and recomputes hashes, it can rewrite history. Store the head hash externally for third-party attestation. |
| **In Progress** | Declarative capability sandboxing (`vaern.toml`) and Model Context Protocol (MCP) support are actively under development. |

---

## 📦 Repositories

| Repository | Purpose | Status |
|---|---|---|
| [**`vaernjs/vaern`**](https://github.com/vaernjs/vaern) | Core engine written in Rust, QuickJS runtime integration, CLI, and npm packages | 🟢 Active |

---

## 📚 Documentation & Community

* 🌐 **Official Website:** [vaernjs.tech](https://vaernjs.tech)
* 📖 **Documentation:** [vaernjs.tech/docs](https://vaernjs.tech/docs)
* 📊 **Public Benchmarks:** [vaern/bench](https://github.com/vaernjs/vaern/tree/main/bench)
* 📦 **npm Package:** [@vaernjs/vaern](https://www.npmjs.com/package/@vaernjs/vaern)
* 💬 **Discussions:** [GitHub Discussions](https://github.com/vaernjs/vaern/discussions)
* 🐛 **Issue Tracker:** [GitHub Issues](https://github.com/vaernjs/vaern/issues)

---

<div align="center">

Maintained by **[DEVLOGIX TECHNOLOGY (PRIVATE) LIMITED](https://devlogix.com.pk)**  
Licensed under the [MIT License](https://opensource.org/licenses/MIT).

</div>
