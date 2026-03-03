# PRD: Cardinal Element Agent Interface

**Product**: CE Agent Interface
**Version**: 1.0
**Date**: 2026-03-03
**Author**: Cardinal Element Engineering
**Status**: Draft

---

## 1. Overview

### 1.1 Problem Statement

Cardinal Element operates a sophisticated multi-agent ecosystem spanning executive-function AI agents (C-Suite), multi-agent orchestration protocols, evaluation frameworks, recursive loop patterns, and production automation workflows. Today, there is no unified interface to visualize, manage, interact with, or monitor this system. Operators must rely on logs, CLI tools, and manual inspection to understand what agents are doing, how they're coordinating, and whether outcomes meet quality standards.

### 1.2 Vision

A single, real-time interface that serves as the operational command center for the entire CE agent ecosystem — enabling users to observe agent activity, direct orchestration, configure agent roles, evaluate performance, and interact conversationally with any agent or agent team.

### 1.3 Scope

This PRD covers the full interface layer for the CE Agents monorepo, encompassing:

| Component | Upstream Source | Interface Coverage |
|---|---|---|
| Agent Builder (C-Suite) | `ce-c-suite` | Agent configuration, role management, MCP integrations |
| Multi-Agent Orchestration | `coordination-lab` | Orchestration visualization, delegation flows, team topologies |
| Evals | `CE-Evals` | Performance dashboards, rubric management, benchmarking |
| Recursive Loops | Internal | Loop monitoring, convergence tracking, iteration analysis |
| n8n Workflows | n8n instance | Workflow status, trigger management, execution history |

---

## 2. Users & Personas

### 2.1 Primary Users

| Persona | Role | Needs |
|---|---|---|
| **Operator** | Runs and monitors agent systems in production | Real-time visibility into agent states, task queues, errors, and performance. Ability to intervene (pause, restart, redirect agents). |
| **Builder** | Designs and configures agent teams and workflows | Visual tools for defining agent roles, setting orchestration patterns, configuring MCP servers, and designing delegation hierarchies. |
| **Evaluator** | Measures and improves agent quality | Access to eval results, rubric editors, benchmark comparisons, and quality trend analysis. |
| **Executive Stakeholder** | Needs high-level understanding of agent outputs | Summary dashboards, outcome reports, and cost/performance overviews. |

### 2.2 Secondary Users

- **Developers** integrating new agents or MCP servers into the ecosystem
- **Researchers** experimenting with coordination protocols and recursive patterns

---

## 3. Architecture

### 3.1 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CE Agent Interface                        │
│  ┌───────────┬──────────┬───────────┬──────────┬──────────┐ │
│  │ Dashboard │ Topology │  Console  │  Evals   │ Workflow │ │
│  │   View    │   View   │   View    │   View   │   View   │ │
│  └─────┬─────┴────┬─────┴─────┬─────┴────┬─────┴────┬─────┘ │
│        │          │           │          │          │        │
│  ┌─────┴──────────┴───────────┴──────────┴──────────┴─────┐ │
│  │              Unified Event Bus (WebSocket)              │ │
│  └─────┬──────────┬───────────┬──────────┬──────────┬─────┘ │
└────────┼──────────┼───────────┼──────────┼──────────┼───────┘
         │          │           │          │          │
┌────────┴───┐ ┌────┴─────┐ ┌──┴───┐ ┌───┴────┐ ┌───┴──────┐
│  C-Suite   │ │Orchestr. │ │ Eval │ │Recurse │ │   n8n    │
│  Agents    │ │  Engine   │ │Engine│ │ Loops  │ │ Workflows│
└────────────┘ └──────────┘ └──────┘ └────────┘ └──────────┘
```

### 3.2 Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| **Frontend** | React + TypeScript | Component-driven, strong typing, ecosystem maturity |
| **State Management** | Zustand | Lightweight, minimal boilerplate, good for real-time state |
| **Real-time Transport** | WebSocket (Socket.IO) | Bi-directional real-time communication for agent events |
| **Visualization** | React Flow (topology), Recharts (metrics), Monaco (code/config) | Purpose-built libraries for each visualization need |
| **Backend API** | FastAPI (Python) | Matches agent codebase language, async-native, auto-generated OpenAPI |
| **Event Bus** | Redis Streams | Durable event log, supports fan-out to multiple consumers |
| **Persistence** | PostgreSQL + TimescaleDB | Relational data + time-series for metrics/events |
| **Auth** | OAuth 2.0 / API keys | Flexible for both human users and programmatic access |

### 3.3 Data Flow

```
Agent Runtime → Redis Streams → WebSocket Gateway → Browser Client
                     ↓
              PostgreSQL (persistence)
                     ↓
              TimescaleDB (time-series metrics)
```

Every agent action, state transition, message, delegation event, and eval result is published as a structured event to Redis Streams. The WebSocket gateway subscribes and fans out to connected clients. PostgreSQL persists the canonical state; TimescaleDB handles time-series queries for dashboards and trend analysis.

---

## 4. Core Interface Views

### 4.1 Dashboard View — System Overview

The landing page providing at-a-glance health and activity for the entire agent ecosystem.

#### 4.1.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│  CE Agent Interface              [user] [settings] [⋮]  │
├──────┬──────────────────────────────────────────────────┤
│      │  System Health          Active Agents            │
│  N   │  ┌──────────────────┐  ┌──────────────────────┐ │
│  A   │  │ ● All Systems OK │  │ CEO ● | CTO ● |CFO ● │ │
│  V   │  │ Uptime: 99.97%   │  │ COO ● | CMO ○ |CIO ● │ │
│      │  └──────────────────┘  └──────────────────────┘ │
│  B   │                                                  │
│  A   │  Active Tasks              Recent Events         │
│  R   │  ┌──────────────────┐  ┌──────────────────────┐ │
│      │  │ ▶ 12 in progress │  │ 10:31 CEO delegated  │ │
│      │  │ ◼ 3 queued       │  │ 10:30 CTO completed  │ │
│      │  │ ✓ 47 completed   │  │ 10:29 Eval scored    │ │
│      │  └──────────────────┘  └──────────────────────┘ │
│      │                                                  │
│      │  Performance Trends     Resource Utilization     │
│      │  ┌──────────────────┐  ┌──────────────────────┐ │
│      │  │  ╱╲    ╱╲        │  │ Tokens: ████░░ 67%   │ │
│      │  │ ╱  ╲╱╱  ╲╱      │  │ API:    ██████░ 85%  │ │
│      │  │╱         ╲      │  │ Cost:   $42.17 today  │ │
│      │  └──────────────────┘  └──────────────────────┘ │
└──────┴──────────────────────────────────────────────────┘
```

