# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FlowPilot là ứng dụng desktop automation sử dụng kiến trúc **monorepo** với hai thành phần chính:

```
FlowPilot/
├── FlowPilotUI/          # Electron + Vue 3 + TypeScript (đã có code)
└── FlowPilotBE/          # Python + FastAPI (chưa code - todo)
```

## Commands

### FlowPilotUI

```bash
cd FlowPilotUI

# Install dependencies
npm install

# Development (HMR enabled)
npm run dev

# Production build
npm run build

# Platform-specific builds
npm run build:win
npm run build:mac
npm run build:linux

# Code quality
npm run lint         # ESLint
npm run format       # Prettier
npm run typecheck    # TypeScript (tsc --noEmit)

# After any file change — always run all three
npm run format && npm run lint && npm run typecheck
```

## Architecture

### FlowPilotUI — Electron Process Model

```
┌─────────────────────────────────────────────────────────┐
│                    Main Process                         │
│   src/main/                                             │
│   ├── index.ts        # App entry, window creation     │
│   ├── ipc/            # IPC handlers (domain-based)     │
│   │   ├── index.ts    # Registry — register all here   │
│   │   ├── automation.ipc.ts                            │
│   │   ├── file.ipc.ts                                  │
│   │   └── system.ipc.ts                                │
│   └── services/       # Tray, auto-updater             │
└─────────────────────────────────────────────────────────┘
                          │ contextBridge
┌─────────────────────────────────────────────────────────┐
│                  Renderer Process (Vue 3)                │
│   src/renderer/src/                                     │
│   ├── composables/useIpc.ts    # IPC client wrapper     │
│   ├── stores/                  # Pinia stores          │
│   ├── components/ui/            # Fp-prefixed components │
│   ├── views/                   # Page components        │
│   └── router/                  # Vue Router config      │
└─────────────────────────────────────────────────────────┘
```

### IPC Communication

- **Renderer → Main**: `useIpc().call(channel, args)` via `ipcMain.handle`
- **Main → Renderer**: `window.electron.send(channel, data)` via `ipcRenderer.on`
- **Channel naming**: `domain:action-subaction` (e.g., `automation:run-task`)
- **IPC registry**: Handlers registered in `src/main/ipc/index.ts` — MUST be called in `app.whenReady()` (hiện tại chưa được wire, xem bug trong CLAUDE.local.md)

### Shared Types

- `shared/types/` — TypeScript types dùng chung giữa main và renderer
- `shared/schemas/` — JSON schemas định nghĩa IPC payload structures

### Design System

- Component prefix: `Fp` (FpButton, FpInput, FpSelect...)
- Styling: Tailwind CSS v4
- State: Pinia (Composition API style)
- Component style composables: `src/renderer/src/composables/styles/`

## Key Rules

### Vue 3 Components

- Luôn dùng `<script setup lang="ts">`
- Component files: `PascalCase.vue`, root class kebab-case (e.g. `fp-button`)
- Props/emits: typed với `withDefaults` và `defineEmits`

### IPC Handlers

- Validate input TRƯỚC khi xử lý (type check, empty check, path traversal)
- Response shape: `{ success: boolean; data?: T; error?: string }`
- Log security-relevant events

### Security

- `sandbox: true`, `contextIsolation: true`, `nodeIntegration: false`
- App ID: `com.flowpilot.app` (set trong `src/main/index.ts`)
- Không log sensitive data (passwords, tokens, file contents)

## FlowPilotBE — Backend (Python/FastAPI)

Chưa được khởi tạo. Khi bắt đầu code backend:

- Sử dụng FastAPI
- CORS configuration để cho phép Electron app gọi API
- Các services xử lý automation logic (screen recording, input simulation, v.v.)

## Sub-project Detailed Rules

Mỗi sub-project có detailed rules riêng tại:

- **FlowPilotUI**: `FlowPilotUI/.claude/rules/` (10 files — code style, Vue conventions, IPC, state, error handling, security, performance, testing, git workflow, project structure)
- **FlowPilotUI**: `FlowPilotUI/CLAUDE.local.md` — local overrides cho máy này

