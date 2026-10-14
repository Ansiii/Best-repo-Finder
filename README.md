# Best Repo Finder: tools that extend Claude Code

A curated list of MCP servers, skills, plugins and CLIs that give Claude Code new
abilities, such as seeing and driving browsers, reverse engineering, pixel art,
web design, 3D and game work, and office files. It favours tools that work
across many projects, not one niche.

> Snapshot: October 2026. Star counts come from third-party directories and
> change often. Check each repo before installing, and read a third-party
> `SKILL.md` before you trust it, since a skill can grant itself tools.

---

## 0. The three kinds of add-on

| Kind | What it is | Install | Best for |
|---|---|---|---|
| **MCP server** | A process that exposes tools Claude can call | `claude mcp add <name> -- <command>` | Driving another program (browser, Ghidra, Blender, Aseprite) |
| **Skill** | A folder with a `SKILL.md` that teaches Claude a workflow | Copy into `~/.claude/skills/` (global) or `.claude/skills/` (per project) | Know-how: design taste, email rules, RE playbooks |
| **CLI + skill** | A normal shell tool plus a skill that teaches Claude to use it | `npm i -g …`, then the tool's skill installer | Token-efficient automation, **parallel sessions** |
| **Plugin** | A bundle of skills, agents, hooks and MCP servers | `/plugin marketplace add <owner/repo>`, then `/plugin install …` | Bundled toolkits |

Rule of thumb: install an MCP server when Claude needs to *operate* something,
and a skill when Claude needs to *know how* to do something well.

---

## 1. Start here: the most useful cross-project tools

