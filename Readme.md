# 🦍 gorilla-mode

> A gamified, pixel-art terminal shell UI built for the Inner Animal Media platform.
> Think: if Donkey Kong and a Cloudflare dev dashboard had a baby and raised it on chiptune.

-----

## Vision

`gorilla-mode` is not a terminal emulator. It is a **game-feel UI layer** that sits on top of a real PTY shell and transforms the developer experience into something with personality, presence, and power.

The gorilla is not a mascot. He is the interface.

**Target experience:** Launch the terminal → gorilla walks in → you feel like you’re booting up a game → you do real work inside it without ever feeling like you’re using a boring dev tool.

**Long-term:** `gorilla-mode` ships as a standalone PWA and optionally as an Electron desktop app, while also being embeddable as a tab inside the IAM dashboard (`inneranimal-dashboard` Worker).

-----

## The Stack

|Layer            |Tech                                         |Reason                                           |
|-----------------|---------------------------------------------|-------------------------------------------------|
|UI Shell         |React + Vite                                 |Component model, hot reload, JSX for HUD overlays|
|Terminal Emulator|xterm.js                                     |Industry standard, supports WebGL renderer       |
|PTY Backend      |Existing `iam-pty` (Node/WS)                 |Already built — `github.com/SamPrimeaux/iam-pty` |
|Animations       |CSS sprite sheets + keyframes                |No canvas overhead, theme-swappable              |
|Sound            |Web Audio API (Howler.js)                    |Lightweight, works in browser + PWA              |
|AI Buddy         |IAM MCP endpoint (`mcp.inneranimalmedia.com`)|Agent Sam as the in-terminal assistant           |
|Styling          |CSS custom properties only                   |Theme system requires it — no hardcoded hex      |
|Deploy           |Cloudflare Pages                             |Zero config, instant preview URLs                |
|State            |Zustand                                      |Lightweight, no Redux overhead                   |

-----

## Architecture

```
gorilla-mode/
├── src/
│   ├── core/
│   │   ├── TerminalEngine.jsx      # xterm.js wrapper + PTY WebSocket bridge
│   │   ├── CommandParser.js        # Intercepts / commands before shell
│   │   └── SessionManager.js      # Connects to iam-pty session API
│   │
│   ├── hud/
│   │   ├── HUDLayer.jsx            # Floating overlay container (z-index above terminal)
│   │   ├── QuestLog.jsx            # Live todo/step tracker (replaces plain checklist)
│   │   ├── StatusBar.jsx           # Health bar, XP, deploy streak, error count
│   │   └── ToolUsePanel.jsx        # "About to call X tool — proceed?" gate UI
│   │
│   ├── gorilla/
│   │   ├── GorillaLaunchScreen.jsx # Boot animation, numbered menu, theme-aware BG
│   │   ├── GorillaSpriteController.jsx # State machine: idle/thinking/success/error
│   │   ├── sprites/                # PNG sprite sheets (idle, think, chest-pound, facepalm)
│   │   └── sounds/                 # Chiptune SFX: boot, success, error, deploy, coin
│   │
│   ├── buddy/
│   │   ├── BuddyPanel.jsx          # AI Assist slide-up panel (context-aware)
│   │   ├── BuddyQuickActions.jsx   # "Explain error" / "Show fix" / "Run fix" pills
│   │   └── buddyClient.js          # Calls IAM MCP endpoint for Agent Sam responses
│   │
│   ├── themes/
│   │   ├── jungle-night.css        # Default dark — current IAM teal/navy
│   │   ├── jungle-day.css          # Warm amber/green
│   │   └── lava.css                # Deep red — for when prod is on fire
│   │
│   └── App.jsx                     # Root: LaunchScreen → TerminalEngine + HUD
│
├── public/
│   └── pixel-assets/               # Gorilla sprites, background tiles, fire animations
│
├── wrangler.toml                   # Cloudflare Pages deploy config
└── README.md
```

-----

## Feature Phases

### Phase 0 — Prototype (Standalone React app, no PTY)

**Goal:** Prove the visual/game feel before connecting real shell.

- [ ] Gorilla launch screen with pixel art background
- [ ] Animated gorilla idle sprite (CSS sprite sheet)
- [ ] Numbered menu (1. Workspace, 2. Agent, 3. Tools, 4. Theme, 5. Diag)
- [ ] Theme switcher (jungle-night / jungle-day / lava)
- [ ] Mock terminal output (static scroll, no real PTY)
- [ ] Boot sound + menu select SFX
- [ ] Deploy to Cloudflare Pages for preview

**Exit criteria:** Someone sees it and says “what the hell is this, I want it.”

-----

### Phase 1 — Real Shell (PTY Connected)

**Goal:** Replace mock terminal with live `iam-pty` WebSocket connection.