## Command Execution

BẮT BUỘC: Bạn phải luôn thực thi mọi lệnh bằng môi trường Bash. Tuyệt đối KHÔNG ĐƯỢC sử dụng các lệnh của PowerShell (như `Get-ChildItem`, `Select-Object`). Hãy sử dụng trực tiếp cú pháp lệnh chuẩn của Linux/Bash (ví dụ: `find`, `ls`, `grep`, `cat`) cho công cụ thực thi lệnh. Đảm bảo xử lý các đường dẫn file Windows cẩn thận trong dấu ngoặc kép khi sử dụng Bash.

---

<!-- rtk-instructions v2 -->

# RTK (Rust Token Killer) - Token-Optimized Commands

## Golden Rule

**Always prefix commands with `rtk`**. If RTK has a dedicated filter, it uses it. If not, it passes through unchanged. This means RTK is always safe to use.

**Important**: Even in command chains with `&&`, use `rtk`:

```bash
# ❌ Wrong
git add . && git commit -m "msg" && git push

# ✅ Correct
rtk git add . && rtk git commit -m "msg" && rtk git push
```

## RTK Commands by Workflow

### Build & Compile (80-90% savings)

```bash
rtk cargo build         # Cargo build output
rtk cargo check         # Cargo check output
rtk cargo clippy        # Clippy warnings grouped by file (80%)
rtk tsc                 # TypeScript errors grouped by file/code (83%)
rtk lint                # ESLint/Biome violations grouped (84%)
rtk prettier --check    # Files needing format only (70%)
rtk next build          # Next.js build with route metrics (87%)
```

### Test (90-99% savings)

```bash
rtk cargo test          # Cargo test failures only (90%)
rtk vitest run          # Vitest failures only (99.5%)
rtk playwright test     # Playwright failures only (94%)
rtk test <cmd>          # Generic test wrapper - failures only
```

### Git (59-80% savings)

```bash
rtk git status          # Compact status
rtk git log             # Compact log (works with all git flags)
rtk git diff            # Compact diff (80%)
rtk git show            # Compact show (80%)
rtk git add             # Ultra-compact confirmations (59%)
rtk git commit          # Ultra-compact confirmations (59%)
rtk git push            # Ultra-compact confirmations
rtk git pull            # Ultra-compact confirmations
rtk git branch          # Compact branch list
rtk git fetch           # Compact fetch
rtk git stash           # Compact stash
rtk git worktree        # Compact worktree
```

Note: Git passthrough works for ALL subcommands, even those not explicitly listed.

### GitHub (26-87% savings)

```bash
rtk gh pr view <num>    # Compact PR view (87%)
rtk gh pr checks        # Compact PR checks (79%)
rtk gh run list         # Compact workflow runs (82%)
rtk gh issue list       # Compact issue list (80%)
rtk gh api              # Compact API responses (26%)
```

### JavaScript/TypeScript Tooling (70-90% savings)

```bash
rtk pnpm list           # Compact dependency tree (70%)
rtk pnpm outdated       # Compact outdated packages (80%)
rtk pnpm install        # Compact install output (90%)
rtk npm run <script>    # Compact npm script output
rtk npx <cmd>           # Compact npx command output
rtk prisma              # Prisma without ASCII art (88%)
```

### Files & Search (60-75% savings)

```bash
rtk ls <path>           # Tree format, compact (65%)
rtk read <file>         # Code reading with filtering (60%)
rtk grep <pattern>      # Search grouped by file (75%)
rtk find <pattern>      # Find grouped by directory (70%)
```

### Analysis & Debug (70-90% savings)

```bash
rtk err <cmd>           # Filter errors only from any command
rtk log <file>          # Deduplicated logs with counts
rtk json <file>         # JSON structure without values
rtk deps                # Dependency overview
rtk env                 # Environment variables compact
rtk summary <cmd>       # Smart summary of command output
rtk diff                # Ultra-compact diffs
```

### Infrastructure (85% savings)

```bash
rtk docker ps           # Compact container list
rtk docker images       # Compact image list
rtk docker logs <c>     # Deduplicated logs
rtk kubectl get         # Compact resource list
rtk kubectl logs        # Deduplicated pod logs
```