#### 4.1.2 Components

| Component | Description | Data Source |
|---|---|---|
| **System Health** | Aggregate status indicator with uptime percentage | Heartbeat events from all agents |
| **Active Agents** | Grid of agent avatars with status dots (● active, ○ idle, ✕ error) | Agent state events |
| **Active Tasks** | Counts of in-progress, queued, and completed tasks with drill-down | Task lifecycle events |
| **Recent Events** | Reverse-chronological feed of significant system events | Redis Streams (filtered) |
| **Performance Trends** | Time-series charts for throughput, latency, success rate | TimescaleDB aggregations |
| **Resource Utilization** | Token usage, API call rates, cost tracking | Usage metering events |

#### 4.1.3 Interactions

- Click any agent → navigate to Agent Detail View
- Click any task → navigate to Task Detail with full execution trace
- Click any event → expand inline with full context
- Time range selector (1h, 6h, 24h, 7d, 30d) controls all charts
- Auto-refresh toggle (default: 5s interval via WebSocket push)

---

### 4.2 Topology View — Agent Network Visualization

Interactive graph visualization of the agent network, showing relationships, delegation flows, and real-time communication.

#### 4.2.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│ Topology    [Hierarchy ▾] [Live ●] [Filter] [Zoom ±]   │
├──────┬──────────────────────────────────────────────────┤
│      │                                                  │
│  F   │           ┌─────┐                                │
│  I   │           │ CEO │ ◄─── orchestrator              │
│  L   │          ╱│     │╲                               │
│  T   │         ╱ └─────┘ ╲                              │
│  E   │        ╱     │     ╲                             │
│  R   │  ┌─────┐ ┌─────┐ ┌─────┐                       │
│      │  │ CTO │ │ CFO │ │ COO │ ◄─── sub-agents       │
│  P   │  │     │ │     │ │     │                        │
│  A   │  └──┬──┘ └─────┘ └──┬──┘                       │
│  N   │     │                │                           │
│  E   │  ┌──┴──┐         ┌──┴──┐                       │
│  L   │  │ Dev │         │ Ops │ ◄─── leaf agents       │
│      │  │Agent│         │Agent│                        │
│      │  └─────┘         └─────┘                        │
│      │                                                  │
│      │  ─── active delegation  ╌╌╌ idle connection     │
│      │  ━━━ high-traffic path  → message direction     │
└──────┴──────────────────────────────────────────────────┘
```

#### 4.2.2 Components

| Component | Description |
|---|---|
| **Agent Nodes** | Visual representations of each agent with name, role, status indicator, and current task summary. Nodes pulse when actively processing. |
| **Edges / Connections** | Directional lines showing delegation relationships, message flow, and orchestration paths. Line thickness indicates traffic volume. Animated particles show real-time message flow. |
| **Filter Panel** | Left sidebar for filtering by: agent role, status, team, communication protocol, orchestration pattern. |
| **Layout Modes** | Toggle between: Hierarchy (tree), Force-directed (organic), Circular (equal weight), Sequential (pipeline). |
| **Mini-map** | Bottom-right corner navigation for large topologies. |

#### 4.2.3 Real-time Behaviors

- **Message Animation**: When Agent A sends a message to Agent B, an animated particle travels along the edge
- **Status Transitions**: Nodes change color/border in real-time as agent states change
- **Delegation Events**: New edges appear with a fade-in animation when delegation occurs
- **Load Indication**: Edge thickness scales with message frequency (updated every 2s)
- **Error Highlighting**: Nodes/edges involved in errors pulse red with a notification badge

#### 4.2.4 Interactions

- **Hover node** → tooltip with agent summary (role, current task, uptime, message count)
- **Click node** → side panel with agent detail (config, recent messages, metrics)
- **Right-click node** → context menu (pause, restart, reassign task, view logs, open in Console)
- **Drag edge** → manually reroute delegation (with confirmation)
- **Click edge** → view message history between two agents
- **Pinch/scroll zoom** → zoom in/out of topology
- **Shift+drag** → select multiple nodes for batch operations

#### 4.2.5 Team Topology Overlays

Support for visualizing coordination-lab team topologies:

| Topology | Visualization |
|---|---|
| **Hierarchical** | Tree layout with clear reporting lines |
| **Flat/Collaborative** | Force-directed graph with equal node weights |
| **Pipeline** | Left-to-right sequential flow |
| **Hub-and-Spoke** | Central orchestrator with radiating connections |
| **Mesh** | Fully connected graph showing peer-to-peer communication |

---

### 4.3 Console View — Conversational Interface

A chat-based interface for interacting with individual agents or the orchestrated system as a whole.

#### 4.3.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│ Console                        [Agent: CEO ▾] [Team ▾]  │
├──────┬──────────────────────────────────────────────────┤
│      │ ┌──────────────────────────────────────────────┐ │
│  A   │ │ SYSTEM: CEO agent initialized. Ready.        │ │
│  G   │ │                                              │ │
│  E   │ │ USER: Analyze Q4 performance and propose     │ │
│  N   │ │ strategic initiatives for Q1.                │ │
│  T   │ │                                              │ │
│      │ │ CEO: I'll coordinate this across the team.   │ │
│  L   │ │ Delegating financial analysis to CFO and     │ │
│  I   │ │ operational review to COO.                   │ │
│  S   │ │                                              │ │
│  T   │ │   ┌─ DELEGATION ─────────────────────────┐   │ │
│      │ │   │ → CFO: Q4 financial analysis         │   │ │
│      │ │   │ → COO: Q4 operational metrics review │   │ │
│      │ │   │ → CTO: Tech debt assessment          │   │ │
│      │ │   └──────────────────────────────────────┘   │ │
│      │ │                                              │ │
│      │ │ CFO: Q4 revenue was $2.4M, up 18% QoQ...    │ │
│      │ │ [expanding with full analysis]               │ │
│      │ └──────────────────────────────────────────────┘ │
│      │                                                  │
│      │ ┌──────────────────────────────────────────────┐ │
│      │ │ Type a message...              [Send] [⚙️]   │ │
│      │ └──────────────────────────────────────────────┘ │
└──────┴──────────────────────────────────────────────────┘
```

