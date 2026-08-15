# Munder Difflin — Comprehensive Developer Guide & Step-by-Step Tutorial

Welcome to the definitive guide for **Munder Difflin**. Whether you're an engineer looking to understand how the system works under the hood, a developer wanting to build on top of it, or a user who wants to maximize their workflow with AI agents, this guide covers everything you need to know.

This document serves as an exhaustive, standalone manual designed for a developer audience with zero prior knowledge of the project.

---

## Table of Contents

1. [What is Munder Difflin?](#1-what-is-munder-difflin)
2. [System Architecture Overview](#2-system-architecture-overview)
3. [The Hive: Autonomous Multi-Agent Layer](#3-the-hive-autonomous-multi-agent-layer)
4. [The Design System: Pixi.js & Retro Aesthetics](#4-the-design-system-pixijs--retro-aesthetics)
5. [Step-by-Step Setup & Installation Tutorial](#5-step-by-step-setup--installation-tutorial)
6. [Step-by-Step Usage Tutorial](#6-step-by-step-usage-tutorial)
7. [Codebase Deep Dive & Structure](#7-codebase-deep-dive--structure)
8. [Developer Guide: Extending & Contributing](#8-developer-guide-extending--contributing)
9. [Telemetry & Observability](#9-telemetry--observability)

---

## 1. What is Munder Difflin?

**Munder Difflin** is an open-source, local-first desktop application that transforms terminal-based AI coding agents (like Claude Code, Antigravity, OpenAI Codex, etc.) into a collaborating, self-coordinating team.

Instead of running separate, isolated terminal windows, Munder Difflin wraps these real CLI processes in pseudo-terminals (`node-pty`) and visualizes them on a 2D Pixi.js office floor. Every agent is represented as a character (avatar) that physically "walks" to different stations depending on what tool they are using (e.g., walking to a file shelf when reading a file).

More importantly, it provides a shared memory and communication layer (the **Hive**) and introduces a **GOD Agent** (an orchestrator) that delegates tasks, answers questions, and escalates critical decisions to you.

### Core Features
- **Every terminal is an agent:** Wraps actual CLI processes.
- **Visual orchestration:** Watch agents work in a 2D space.
- **The Hive:** A shared markdown-first memory, mailbox system, and task board running over a single-committer Git repository.
- **God-Mode Autonomy:** A built-in orchestrator (Michael) handles traffic and routing so you only deal with escalations.
- **Built-in IDE & Command Center:** Inspect logs, edit files, manage git state, and view token/cost budgets.

---

## 2. System Architecture Overview

Munder Difflin is built on a split-plane architecture: the **Terminal Plane** and the **Event Plane**, mediated by an Electron application.

### The Stack
- **Electron:** Desktop wrapper providing the Node main process and React renderer.
- **React & TypeScript:** UI framework and language.
- **Pixi.js:** Renders the 2D "office floor" and sprite animations.
- **xterm.js:** Renders the terminal stream authentically.
- **node-pty:** Spawns and manages the raw pseudo-terminal processes for the agents.

### The Two Data Planes
1. **Terminal Plane (Raw PTY):**
   The Electron Main Process spawns a real `claude` (or other) process via `node-pty`. It captures `stdout`/`stderr` and streams the raw bytes over an IPC channel (`window.cth`) to the React renderer, where `xterm.js` displays it. User input is streamed back to `stdin`.
2. **Event Plane (Structured State):**
   Agents emit structured events (e.g., `PreToolUse`, `PostToolUse`). The application hooks into these events via a custom Unix Domain Socket (`~/.cth/events.sock`). When an agent uses a tool, a hook payload is POSTed to the app, which then updates the React state to tell Pixi.js to animate the avatar walking to the corresponding station.

---

## 3. The Hive: Autonomous Multi-Agent Layer

The "Hive" is how Munder Difflin turns a room full of independent agent processes into a coordinating team.

### How it Works
The Hive exists entirely as files in a local Git repository managed by the app (located in `<harnessHome>/hive/`). **Only the Electron main process commits to this repo** to prevent `.git/index.lock` collisions.

- **Mailboxes (`inbox/` & `outbox/`):** When Agent A wants to message Agent B, it writes a JSON file to its `outbox/`. The main process (the router) detects this and moves it to Agent B's `inbox/`.
- **The Loop:** When an agent finishes a task, a `Stop` hook fires. The app checks the agent's `inbox/`. If there are unread messages, it intercepts the stop and forces the agent to keep working on the new messages.
- **Memory (`memory.md`):** Each agent has long-term markdown memory. A semantic SQLite FTS index (via an optional MemPalace CLI) allows agents to retrieve memories from past sessions.
- **The GOD Agent (Michael):** A fixed, always-on agent sitting at the CEO desk. It reads outbound requests from other agents. Routine issues it resolves itself or routes to the correct specialist. Only critical actions (destructive operations, large spends) are escalated to the human user.

---

## 4. The Design System: Pixi.js & Retro Aesthetics

Munder Difflin strictly adheres to a chunky, friendly, pixel-snapped aesthetic heavily inspired by *Earthbound*, *Animal Crossing*, and SNES UIs.

- **Visuals:** Uses Pixi.js for a 2D tile-based scene. Grid-aligned movement, 4-directional 8-fps sprites.
- **Components:** Built with a strict CSS system (`tokens.css` and `tokens.ts`). Drop shadows are hard (no blur), borders are 3-layered (SNES style), and text uses pixel fonts (`Press Start 2P`, `Pixelify Sans`, `VT323`).
- **Meaningful Motion:** Animations convey system state. If an agent is walking to the web portal, you instantly know it's fetching a webpage.

---

## 5. Step-by-Step Setup & Installation Tutorial

### Prerequisites
1. **OS:** macOS (primary target), Linux, or Windows.
2. **Node.js:** v18 or higher.
3. **C/C++ Toolchain:** Required to compile `node-pty`.
   - *macOS:* Run `xcode-select --install`
4. **Agent CLI:** You must have at least one supported agent CLI installed globally on your system.
   - E.g., `npm install -g @anthropic-ai/claude-code`

### Installation
1. **Clone the repository:**
   ```bash
   git clone https://github.com/chaitanyagiri/munder-difflin.git
   cd munder-difflin
   ```
2. **Install dependencies:**
   ```bash
   npm install
   ```
   *Note: This runs a `postinstall` script to rebuild `node-pty` against Electron's ABI. If you get architecture mismatch errors, confirm your toolchain and run `npm install` again.*
3. **Start the development server:**
   ```bash
   npm run dev
   ```
   This command starts the Electron app with hot-reloading for UI changes.

---

## 6. Step-by-Step Usage Tutorial

### 6.1 Spawning Your First Agent
1. When the app launches, you will see an onboarding wizard that explains the concept.
2. Click **Add Agent**.
3. Choose your provider (e.g., Claude Code), give the agent a name, and select a character sprite.
4. Munder Difflin will automatically spawn a pseudo-terminal running the CLI in the background.
5. The **GOD Agent (Michael)** will automatically be spawned and placed at the CEO desk to manage the floor.

### 6.2 Managing the Workspace
- **The Command Center:** Located in the UI, this panel lets you view scheduled missions, see active tasks on a Kanban board, and monitor memory usage.
- **Typing to an Agent:** Click on an agent to open their terminal panel. You can type commands directly into their stream.
- **Notifications:** If an agent attempts a destructive action or asks for permission, Michael will catch it and flag it to you as a notification toast. You act as the ultimate Human-in-the-Loop (HITL).

### 6.3 Using the Built-in IDE
Munder Difflin comes with a built-in Monaco Editor. You can view the files the agents are generating, compare Git diffs, and visually track what code is being changed without leaving the application.

---

## 7. Codebase Deep Dive & Structure

If you want to modify the source code, you need to understand the directory layout:

```text
src/
├── main/                 # Electron main process (Node)
│   ├── index.ts          # IPC handlers, window creation
│   ├── pty.ts            # node-pty manager (spawn, write, stream)
│   ├── hive.ts           # On-disk multi-agent layer (mailbox routing)
│   ├── hooks.ts          # Socket server receiving tool events from agents
│   └── ...               # Config, Telemetry, DB, File System bridges
├── preload/              # contextBridge API exposing main methods to renderer
└── renderer/src/         # React application
    ├── App.tsx
    ├── design/           # CSS Tokens, global styles
    ├── components/       # UI Components (PixelPanel, AgentCard, etc.)
    ├── scene/office/     # Pixi.js 2D rendering logic (Floor, Pathfinding)
    └── store/            # Zustand global state
```

### Key Execution Flow
1. User clicks "Add Agent".
2. `renderer` sends IPC `add-agent` to `main/index.ts`.
3. `main/pty.ts` spawns `node-pty`.
4. `main/hooks.ts` intercepts hooks to track tool usage.
5. `renderer/src/store` consumes IPC streams, parsing terminal output to xterm.js and event states to `Pixi.js` via the Zustand store.

---

## 8. Developer Guide: Extending & Contributing

### Adding a New Agent Engine
To add support for a new CLI tool (e.g., a new AI agent CLI):
1. **Hook Shimming:** Ensure the CLI has a way to emit lifecycle events (like PreToolUse). If it doesn't, you must write a wrapper script or shim in `src/main/hooks.ts` to simulate these events.
2. **Terminal Spawning:** Update `src/main/pty.ts` to recognize the command and pass the necessary environment variables.
3. **Capabilities:** Update `hive/registry.json` logic so the system knows what tools the new CLI supports.

### Modifying the UI / Design System
The UI relies strictly on `src/renderer/src/design/tokens.ts` and `tokens.css`.
- **Do not use random hex codes.** Always map colors to the token variables (e.g., `var(--cth-cream-100)`).
- **Do not use CSS transitions that break the pixel grid.** Animations should be frame-based or snapped to integers.

### Adding New Stations or Sprites
- Sprites live in `src/renderer/src/assets/`.
- To add a new station (e.g., a "Coffee Machine"), you must map it in the Pixi.js scene (`src/renderer/src/scene/office/`) and bind it to a specific tool or event state.

### Before Submitting a PR
1. Run `npm run typecheck` to ensure there are no TypeScript errors across Node or Web projects.
2. Run `npm run build` to verify the production build succeeds.
3. Respect the design guidelines outlined in `DESIGN.md`.

---

## 9. Telemetry & Observability

Munder Difflin features a sophisticated telemetry system.
- **Tool Waterfall:** Built-in observability views let you see exactly how long tools took to execute.
- **Budget Tracking:** The system parses agent transcripts to attribute exact token costs to specific actions, logging them to a durable SQLite ledger.
- **Circuit Breaker:** If an agent gets stuck in a loop, the circuit breaker automatically steers, constrains, or stops the process to prevent runaway API costs.

*Note on Privacy:* Official builds send anonymous usage events (app open, agent spawn). These do not contain file paths, code, or prompts. This can be opted out via settings or by building from source.

---

*This guide was generated to help developers immediately grasp the entirety of the Munder Difflin ecosystem. Dive into the code, spawn some agents, and start building!*