| Tool | Type | Why it helps everywhere |
|---|---|---|
| [microsoft/playwright-cli](https://playwright.dev/docs/getting-started-cli) | CLI + skill | Browser control with **named, parallel sessions** (see §2). Uses fewer tokens than the MCP. |
| [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) | MCP | Console, network, performance traces and screenshots. Used to *debug* web pages. |
| [morluto/rea](https://github.com/morluto/rea) | MCP + skill | "Reverse Engineer Anything": binaries, Electron/JS, .NET, APKs, firmware, pcaps, websites |
| [upstash/context7](https://github.com/upstash/context7) | MCP | Pulls current library docs into context, so Claude makes fewer errors from outdated APIs |
| [anthropics/skills](https://github.com/anthropics/skills) | Skills/plugins | Official skills: `frontend-design`, `xlsx`, `docx`, `pptx`, `pdf`, `skill-creator`, … |
| [obra/superpowers](https://github.com/obra/superpowers) | Plugin | Engineering discipline: brainstorm, plan, TDD, debugging and review workflows |
| [firecrawl/firecrawl-mcp-server](https://github.com/firecrawl/firecrawl-mcp-server) or [unclecode/crawl4ai](https://github.com/unclecode/crawl4ai) | MCP / lib | Turns websites into LLM-ready markdown or JSON. Crawl4AI is free and self-hosted. |

```bash
# Browser (parallel-capable)
npm i -g @playwright/cli && playwright-cli install --skills -g
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest --isolated
# Reverse engineering
npx rea-agents setup
# Live docs
claude mcp add context7 -- npx -y @upstash/context7-mcp
# Official skills + Superpowers (inside Claude Code)
/plugin marketplace add anthropics/skills
/plugin marketplace add obra/superpowers-marketplace
```

---

## 2. Browser automation: fixing the "one browser at a time" limit

**Why it happens.** `@playwright/mcp` uses one persistent browser profile by
default. A second Claude session, or a second project, that tries to open the
same profile fails with *"profile already in use"*. You then have to close the
first browser. See [microsoft/playwright-mcp#1294](https://github.com/microsoft/playwright-mcp/issues/1294).

### Option A: keep Playwright MCP and isolate it (quick fix)

```bash
# Every session gets its own in-memory profile, so no lock conflict
claude mcp add playwright -- npx -y @playwright/mcp@latest --isolated
# If you need logins, seed each run from a saved state:
#   ... --isolated --storage-state=./auth.json
# Or give each project its own profile with a project-scoped server:
claude mcp add -s project playwright -- npx -y @playwright/mcp@latest --user-data-dir=./.pw-profile
```

### Option B: switch to a CLI with named sessions (recommended)

| Tool | Parallel model | Notes |
|---|---|---|
| **[Playwright CLI](https://playwright.dev/docs/getting-started-cli)** (Microsoft) | `playwright-cli -s=<name> …`. Any number of named browsers at once, and `playwright-cli list` shows them. | Set `PLAYWRIGHT_CLI_SESSION=projA claude` so each Claude instance gets its own default. `--persistent` keeps the profile on disk. Uses fewer tokens than the MCP. |
| **[vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser)** | `agent-browser --session <name> …`. Sessions are isolated, each with its own auth state. | Rust CLI built for agents and ships an official Claude Code skill. Its compact snapshot refs keep token use low. |
| **[browser-use/browser-use](https://github.com/browser-use/browser-use)** | Local, real-Chrome-profile or cloud modes, with cloud parallelism | Good for long autonomous tasks on sites where you are logged in |

### Option C: many browsers at scale (cloud)

| Tool | Notes |
|---|---|
| [browserbase/mcp-server-browserbase](https://github.com/browserbase/mcp-server-browserbase) (Stagehand) | Hosted browsers, so dozens or hundreds can run at once. Billed per browser-minute. Actions are written in natural language. |
| [browserbase/stagehand](https://github.com/browserbase/stagehand) | The open-source framework under it. Runs locally or on Browserbase. |

### Community wrappers for Playwright MCP (unofficial; check before use)

- `browser-pool-mcp`: runs one Playwright MCP per Claude session on separate ports and cleans up idle ones.
- `playwright-mcp-parallel` / `playwright-parallel-mcp`: N parallel instances, optionally cloning cookies from your Chrome.

**Which to pick.** Use Playwright CLI or agent-browser for everyday multi-project work. Use Chrome DevTools MCP
when a page is broken and you need console, network or performance data. Use Browserbase when you need many browsers at once.

---

## 3. Reverse engineering (apps, binaries, APKs)

Use these only on software you own or are authorized to analyze.

| Tool | Target | Notes |
|---|---|---|
| [morluto/rea](https://github.com/morluto/rea) | Everything | One MCP that routes work to Ghidra, IDA or Hopper when they are installed. JS/.NET analysis needs none of them. |
| [LaurieWired/GhidraMCP](https://github.com/LaurieWired/GhidraMCP) | Native binaries | Free (Ghidra). Decompile, rename, and follow xrefs. |
| [jtang613/GhidrAssistMCP](https://github.com/jtang613/GhidrAssistMCP) | Native binaries | About 31 tools. Needs Ghidra 11.4 or newer. |
| [mrexodia/ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp) | Native binaries | For IDA Pro users |
| [radareorg/radare2-mcp](https://github.com/radareorg/radare2-mcp) (r2mcp) | Native binaries | Lightweight and scriptable |
| [zinja-coder/jadx-ai-mcp](https://github.com/zinja-coder/jadx-ai-mcp) + [apktool-mcp-server](https://github.com/zinja-coder/apktool-mcp-server) | Android APKs | Static analysis: decompile and read smali and resources |
| `frida-mcp` (PyPI) | Running apps | Dynamic analysis: attach, hook, inject. `claude mcp add frida frida-mcp` |
| Chrome DevTools MCP | Web apps | Network and JS inspection of live sites |

A common pairing is **JADX/Ghidra for static** analysis plus **Frida for dynamic** analysis, with REA coordinating.

---

## 4. Pixel art, sprites and images

| Tool | Type | Notes |
|---|---|---|
| [willibrandon/pixel-mcp](https://github.com/willibrandon/pixel-mcp) | MCP (Go) | Most complete Aseprite MCP: layers, palette-aware drawing, animation, PNG/GIF/spritesheet export. Needs Aseprite 1.3+. |
| [diivi/aseprite-mcp](https://github.com/diivi/aseprite-mcp) | MCP (Python) | About 104 tools: shading ramps, dithering, palette presets, and a raw Lua escape hatch |
| [ext-sakamoro/AsepriteMCP](https://github.com/ext-sakamoro/AsepriteMCP) | MCP (Python) | Preset palettes (GameBoy, NES, PICO-8) and batch processing |
| Nano Banana / Gemini image MCPs (e.g. [zhongweili/nanobanana-mcp-server](https://github.com/zhongweili/nanobanana-mcp-server)) | MCP | AI image generation and editing with reference images. Good for concept art you then clean up as pixel art in Aseprite. |
| Canva MCP (connector) | MCP | Layouts, social graphics, background removal |

Tip: if you don't own Aseprite, Claude can draw pixel art with Python and Pillow on an
explicit grid and palette. A skill that defines palette, grid size and a "no anti-aliasing"
rule gets consistent results.

---

## 5. Website and UI design

| Tool | Type | Notes |
|---|---|---|
| `frontend-design` ([anthropics/skills](https://github.com/anthropics/skills)) | Skill | Steers Claude away from generic template-looking UI |
| [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | Skill | Design system generator: styles, palettes, font pairings, accessibility checks, anti-patterns |
| shadcn MCP (`npx shadcn@latest mcp`) | MCP | Browse and install shadcn/ui components by name |
| [21st-dev/magic-mcp](https://github.com/21st-dev/magic-mcp) | MCP | Generates components from text. Needs an API key. |
| Magic UI MCP | MCP | Animated, high-impact components |
| Figma Dev Mode MCP (official) | MCP | Reads real Figma frames, tokens and variables |
| Playwright CLI / Chrome DevTools MCP | — | **Closes the loop**: Claude screenshots its own page, sees the result and iterates |

---

## 6. Email, Excel and documents

| Need | Tool |
|---|---|
| HTML email | `react-email` skill ([resend/react-email](https://github.com/resend/react-email)) plus an `email-best-practices` skill, or an MJML skill (`email-html-mjml`). Render, then screenshot with Playwright CLI to check the layout. |
| Excel review/creation | Official `xlsx` skill (anthropics/skills). [haris-musa/excel-mcp-server](https://github.com/haris-musa/excel-mcp-server) works without Excel installed. |
| Word / PowerPoint / PDF | Official `docx`, `pptx`, `pdf` skills |

---

## 7. 3D and game development

| Tool | Notes |
|---|---|
| [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp) | About 29k stars. Claude builds and edits Blender scenes. `uvx blender-mcp` (may be renamed `mcp-for-blender`). |
| Official Blender connector | Blender 5.1+. Focuses on docs lookup and scene analysis. |
| [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp) | Controls the Unity editor: scenes, scripts, assets |
| [Coding-Solo/godot-mcp](https://github.com/Coding-Solo/godot-mcp) | Runs and debugs Godot projects |

A pixel-art game pipeline: **pixel-mcp** (sprites) → **godot-mcp** (game) → **Playwright CLI** (test the web export).

---

## 8. Where to find more

- [anthropics/skills](https://github.com/anthropics/skills): official skills
- [skills.sh](https://skills.sh) and [claudemarketplaces.com](https://claudemarketplaces.com): skill and plugin directories
- [glama.ai/mcp/servers](https://glama.ai/mcp/servers), [mcp.so](https://mcp.so) and [mcpservers.org](https://mcpservers.org): MCP directories
- [mcp.so/tags/reverse-engineering](https://mcp.so/tags/reverse-engineering): RE-specific MCPs

## Suggested starter stack

1. **Playwright CLI** (parallel browsers) + **Chrome DevTools MCP** (debugging)
2. **REA** (reverse engineering hub)
3. **Context7** (current docs)
4. **anthropics/skills**: `frontend-design`, `xlsx`, `docx`, `pdf`, `skill-creator`
5. **Superpowers** (workflow discipline)
6. Domain packs as needed: **pixel-mcp**, **blender-mcp**, **godot-mcp/unity-mcp**, **ui-ux-pro-max**, **jadx + frida**

Use `skill-creator` to turn your own repeated workflows (for example "make a 32×32 NES-palette sprite
sheet") into reusable skills.