#### 4.3.2 Components

| Component | Description |
|---|---|
| **Agent Selector** | Dropdown to target a specific agent or "System" for orchestrated multi-agent responses |
| **Team Selector** | Choose a pre-configured agent team (e.g., "Executive Team", "Tech Review Board") |
| **Message Thread** | Chronological message display with clear attribution to each agent. Supports markdown, tables, code blocks, and embedded charts. |
| **Delegation Blocks** | Inline visual indicators when an agent delegates subtasks, showing what was delegated to whom |
| **Sub-agent Threads** | Collapsible nested threads showing sub-agent work. Expand to see full sub-agent conversations and reasoning. |
| **Input Bar** | Message input with send button, attachment support, and settings toggle |

#### 4.3.3 Message Types

| Type | Rendering |
|---|---|
| **User Message** | Right-aligned, blue background |
| **Agent Response** | Left-aligned, agent-colored border with avatar and role badge |
| **Delegation Event** | Centered card showing delegation tree with status indicators |
| **Sub-agent Report** | Collapsible section with agent attribution and completion status |
| **System Event** | Centered, muted text for state changes and notifications |
| **Eval Annotation** | Inline quality score badge attached to agent responses |
| **Error** | Red-bordered alert with error details and suggested actions |

#### 4.3.4 Multi-Agent Conversation Features