- [ ] xterm.js terminal renders inside the gorilla shell
- [ ] WebSocket bridge to `iam-pty` backend
- [ ] Gorilla reacts to shell state (idle while waiting, think-scratch while running)
- [ ] Session resume (reconnect on tab switch/refresh)
- [ ] `/` command interceptor hooked in (pre-shell parsing layer)

-----

### Phase 2 — HUD Layer

**Goal:** Game-feel information layer floating above the terminal.

- [ ] **Quest Log** — Agent Sam task steps render as live checklist (updates via MCP)
- [ ] **Status Bar** — deploy streak, error count as health bar, workspace name
- [ ] **Tool Use Gate** — before any destructive MCP tool fires, show “PROCEED? [1] YES [2] NO [esc] CANCEL” with countdown
- [ ] **XP system** — deploys, benchmark passes, tokens saved all award points (stored in D1)
- [ ] Gorilla chest-pound animation on deploy success
- [ ] Gorilla facepalm on error/failed benchmark

-----

### Phase 3 — Buddy System

**Goal:** Agent Sam lives inside the terminal as `/buddy`.

- [ ] `/ buddy` or `Ctrl+A` triggers BuddyPanel slide-up
- [ ] Panel has session context (last N lines of terminal output)
- [ ] Quick action pills: Explain error / How do I fix this / Show fix command / Run it
- [ ] Buddy response streams in real-time (SSE from IAM MCP endpoint)
- [ ] `/deploy`, `/status`, `/diagnostics`, `/theme [name]` slash commands
- [ ] Gorilla “typing” animation while buddy is thinking

-----

### Phase 4 — IAM Dashboard Integration

**Goal:** gorilla-mode runs as a tab inside `inneranimal-dashboard`.

- [ ] Exported as embeddable React component (`<GorillaModeTerminal />`)
- [ ] Receives workspace context via props (workspace ID, user token)
- [ ] `iam_shell_nav` CustomEvent pattern respected (no unmount on tab switch)
- [ ] `?embedded=1` hides gorilla launch screen, drops straight to terminal

-----

### Phase 5 — Standalone App

**Goal:** gorilla-mode ships as its own thing.

- [ ] PWA manifest — installable on mobile and desktop
- [ ] Electron wrapper (optional, for true native terminal feel)
- [ ] Multi-session tab support (each tab = a separate gorilla instance)
- [ ] Onboarding flow for non-IAM users (connect your own PTY or Cloudflare account)
- [ ] Public launch — `gorilla.inneranimalmedia.com`

-----

## The Game Feel Principles

These are non-negotiable design rules, not suggestions:

1. **Every action gets feedback.** No silent operations. Success plays a sound. Errors react visually.
1. **The gorilla is never static.** Idle breathing loop always runs. He acknowledges what’s happening.
1. **Information is game language.** Never show raw error counts — show a health bar. Never show a plain checklist — show a quest log.
1. **Sound is 50% of the feel.** No sound = no game feel. Every phase ships with SFX.
1. **Themes are moods, not skins.** `lava` mode should feel genuinely tense. `jungle-day` should feel light and fast.
1. **CSS variables only.** Zero hardcoded hex. Themes must be swappable at runtime.
1. **No placeholder UI ships.** Every component is either real or not there. No gray boxes.

-----

## Connection to IAM Platform

|gorilla-mode resource|IAM platform connection                                        |
|---------------------|---------------------------------------------------------------|
|PTY sessions         |`terminal_sessions` table in `inneranimalmedia-business` D1    |
|AI Buddy calls       |`mcp.inneranimalmedia.com/mcp` (Agent Sam MCP endpoint)        |
|XP / deploy streaks  |New `gorilla_xp` table in D1 (Phase 2)                         |
|Slash command skills |`agentsam_skill` table — `/deploy`, `/status` map to skill rows|
|Theme preference     |`user_settings` table — persisted per user                     |
|Tool use gate        |Hooks into existing `mcp_audit_log` before execution           |
|Quest Log            |Reads from `cicd_pipeline_runs` + `cicd_run_steps` live        |

-----

## Repo Connections

|Repo                                             |Role                                                   |
|-------------------------------------------------|-------------------------------------------------------|
|`SamPrimeaux/iam-pty`                            |PTY backend — WebSocket server gorilla-mode connects to|
|`InnerAnimal/inneranimalmedia-agentsam-dashboard`|IAM dashboard — Phase 4 embed target                   |
|`InnerAnimal/gorilla-mode`                       |This repo — the game shell UI                          |

-----

## Immediate Next Step

Build **Phase 0** as a React artifact first — pure visual prototype, no backend, no PTY.

Deliverable: A shareable Cloudflare Pages URL where the gorilla launch screen runs, the menu is interactive, themes switch, and the mock terminal scrolls fake output with game-feel SFX.

Once that’s greenlit visually → Phase 1 connects the real shell.

-----

*Built by Inner Animal Media. The gorilla was here first.*