### Network (65-70% savings)

```bash
rtk curl <url>          # Compact HTTP responses (70%)
rtk wget <url>          # Compact download output (65%)
```

### Meta Commands

```bash
rtk gain                # View token savings statistics
rtk gain --history      # View command history with savings
rtk discover            # Analyze Claude Code sessions for missed RTK usage
rtk proxy <cmd>         # Run command without filtering (for debugging)
rtk init                # Add RTK instructions to CLAUDE.md
rtk init --global       # Add RTK to ~/.claude/CLAUDE.md
```

## Token Savings Overview

| Category         | Commands                       | Typical Savings |
| ---------------- | ------------------------------ | --------------- |
| Tests            | vitest, playwright, cargo test | 90-99%          |
| Build            | next, tsc, lint, prettier      | 70-87%          |
| Git              | status, log, diff, add, commit | 59-80%          |
| GitHub           | gh pr, gh run, gh issue        | 26-87%          |
| Package Managers | pnpm, npm, npx                 | 70-90%          |
| Files            | ls, read, grep, find           | 60-75%          |
| Infrastructure   | docker, kubectl                | 85%             |
| Network          | curl, wget                     | 65-70%          |

Overall average: **60-90% token reduction** on common development operations.

<!-- /rtk-instructions -->

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **FlowPilot** (62 symbols, 60 relationships, 0 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/FlowPilot/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring

- **Renaming**: MUST use `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` first. Review the preview — graph edits are safe, text_search edits need manual review. Then run with `dry_run: false`.
- **Extracting/Splitting**: MUST run `gitnexus_context({name: "target"})` to see all incoming/outgoing refs, then `gitnexus_impact({target: "target", direction: "upstream"})` to find all external callers before moving code.
- After any refactor: run `gitnexus_detect_changes({scope: "all"})` to verify only expected files changed.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Tools Quick Reference

| Tool | When to use | Command |
|------|-------------|---------|
| `query` | Find code by concept | `gitnexus_query({query: "auth validation"})` |
| `context` | 360-degree view of one symbol | `gitnexus_context({name: "validateUser"})` |
| `impact` | Blast radius before editing | `gitnexus_impact({target: "X", direction: "upstream"})` |
| `detect_changes` | Pre-commit scope check | `gitnexus_detect_changes({scope: "staged"})` |
| `rename` | Safe multi-file rename | `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` |
| `cypher` | Custom graph queries | `gitnexus_cypher({query: "MATCH ..."})` |

## Impact Risk Levels

| Depth | Meaning | Action |
|-------|---------|--------|
| d=1 | WILL BREAK — direct callers/importers | MUST update these |
| d=2 | LIKELY AFFECTED — indirect deps | Should test |
| d=3 | MAY NEED TESTING — transitive | Test if critical path |

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/FlowPilot/context` | Codebase overview, check index freshness |
| `gitnexus://repo/FlowPilot/clusters` | All functional areas |
| `gitnexus://repo/FlowPilot/processes` | All execution flows |
| `gitnexus://repo/FlowPilot/process/{name}` | Step-by-step execution trace |

## Self-Check Before Finishing

Before completing any code modification task, verify:
1. `gitnexus_impact` was run for all modified symbols
2. No HIGH/CRITICAL risk warnings were ignored
3. `gitnexus_detect_changes()` confirms changes match expected scope
4. All d=1 (WILL BREAK) dependents were updated

## Keeping the Index Fresh

After committing code changes, the GitNexus index becomes stale. Re-run analyze to update it:

```bash
npx gitnexus analyze
```

If the index previously included embeddings, preserve them by adding `--embeddings`:

```bash
npx gitnexus analyze --embeddings
```

To check whether embeddings exist, inspect `.gitnexus/meta.json` — the `stats.embeddings` field shows the count (0 means no embeddings). **Running analyze without `--embeddings` will delete any previously generated embeddings.**

> Claude Code users: A PostToolUse hook handles this automatically after `git commit` and `git merge`.

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->