- **Threaded Delegation Tracking**: When the CEO delegates to CFO, CTO, COO, each sub-conversation is trackable as a nested thread. Users can expand/collapse sub-agent work.
- **Agent Thinking Visibility**: Toggle to show/hide agent reasoning chains and internal deliberation (similar to Claude's extended thinking).
- **Recursive Loop Indicator**: When an agent enters a recursive refinement loop, a progress indicator shows iteration count, convergence score, and estimated remaining iterations.
- **Cross-agent References**: When one agent references another agent's output, a clickable link connects to the source message.
- **Intervention Controls**: At any point in a multi-agent conversation, the user can pause execution, redirect an agent, override a delegation decision, or inject new context.

#### 4.3.5 Input Capabilities

- `/ask <agent>` — direct a question to a specific agent
- `/delegate <task> to <agent>` — manually trigger a delegation
- `/pause` — pause all active agent work
- `/resume` — resume paused work
- `/eval` — trigger an evaluation of the last agent response
- `/topology` — show current agent topology inline
- `/cost` — show token/cost summary for current conversation
- Attachment support: files, images, URLs, structured data (CSV, JSON)

---

### 4.4 Agent Detail View

Deep-dive into a single agent's configuration, state, and history.

#### 4.4.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│ ← Back    Agent: CTO                    [Edit] [⋮]     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌── Identity ──────┐  ┌── Status ──────────────────┐  │
│  │ Role: CTO        │  │ State: Active ●            │  │
│  │ Model: opus-4.6  │  │ Current Task: Tech review  │  │
│  │ Temp: 0.7        │  │ Uptime: 4h 23m             │  │
│  │ MCP: github,     │  │ Tasks Done: 14             │  │
│  │      linear,     │  │ Avg Latency: 2.3s          │  │
│  │      sentry      │  │ Eval Score: 92/100         │  │
│  └──────────────────┘  └────────────────────────────┘  │
│                                                         │
│  ┌── System Prompt ─────────────────────────────────┐  │
│  │ You are the CTO agent. Your responsibilities     │  │
│  │ include technical strategy, architecture review,  │  │
│  │ engineering team oversight...                     │  │
│  │                                   [Edit] [Copy]  │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  [Messages] [Tasks] [Delegations] [Metrics] [Logs]     │
│  ┌──────────────────────────────────────────────────┐  │
│  │ Tab content area                                  │  │
│  │ ...                                               │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

#### 4.4.2 Tabs

| Tab | Content |
|---|---|
| **Messages** | Full message history for this agent, including messages sent and received from other agents and users |
| **Tasks** | List of all tasks this agent has handled, with status, duration, and outcome |
| **Delegations** | Tree view of all delegations this agent has initiated or received, with statuses |
| **Metrics** | Agent-specific performance charts: response time, token usage, eval scores over time, success rate |
| **Logs** | Raw structured logs from this agent's execution, with severity filtering and search |

#### 4.4.3 Configuration Editing

Agents are configurable through the interface:

| Field | Type | Description |
|---|---|---|
| **Name** | string | Display name of the agent |
| **Role** | string | Functional role (CEO, CTO, CFO, etc.) |
| **Model** | select | Underlying LLM model (opus-4.6, sonnet-4.6, haiku-4.5) |
| **Temperature** | slider (0-1) | Response creativity/determinism |
| **System Prompt** | textarea | Core instructions defining agent behavior |
| **MCP Servers** | multi-select | Connected MCP tool servers (GitHub, Linear, Sentry, etc.) |
| **Delegation Rules** | rule builder | Conditions under which this agent delegates to others |
| **Max Concurrent Tasks** | number | Concurrency limit |
| **Timeout** | duration | Maximum time per task before escalation |
| **Eval Rubric** | select | Which evaluation rubric applies to this agent's outputs |

---

### 4.5 Orchestration View

Dedicated view for designing, monitoring, and managing multi-agent orchestration patterns.

#### 4.5.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│ Orchestration          [New Pattern ▾] [Import] [Run]   │
├──────┬──────────────────────────────────────────────────┤
│      │                                                  │
│  P   │  ┌─────────────────────────────────────────────┐│
│  A   │  │          Orchestration Canvas                ││
│  T   │  │                                             ││
│  T   │  │   [Trigger] ──→ [CEO] ──→ [Router]         ││
│  E   │  │                           ╱  │  ╲          ││
│  R   │  │                     [CTO] [CFO] [COO]      ││
│  N   │  │                       │     │     │         ││
│  S   │  │                     [Merge] ◄─────┘         ││
│      │  │                       │                      ││
│  L   │  │                  [Synthesize]                ││
│  I   │  │                       │                      ││
│  B   │  │                    [Output]                  ││
│  R   │  │                                             ││
│  A   │  └─────────────────────────────────────────────┘│
│  R   │                                                  │
│  Y   │  Properties ─────────────────────────────────── │
│      │  Node: Router                                    │
│      │  Type: Conditional Delegation                    │
│      │  Rules: financial → CFO, technical → CTO, ...   │
└──────┴──────────────────────────────────────────────────┘
```

#### 4.5.2 Components

| Component | Description |
|---|---|
| **Patterns Library** | Left sidebar listing saved orchestration patterns: Sequential, Parallel Fan-out, Hierarchical Delegation, Consensus, Debate, Iterative Refinement, Pipeline, Map-Reduce |
| **Orchestration Canvas** | Visual drag-and-drop canvas for designing agent workflows. Nodes represent agents or control-flow operators (router, merger, synthesizer, conditional, loop). Edges represent data/message flow. |
| **Node Types** | Agent Node, Trigger Node, Router (conditional delegation), Merger (aggregate results), Synthesizer (combine outputs), Loop (recursive iteration), Gate (approval checkpoint), Output Node |
| **Properties Panel** | Bottom panel showing configuration for the selected node — routing rules, merge strategies, loop conditions, etc. |
| **Execution Overlay** | When a pattern is running, nodes light up as they activate. Edges animate with data flow. Each node shows real-time status (waiting, active, complete, error). |

#### 4.5.3 Orchestration Patterns (Pre-built)

| Pattern | Description | Use Case |
|---|---|---|
| **Hierarchical Delegation** | CEO routes to specialist agents based on task domain | Strategic planning, cross-functional projects |
| **Parallel Fan-out** | Task broadcast to multiple agents simultaneously, results merged | Research, data analysis, multi-perspective review |
| **Sequential Pipeline** | Output of one agent feeds as input to the next | Document processing, staged analysis |
| **Debate/Adversarial** | Two agents argue opposing positions, a judge agent synthesizes | Decision-making, risk assessment |
| **Consensus** | All agents must agree before proceeding | High-stakes decisions, compliance review |
| **Iterative Refinement** | Agent refines output through recursive loops until convergence | Content creation, code review, optimization |
| **Map-Reduce** | Task split across agents, results reduced into final output | Large-scale analysis, batch processing |
| **Escalation Chain** | Task escalates through agent hierarchy if lower-level agents can't resolve | Support, incident response |

#### 4.5.4 Orchestration Controls

- **Run** — Execute the pattern with given inputs
- **Pause** — Pause mid-execution at the current node
- **Step** — Execute one node at a time (debug mode)
- **Replay** — Re-run a completed execution with the same or modified inputs
- **Fork** — Clone a running execution to test alternative paths
- **Dry Run** — Simulate execution without calling LLMs (validates flow logic)

---

### 4.6 Eval View — Evaluation & Quality Dashboard

Interface for CE-Evals integration — measuring, tracking, and improving agent quality.

#### 4.6.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│ Evaluations          [New Eval ▾] [Rubrics] [Benchmarks]│
├──────┬──────────────────────────────────────────────────┤
│      │                                                  │
│  R   │  Overall Quality Score                           │
│  U   │  ┌──────────────────────────────────────────┐   │
│  B   │  │        92 / 100  (▲ 3 from last week)    │   │
│  R   │  └──────────────────────────────────────────┘   │
│  I   │                                                  │
│  C   │  Agent Scores                                    │
│  S   │  ┌──────────────────────────────────────────┐   │
│      │  │ CEO  ████████████████████░░░  89          │   │
│  L   │  │ CTO  █████████████████████░░  94          │   │
│  I   │  │ CFO  ███████████████████░░░░  86          │   │
│  S   │  │ COO  ████████████████████░░░  91          │   │
│  T   │  │ CMO  ██████████████████░░░░░  83          │   │
│      │  └──────────────────────────────────────────┘   │
│      │                                                  │
│      │  Quality Trends          Rubric Breakdown        │
│      │  ┌──────────────────┐  ┌──────────────────────┐ │
│      │  │     ╱‾╲  ╱‾‾     │  │ Accuracy:     95     │ │
│      │  │   ╱    ╲╱        │  │ Relevance:    91     │ │
│      │  │ ╱                │  │ Completeness: 88     │ │
│      │  └──────────────────┘  │ Coherence:    94     │ │
│      │                        └──────────────────────┘ │
└──────┴──────────────────────────────────────────────────┘
```

#### 4.6.2 Components

| Component | Description |
|---|---|
| **Overall Score** | Weighted aggregate quality score across all agents and rubrics |
| **Agent Scores** | Per-agent quality scores with bar chart visualization and trend indicators |
| **Quality Trends** | Time-series chart showing quality scores over time, with annotations for config changes |
| **Rubric Breakdown** | Scores decomposed by rubric dimension (accuracy, relevance, completeness, coherence, safety, etc.) |
| **Eval History** | Table of past evaluations with filters for agent, rubric, date range, score range |
| **Rubric Editor** | Form-based editor for creating and modifying evaluation rubrics |
| **Benchmark Comparisons** | Side-by-side comparison of agent performance across different model versions, prompts, or configurations |
| **Regression Alerts** | Highlighted alerts when quality scores drop below thresholds or show downward trends |

#### 4.6.3 Eval Triggers

| Trigger | Description |
|---|---|
| **Manual** | User triggers eval on a specific response or conversation via UI button |
| **Automatic** | Every Nth agent response is automatically evaluated |
| **Continuous** | All agent responses evaluated in background (async, non-blocking) |
| **Scheduled** | Batch evaluations run on a schedule (daily, weekly) |
| **Threshold-based** | Re-eval triggered when confidence or quality indicators drop |

---

### 4.7 Recursive Loop Monitor

Specialized view for monitoring and managing recursive agent execution patterns.

#### 4.7.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│ Recursive Loops                    [Active: 3] [History]│
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Loop: Code Review Refinement            Status: Active │
│  Agent: CTO                          Iteration: 4 / 10 │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Convergence                                      │   │
│  │ Score                                            │   │
│  │  1.0 ┤                          ●──●             │   │
│  │  0.8 ┤                    ●──●╱                  │   │
│  │  0.6 ┤              ●──●╱                        │   │
│  │  0.4 ┤        ●──●╱                              │   │
│  │  0.2 ┤  ●──●╱                                    │   │
│  │  0.0 ┤●╱                                         │   │
│  │      └──┬──┬──┬──┬──┬──┬──┬──┬──┬── Iteration   │   │
│  │         1  2  3  4  5  6  7  8  9                │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  Iteration Details                                      │
│  ┌─────────────────────────────────────────────────┐   │
│  │ #1  Score: 0.12  Δ: —     Duration: 3.2s        │   │
│  │ #2  Score: 0.31  Δ: +0.19 Duration: 2.8s        │   │
│  │ #3  Score: 0.58  Δ: +0.27 Duration: 3.1s        │   │
│  │ #4  Score: 0.74  Δ: +0.16 Duration: 2.9s  ← now │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  [Pause] [Skip to Converge] [Set Max Iterations] [Stop]│
└─────────────────────────────────────────────────────────┘
```

#### 4.7.2 Components

| Component | Description |
|---|---|
| **Active Loops List** | All currently running recursive loops with agent, task, iteration count, and convergence % |
| **Convergence Chart** | Real-time line chart showing convergence score over iterations. Threshold line indicates the convergence target. |
| **Iteration Details** | Per-iteration breakdown: score, delta from previous, duration, token cost, and expandable output diff |
| **Output Diff** | Side-by-side diff of the agent's output between consecutive iterations, highlighting what changed |
| **Loop Controls** | Pause, resume, stop, skip-to-convergence (accept current output), adjust max iterations |
| **Loop Configuration** | Convergence threshold, max iterations, cooling schedule, early-stop conditions |

#### 4.7.3 Loop Types

| Type | Description |
|---|---|
| **Self-Critique** | Agent critiques its own output and refines based on self-identified issues |
| **Eval-Driven** | External eval scores the output; agent refines until eval score exceeds threshold |
| **Peer Review** | Another agent reviews and provides feedback; original agent incorporates feedback |
| **Adversarial** | A challenger agent attempts to find flaws; original agent addresses challenges |
| **Convergence** | Agent iterates until consecutive outputs have minimal diff (semantic or textual) |

---

### 4.8 Workflow View — n8n Integration

Interface for monitoring and managing n8n automation workflows that integrate with the agent system.

#### 4.8.1 Components

| Component | Description |
|---|---|
| **Workflow List** | All n8n workflows with status (active, inactive, error), last execution time, and trigger type |
| **Execution History** | Timeline of workflow executions with status, duration, and linked agent interactions |
| **Trigger Management** | Configure workflow triggers: webhook, schedule, agent event, manual |
| **Agent-Workflow Mapping** | Visual mapping showing which agents can trigger which workflows and vice versa |
| **Error Log** | Workflow execution errors with stack traces and suggested fixes |

---

## 5. Data Models

### 5.1 Core Entities

```typescript
// Agent definition and runtime state
interface Agent {
  id: string;                    // uuid
  name: string;                  // "Chief Technology Officer"
  role: AgentRole;               // CEO | CTO | CFO | COO | CMO | CIO | custom
  status: AgentStatus;           // active | idle | paused | error | terminated
  model: string;                 // "claude-opus-4-6"
  temperature: number;           // 0.0 - 1.0
  systemPrompt: string;
  mcpServers: MCPServer[];
  delegationRules: DelegationRule[];
  maxConcurrentTasks: number;
  timeout: number;               // ms
  evalRubricId: string;
  metadata: Record<string, unknown>;
  createdAt: DateTime;
  updatedAt: DateTime;
}

// Message between any participants (user, agents, system)
interface Message {
  id: string;
  conversationId: string;
  parentMessageId: string | null;  // for threading
  sender: {
    type: 'user' | 'agent' | 'system';
    agentId?: string;
  };
  recipient: {
    type: 'user' | 'agent' | 'broadcast';
    agentId?: string;
  };
  content: string;                 // markdown
  contentType: MessageContentType; // text | delegation | report | error | eval
  attachments: Attachment[];
  metadata: Record<string, unknown>;
  createdAt: DateTime;
}

// Task assigned to an agent
interface Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;             // queued | in_progress | completed | failed | cancelled
  assignedAgentId: string;
  parentTaskId: string | null;    // for delegation chains
  delegatedFrom: string | null;   // agentId that delegated this task
  conversationId: string;
  priority: number;               // 0-100
  input: unknown;
  output: unknown;
  error: string | null;
  startedAt: DateTime | null;
  completedAt: DateTime | null;
  tokenUsage: TokenUsage;
  evalScore: number | null;
  createdAt: DateTime;
}

// Orchestration pattern definition
interface OrchestrationPattern {
  id: string;
  name: string;
  description: string;
  type: PatternType;              // hierarchical | fan_out | pipeline | debate | consensus | iterative | map_reduce | escalation
  nodes: OrchestrationNode[];
  edges: OrchestrationEdge[];
  config: Record<string, unknown>;
  createdAt: DateTime;
  updatedAt: DateTime;
}

interface OrchestrationNode {
  id: string;
  type: NodeType;                 // agent | trigger | router | merger | synthesizer | loop | gate | output
  agentId?: string;               // if type is 'agent'
  position: { x: number; y: number };
  config: Record<string, unknown>;
}

interface OrchestrationEdge {
  id: string;
  sourceNodeId: string;
  targetNodeId: string;
  condition?: string;             // routing condition expression
  label?: string;
}

// Orchestration execution instance
interface Execution {
  id: string;
  patternId: string;
  status: ExecutionStatus;        // running | paused | completed | failed | cancelled
  input: unknown;
  output: unknown;
  nodeStates: Map<string, NodeExecutionState>;
  startedAt: DateTime;
  completedAt: DateTime | null;
  totalTokenUsage: TokenUsage;
  totalCost: number;
}

interface NodeExecutionState {
  nodeId: string;
  status: 'waiting' | 'active' | 'completed' | 'error' | 'skipped';
  input: unknown;
  output: unknown;
  startedAt: DateTime | null;
  completedAt: DateTime | null;
  error: string | null;
}

// Delegation event
interface Delegation {
  id: string;
  fromAgentId: string;
  toAgentId: string;
  taskId: string;
  reason: string;
  status: DelegationStatus;       // pending | accepted | rejected | completed | failed
  createdAt: DateTime;
  resolvedAt: DateTime | null;
}

// Recursive loop tracking
interface RecursiveLoop {
  id: string;
  agentId: string;
  taskId: string;
  type: LoopType;                 // self_critique | eval_driven | peer_review | adversarial | convergence
  status: 'running' | 'converged' | 'max_iterations' | 'stopped' | 'failed';
  currentIteration: number;
  maxIterations: number;
  convergenceThreshold: number;
  iterations: LoopIteration[];
  createdAt: DateTime;
  completedAt: DateTime | null;
}

interface LoopIteration {
  number: number;
  input: string;
  output: string;
  score: number;
  delta: number;
  duration: number;               // ms
  tokenUsage: TokenUsage;
  feedback: string | null;        // from critic/peer/eval
  timestamp: DateTime;
}

// Evaluation result
interface EvalResult {
  id: string;
  agentId: string;
  taskId: string;
  messageId: string;
  rubricId: string;
  overallScore: number;           // 0-100
  dimensionScores: {
    dimension: string;            // accuracy, relevance, completeness, coherence, safety
    score: number;
    reasoning: string;
  }[];
  evaluatorModel: string;
  evaluatedAt: DateTime;
}

// Evaluation rubric definition
interface EvalRubric {
  id: string;
  name: string;
  description: string;
  dimensions: {
    name: string;
    weight: number;               // 0-1, must sum to 1
    criteria: string;
    scoringGuide: {
      score: number;
      description: string;
    }[];
  }[];
  createdAt: DateTime;
  updatedAt: DateTime;
}

// MCP server connection
interface MCPServer {
  id: string;
  name: string;                   // "github", "linear", "sentry"
  uri: string;
  protocol: 'stdio' | 'sse' | 'streamable-http';
  tools: MCPTool[];
  status: 'connected' | 'disconnected' | 'error';
  config: Record<string, unknown>;
}

interface MCPTool {
  name: string;
  description: string;
  inputSchema: Record<string, unknown>;
}

// Token usage tracking
interface TokenUsage {
  inputTokens: number;
  outputTokens: number;
  totalTokens: number;
  estimatedCost: number;          // USD
}

// System event for the event bus
interface AgentEvent {
  id: string;
  type: EventType;
  timestamp: DateTime;
  source: {
    type: 'agent' | 'system' | 'user' | 'workflow';
    id: string;
  };
  data: Record<string, unknown>;
}

type EventType =
  | 'agent.started'
  | 'agent.stopped'
  | 'agent.error'
  | 'agent.state_changed'
  | 'task.created'
  | 'task.started'
  | 'task.completed'
  | 'task.failed'
  | 'delegation.created'
  | 'delegation.accepted'
  | 'delegation.completed'
  | 'delegation.rejected'
  | 'message.sent'
  | 'message.received'
  | 'loop.started'
  | 'loop.iteration'
  | 'loop.converged'
  | 'loop.stopped'
  | 'eval.completed'
  | 'eval.regression'
  | 'workflow.triggered'
  | 'workflow.completed'
  | 'workflow.error'
  | 'mcp.connected'
  | 'mcp.disconnected'
  | 'mcp.tool_called';
```

---

## 6. API Specification

### 6.1 REST Endpoints

#### Agents
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/agents` | List all agents (with filtering & pagination) |
| `POST` | `/api/agents` | Create a new agent |
| `GET` | `/api/agents/:id` | Get agent details |
| `PATCH` | `/api/agents/:id` | Update agent configuration |
| `DELETE` | `/api/agents/:id` | Remove an agent |
| `POST` | `/api/agents/:id/start` | Start an agent |
| `POST` | `/api/agents/:id/stop` | Stop an agent |
| `POST` | `/api/agents/:id/pause` | Pause an agent |
| `POST` | `/api/agents/:id/resume` | Resume a paused agent |
| `GET` | `/api/agents/:id/messages` | Get agent message history |
| `GET` | `/api/agents/:id/tasks` | Get agent task history |
| `GET` | `/api/agents/:id/metrics` | Get agent performance metrics |
| `GET` | `/api/agents/:id/logs` | Get agent execution logs |

#### Conversations
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/conversations` | List conversations |
| `POST` | `/api/conversations` | Start a new conversation |
| `GET` | `/api/conversations/:id` | Get conversation with messages |
| `POST` | `/api/conversations/:id/messages` | Send a message |
| `DELETE` | `/api/conversations/:id` | Delete a conversation |

#### Tasks
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/tasks` | List tasks (with filtering & pagination) |
| `GET` | `/api/tasks/:id` | Get task detail with execution trace |
| `POST` | `/api/tasks/:id/cancel` | Cancel a running task |
| `GET` | `/api/tasks/:id/delegation-tree` | Get full delegation tree for a task |

#### Orchestration
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/orchestration/patterns` | List orchestration patterns |
| `POST` | `/api/orchestration/patterns` | Create a new pattern |
| `GET` | `/api/orchestration/patterns/:id` | Get pattern detail |
| `PATCH` | `/api/orchestration/patterns/:id` | Update a pattern |
| `DELETE` | `/api/orchestration/patterns/:id` | Delete a pattern |
| `POST` | `/api/orchestration/patterns/:id/execute` | Execute a pattern |
| `POST` | `/api/orchestration/patterns/:id/dry-run` | Dry-run a pattern |
| `GET` | `/api/orchestration/executions` | List executions |
| `GET` | `/api/orchestration/executions/:id` | Get execution detail |
| `POST` | `/api/orchestration/executions/:id/pause` | Pause execution |
| `POST` | `/api/orchestration/executions/:id/resume` | Resume execution |
| `POST` | `/api/orchestration/executions/:id/stop` | Stop execution |

#### Evaluations
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/evals` | List evaluation results |
| `POST` | `/api/evals` | Trigger a new evaluation |
| `GET` | `/api/evals/:id` | Get evaluation detail |
| `GET` | `/api/evals/rubrics` | List rubrics |
| `POST` | `/api/evals/rubrics` | Create a rubric |
| `PATCH` | `/api/evals/rubrics/:id` | Update a rubric |
| `GET` | `/api/evals/benchmarks` | Get benchmark comparisons |
| `GET` | `/api/evals/trends` | Get quality trend data |

#### Recursive Loops
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/loops` | List recursive loops |
| `GET` | `/api/loops/:id` | Get loop detail with iterations |
| `POST` | `/api/loops/:id/pause` | Pause a loop |
| `POST` | `/api/loops/:id/resume` | Resume a loop |
| `POST` | `/api/loops/:id/stop` | Stop a loop |
| `PATCH` | `/api/loops/:id` | Update loop config (max iterations, threshold) |

#### System
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/system/health` | System health check |
| `GET` | `/api/system/metrics` | Aggregate system metrics |
| `GET` | `/api/system/events` | Paginated event history |
| `GET` | `/api/system/config` | System configuration |

### 6.2 WebSocket Events

Connection: `ws://host/ws`

#### Client → Server
| Event | Payload | Description |
|---|---|---|
| `subscribe` | `{ channels: string[] }` | Subscribe to event channels |
| `unsubscribe` | `{ channels: string[] }` | Unsubscribe from channels |
| `send_message` | `{ conversationId, content, targetAgentId? }` | Send a message |
| `agent_command` | `{ agentId, command: 'pause'|'resume'|'stop' }` | Control an agent |

#### Server → Client
| Event | Payload | Description |
|---|---|---|
| `agent_event` | `AgentEvent` | Any agent lifecycle event |
| `message` | `Message` | New message in a subscribed conversation |
| `task_update` | `Task` | Task status change |
| `delegation_update` | `Delegation` | Delegation status change |
| `loop_update` | `{ loopId, iteration: LoopIteration }` | New loop iteration |
| `eval_result` | `EvalResult` | New evaluation result |
| `metrics_update` | `{ metrics: SystemMetrics }` | Periodic metrics push |
| `system_alert` | `{ severity, message, source }` | System alerts and errors |

#### Channels
- `system` — system-wide events
- `agent:{agentId}` — events for a specific agent
- `conversation:{conversationId}` — messages in a conversation
- `execution:{executionId}` — orchestration execution events
- `loop:{loopId}` — recursive loop events
- `evals` — evaluation results

---

## 7. Non-Functional Requirements

### 7.1 Performance

| Metric | Target |
|---|---|
| Dashboard load time | < 2s (initial), < 500ms (subsequent) |
| WebSocket event latency | < 100ms from event occurrence to UI render |
| Topology rendering | Smooth 60fps for up to 50 agent nodes |
| API response time | p50 < 100ms, p99 < 500ms |
| Concurrent WebSocket connections | Support 100+ simultaneous clients |
| Event throughput | Process 10,000+ events/second through Redis Streams |

### 7.2 Reliability

| Requirement | Description |
|---|---|
| WebSocket reconnection | Auto-reconnect with exponential backoff, state sync on reconnect |
| Event durability | All events persisted to Redis Streams with 7-day retention |
| Graceful degradation | UI remains functional if individual backend services are down (show cached data + status indicators) |
| Optimistic updates | UI updates immediately on user actions, rolls back on server rejection |

### 7.3 Security

| Requirement | Description |
|---|---|
| Authentication | OAuth 2.0 with PKCE for browser clients, API keys for programmatic access |
| Authorization | Role-based access control (admin, operator, viewer) |
| Transport | TLS 1.3 for all connections (HTTPS, WSS) |
| Input validation | All user inputs validated and sanitized server-side |
| Audit log | All state-changing operations logged with user attribution |
| Secrets management | MCP server credentials and API keys stored encrypted, never exposed to frontend |

### 7.4 Accessibility

| Requirement | Description |
|---|---|
| WCAG Level | AA compliance |
| Keyboard navigation | Full keyboard navigation for all views |
| Screen reader | Semantic HTML, ARIA labels, live regions for real-time updates |
| Color contrast | 4.5:1 minimum contrast ratio; status indicators use shape + color (not color alone) |
| Reduced motion | Respect `prefers-reduced-motion`; disable animations when set |

### 7.5 Scalability

| Dimension | Approach |
|---|---|
| Agents | Support up to 100 concurrent agents without performance degradation |
| Messages | Paginated loading, virtual scrolling for large conversation histories |
| Metrics | TimescaleDB continuous aggregates for efficient time-series queries at any zoom level |
| Events | Redis Streams consumer groups for horizontal scaling of event processing |

---

## 8. Implementation Phases

### Phase 1: Foundation (Weeks 1-4)

**Goal**: Core infrastructure and basic monitoring

- [ ] Project scaffolding (React + TypeScript frontend, FastAPI backend)
- [ ] Database schema and migrations (PostgreSQL + TimescaleDB)
- [ ] Redis Streams event bus setup
- [ ] WebSocket gateway with channel subscriptions
- [ ] Authentication and authorization
- [ ] Agent CRUD API endpoints
- [ ] Dashboard View with system health, active agents, and event feed
- [ ] Agent Detail View (read-only)
- [ ] Basic navigation and layout shell

**Deliverable**: Users can view agent status and system health in real-time

### Phase 2: Interaction (Weeks 5-8)

**Goal**: Conversational interface and agent management

- [ ] Console View with message threading
- [ ] Multi-agent conversation support (delegation blocks, sub-agent threads)
- [ ] Agent configuration editing (system prompt, model, temperature, MCP servers)
- [ ] Task management API and UI
- [ ] Agent start/stop/pause controls
- [ ] Conversation history and search
- [ ] Slash command support in Console

**Deliverable**: Users can interact with agents conversationally and manage their configuration

### Phase 3: Orchestration (Weeks 9-12)

**Goal**: Visual orchestration builder and execution monitoring

- [ ] Topology View with React Flow
- [ ] Real-time message animation on topology edges
- [ ] Team topology overlay modes
- [ ] Orchestration View with drag-and-drop canvas
- [ ] Pre-built orchestration pattern library
- [ ] Orchestration execution engine integration
- [ ] Execution monitoring with node-level status
- [ ] Orchestration controls (pause, step, replay, dry-run)

**Deliverable**: Users can design, visualize, and monitor multi-agent orchestration patterns

### Phase 4: Quality & Optimization (Weeks 13-16)

**Goal**: Evaluation dashboards, recursive loop monitoring, and production hardening

- [ ] Eval View with quality scores and trends
- [ ] Rubric editor and management
- [ ] Benchmark comparison tools
- [ ] Regression alerting
- [ ] Recursive Loop Monitor with convergence charts
- [ ] Loop controls and configuration
- [ ] n8n Workflow View integration
- [ ] Performance optimization (virtual scrolling, lazy loading, caching)
- [ ] Accessibility audit and remediation
- [ ] Production deployment configuration

**Deliverable**: Complete interface with evaluation, recursive loop monitoring, and production readiness

---

## 9. Success Metrics

| Metric | Target | Measurement |
|---|---|---|
| **Mean time to detect agent issues** | < 30 seconds (down from minutes via logs) | Time from error event to operator awareness |
| **Agent configuration time** | < 2 minutes per agent (down from editing config files) | Time to modify and deploy an agent configuration change |
| **Orchestration pattern creation** | < 10 minutes for standard patterns | Time from intent to running orchestration |
| **Eval review cadence** | Daily (up from ad-hoc) | Frequency of eval dashboard visits |
| **Intervention response time** | < 1 minute from issue to pause/redirect | Time from error notification to operator action |
| **User satisfaction** | > 4.0/5.0 | Internal survey of operators and builders |

---

## 10. Open Questions

| # | Question | Impact | Decision Needed By |
|---|---|---|---|
| 1 | Should the interface support multi-tenant agent deployments, or is single-tenant sufficient for V1? | Architecture, auth model | Phase 1 start |
| 2 | What is the deployment target — self-hosted, cloud-hosted, or both? | Infrastructure, packaging | Phase 1 start |
| 3 | Should the Orchestration Canvas support custom code nodes (user-defined Python/JS functions), or only pre-built node types? | Orchestration View scope | Phase 3 start |
| 4 | What level of n8n integration is needed — read-only monitoring, or full bidirectional control? | Workflow View scope | Phase 4 start |
| 5 | Should eval results be visible inline in the Console View (annotated on agent messages), or only in the dedicated Eval View? | Console View complexity | Phase 2 start |
| 6 | What MCP servers are currently in use, and should the interface support dynamic MCP server discovery/registration? | Agent config scope | Phase 1 start |

---

## 11. Appendix

### A. Glossary

| Term | Definition |
|---|---|
| **Agent** | An AI entity with a defined role, system prompt, and model configuration that can perform tasks autonomously |
| **C-Suite** | The collection of executive-role agents (CEO, CTO, CFO, COO, CMO, CIO) |
| **Delegation** | When one agent assigns a subtask to another agent |
| **MCP** | Model Context Protocol — standard for connecting AI models to external tools and data sources |
| **Orchestration Pattern** | A reusable workflow defining how multiple agents coordinate to accomplish a goal |
| **Recursive Loop** | An agent iteratively refining its output until a convergence criterion is met |
| **Rubric** | A structured evaluation framework defining quality dimensions and scoring criteria |
| **Convergence** | The point at which successive iterations of a recursive loop produce minimal meaningful change |
| **Topology** | The network structure of agent relationships and communication paths |

### B. Reference Architecture Alignment

This interface maps to the CE Agents monorepo structure:

| Monorepo Directory | Interface Coverage |
|---|---|
| `CE - Agent Builder/` | Agent Detail View, Agent Config, Console View |
| `CE - Multi-Agent Orchestration/` | Topology View, Orchestration View |
| `CE - Evals/` | Eval View, Rubric Editor, Benchmarks |
| `CE - Recursive Loops/` | Recursive Loop Monitor |
| `n8n Workflows/` | Workflow View |
| `Shared/` | Data models, event bus, common utilities |
| `Scripts/` | Backend automation, deployment scripts |
