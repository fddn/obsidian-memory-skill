---
name: obsidian-memory
description: >
  Set up Obsidian as Claude Code's long-term memory vault. Use when the user
  says "install Obsidian as my memory", "set up my second brain", "configure
  Obsidian for Claude", "make Claude remember things in Obsidian", or wants a
  structured knowledge vault that Claude reads across sessions. Creates the
  full folder architecture, navigation files, templates, session protocols,
  and CLAUDE.md integration so Claude treats the vault as durable memory.
---

# Obsidian Memory Vault — Setup Skill

Sets up an Obsidian vault as **long-term memory** for Claude Code. After running this skill the user will have:

- A structured vault with a proper hierarchy (not a pile of notes)
- Navigation files (INDEX + MOC) that Claude reads first
- Templates for people, projects, clients, daily reports
- A protocol for when to save and when to read
- `CLAUDE.md` with the vault wired into every session
- (Optional) An MCP Obsidian server for direct access
- (Optional) Hooks for automatic entries

**This isn't just a folder of notes — it's an operating system for memory.** The vault becomes the source of truth about people, projects, decisions, lessons.

---

## How to use this skill

When the user asks "set up Obsidian as memory", follow the steps below. Don't skip Step 0: if you don't collect the basic parameters, the skill will create the vault in the wrong place.

### Step 0 — Collect parameters

Ask the user (in one message, not one at a time):

1. **Vault path** — where to create it? If there is an existing Obsidian vault, use it. If not, suggest a path (for example `~/ObsidianVault` on Mac/Linux, `C:\ObsidianVault` on Windows).
2. **Content language** — what language to write the templates and navigation in? (EN / UA / RU / ES / other)
3. **Main use** — personal memory / client work (CRM) / technical projects / all of the above?
4. **MCP Obsidian** — install an MCP server for direct reading/writing through the Obsidian API? (Y/N — if the user doesn't know what this is, default to **no**; the vault will be read through the file system, which is enough)
5. **Automation hooks** — set up session-start and session-stop hooks to load context automatically? (Y/N — default **no**; it's more complex and not everyone needs it)

If the user didn't answer some of these, choose sensible defaults and tell them which ones.

### Step 1 — Create the folder structure

Adapt it to the chosen use case (from Step 0.3):

**Minimal (personal memory):**
```
vault/
├── 00-INDEX.md
├── MANIFEST.md
├── Memory/
│   ├── Active Tasks.md
│   ├── Session Log.md
│   ├── Lessons Learned.md
│   └── Daily Reports/
├── People/
├── Projects/
├── Rules/
└── Templates/
```

**Full (work + CRM + projects):**
```
vault/
├── 00-INDEX.md                     ← Entry point, read FIRST
├── MANIFEST.md                     ← Full vault index
├── Memory/                         ← Core of the memory
│   ├── Active Tasks.md             — unfinished tasks
│   ├── Session Log.md              — what was done in each session
│   ├── Lessons Learned.md          — mistakes + conclusions
│   ├── Skills Registry.md          — when to call which skill/agent
│   └── Daily Reports/              — daily reports
├── Rules/                          ← Operating rules
│   ├── Rules MOC.md                — rules index
│   ├── Session Protocol.md         — how to start/end a session
│   ├── Language Rules.md           — language of replies and content
│   └── Content Rules.md            — rules for creating content
├── People/                         ← CRM
│   ├── Clients MOC.md              — CRM entry point
│   ├── Clients/                    — active clients (🟡🟢🔴)
│   ├── Partners/                   — partners
│   └── Team/                       — team
├── Projects/                       ← Technical / work projects
│   ├── Projects MOC.md             — projects index
│   └── Dashboard.md                — live status of all projects
├── Business/                       ← (Optional) products, prices, offers
│   └── Business MOC.md
├── Templates/                      ← Templates for the Obsidian Template plugin
│   ├── Person.md
│   ├── Client.md
│   ├── Project.md
│   └── Daily Report.md
└── Credentials/                    ← (Optional) placeholders for tokens
```

Create all folders with `mkdir -p` (Unix) or `New-Item -ItemType Directory` (PowerShell). **Obsidian hides folders without a `.gitkeep` — not a problem, the content shows up once you create files.**

### Step 2 — Create the navigation files

This is **the most important step**. Without these files Claude won't know what to read first.

Create these files from the template content in `templates/`:

1. **`00-INDEX.md`** — a copy of [templates/00-INDEX.md](templates/00-INDEX.md), adapted to the chosen structure. This is the entry point.
2. **`MANIFEST.md`** — a full index of every vault file with a short description of each (filled in automatically, then updated by hand).
3. **`Memory/Active Tasks.md`** — from [templates/Active-Tasks.md](templates/Active-Tasks.md). An empty template with Hot/Sales/Product/Technical sections.
4. **`Memory/Session Log.md`** — from [templates/Session-Log.md](templates/Session-Log.md). An empty session log.
5. **`Memory/Lessons Learned.md`** — from [templates/Lessons-Learned.md](templates/Lessons-Learned.md). An empty list of lessons.
6. **MOC files** (Maps of Content) — an index for each category: `Clients MOC.md`, `Projects MOC.md`, `Rules MOC.md`, `Business MOC.md`. Taken from [templates/MOC-template.md](templates/MOC-template.md).

Important: **translate the headings and descriptions into the language from Step 0.2**, but don't change the structure.

### Step 3 — Copy the templates

Copy into `vault/Templates/`:

- `Person.md` — a general-purpose contact template
- `Client.md` — a CRM client card with statuses
- `Project.md` — a project card with architectural decisions
- `Daily Report.md` — a daily report

All of them come from this skill's `templates/` folder.

Tell the user how to enable the Obsidian Template plugin: Settings → Core plugins → Templates → enable → set template folder location = `Templates`.

### Step 4 — Create the `CLAUDE.md` integration

This is critical. `CLAUDE.md` holds the project instructions that Claude Code reads at the start of every session in that folder. It must point to the vault.

**Where to create it:**

- If the vault **is** the user's working folder → `CLAUDE.md` in the vault root
- If the vault is **separate** from the working folders → `CLAUDE.md` in every working folder that needs access to the memory, OR the global `~/.claude/CLAUDE.md`

Take [examples/CLAUDE-md-example.md](examples/CLAUDE-md-example.md), fill in the user's actual vault path, adapt the language.

Minimum content:
```markdown
## Memory — Obsidian vault

- **Path:** `<full path to the vault>`
- **Vault map:** `00-INDEX.md` ← read FIRST at session start
- **Unfinished tasks:** `Memory/Active Tasks.md`
- **Lessons from mistakes:** `Memory/Lessons Learned.md`

## Memory protocol

### When to READ (session start):
1. `00-INDEX.md` → overview of the structure
2. `Memory/Active Tasks.md` → what's in progress
3. The latest Daily Report (if you need "what happened yesterday" context)

### When to SAVE (without asking):
- New person → `People/{Name}.md` + update the MOC
- New business decision → `Business/{Name}.md`
- New rule → `Rules/{Name}.md`
- Technical decision → `Projects/{Name}.md`
- Mistake + conclusion → `Memory/Lessons Learned.md`
- End of session → update `Memory/Active Tasks.md` + `Memory/Session Log.md`
```

### Step 5 (optional) — MCP Obsidian server

If the user chose Y in Step 0.4, install the MCP server. There are two popular ones:

**Option A — `obsidian-mcp` (recommended, simpler):**
```bash
claude mcp add obsidian npx -y obsidian-mcp <path_to_vault>
```

**Option B — `mcp-obsidian` (through the REST API plugin):**
1. In Obsidian: Settings → Community plugins → search "Local REST API" → install + enable
2. Generate an API key in the plugin settings
3. `claude mcp add obsidian -e OBSIDIAN_API_KEY=<key> -- npx -y mcp-obsidian`

Check the installation: `claude mcp list` → `obsidian` should show as connected.

MCP pros: direct API requests to the vault (faster than `cat` for large operations), understands wikilinks, tag search. Cons: one more process, a Node.js dependency.

**It also works without MCP.** Claude reads the vault with Read/Grep/Glob — enough for 90% of tasks.

### Step 6 (optional) — Automation hooks

If the user chose Y in Step 0.5, set up a session-start hook that automatically adds to the context:
- The current Active Tasks
- The latest Daily Report
- The last 5 lessons

Detailed instructions: [references/hooks-setup.md](references/hooks-setup.md).

**Don't push hooks on the user.** They help with heavy use but add complexity. Better to offer them later, once the user is comfortable with the basic vault.

### Step 7 — Final check

1. Check that all folders exist: `ls -la <vault>`
2. Check that `00-INDEX.md` opens and has correct links
3. If MCP is installed, check `claude mcp list`
4. Show the user **what to do next**:
   - Open the vault in Obsidian (File → Open vault → choose the folder)
   - Enable Community plugins if needed
   - Start a Claude Code session in the folder with `CLAUDE.md` — Claude should read the vault automatically

### Step 8 — Wrap up

Print a short summary:
- ✅ Created: N folders, M files
- ✅ Vault path: `<path>`
- ✅ CLAUDE.md in: `<path>`
- ⚠️ What to do by hand (if anything): open the vault in Obsidian, enable plugins
- 📖 Next steps: start filling in People/, Projects/

---

## Architecture principles (why it's built this way)

### 3 tiers of memory

| Tier | System | What it stores | Lifecycle |
|--------|---------|-------------|---------------|
| Fast | Claude Memory (`~/.claude/.../memory/`) | Behavioral corrections, bridge | Between sessions |
| Long-term | **Obsidian vault** | People, rules, projects, lessons | Forever |
| Operational | Notion / Todoist / Linear | Tasks, statuses, content | Working |

Obsidian is specifically the **long-term** tier. It holds things that should survive dozens of sessions: who the client is, what was decided, which mistakes not to repeat.

### Why MOC, not tags

A MOC (Map of Content) is an explicit index file. Tags are implicit filters.

Claude finds its way faster through explicit links: `[[Clients MOC]]` → `[[John Doe]]`, than through tags like `#client`. Tags are added on top, not instead.

### Why `00-INDEX.md` and not `README.md`

The `00-` prefix moves the file to the top of the sort order. Obsidian's sidebar always shows it first. Claude also always reads it as the first entry point.

### Why `People/Clients/` and not `Clients/`

Everything about people is in one place. CRM, partners, team — all under `People/`. That way Claude doesn't get confused when it needs to find a contact, and there's no duplication (a person who is both a client and a partner — one file).

### Why Active Tasks isn't in Notion

**Active Tasks is a short, mutable "what's in progress right now" list** for Claude. It's read at the start of every session. Putting it in Notion means an extra API call and extra syncing. In the vault it's `cat Memory/Active\ Tasks.md` and done.

Notion stays for full project management with a team. Active Tasks is for the AI.

---

## Common mistakes

**Mistake 1: Flat structure without MOCs**
> The user creates a vault and drops 200 files into the root. Claude can't find what to read.

Fix: always create MOC files. Even with only 2 clients today, `Clients MOC.md` should exist.

**Mistake 2: CLAUDE.md without the vault path**
> `CLAUDE.md` says "I have an Obsidian vault with memory", but gives no path. Claude doesn't know where it is.

Fix: always give the absolute path. At minimum: `Obsidian vault: /Users/x/ObsidianVault/`.

**Mistake 3: Templates in the vault root**
> Template files get mixed up with real notes. Confusion.

Fix: keep all templates strictly in `Templates/`. Point the Obsidian Templates plugin at that folder.

**Mistake 4: Duplication between Claude Memory and Obsidian**
> The same information is stored both in `~/.claude/.../memory/` and in the vault.

Fix: a clear split. Quick behavioral rules + bridge → Claude Memory. Long-term content → Obsidian. Threshold: if an entry will outlive 10+ sessions, it goes to Obsidian.

**Mistake 5: Vault without Git**
> The user works with the vault for a year with no backup. One bad MCP write wipes everything.

Fix: `git init` in the vault right away. Commit periodically. Even locally, without pushing.

---

## What NOT to do

- **Don't put secrets in the vault.** API tokens and passwords go in `Credentials/` only as placeholders ("Stripe API key — look in 1Password"), OR not in the vault at all.
- **Don't create files like "ProjectX - Copy 2 final (2).md".** The vault is a living system. If you need versions, that's what git is for.
- **Don't make the vault public** until you've scrubbed people's names and business details.
- **Don't install MCP Obsidian** if you don't plan to use it — unnecessary setup.
- **Don't create 50 tags up front.** Add tags as you need them, not in advance.

---

## Additional resources

- [references/architecture.md](references/architecture.md) — details on the 3-tier memory
- [references/session-protocol.md](references/session-protocol.md) — rules for starting/ending a session
- [references/hooks-setup.md](references/hooks-setup.md) — setting up automation hooks
- [references/mcp-setup.md](references/mcp-setup.md) — details on MCP Obsidian
- [examples/CLAUDE-md-example.md](examples/CLAUDE-md-example.md) — an example `CLAUDE.md` with vault integration
- [examples/example-vault-tree.md](examples/example-vault-tree.md) — an example of a filled-in vault

---

## Integration with other skills

This skill creates **the foundation of the memory**. Other skills can write to the vault:

- CRM skills → write to `People/Clients/`
- Reporting skills → write to `Memory/Daily Reports/`
- Lessons-learned skills → write to `Memory/Lessons Learned.md`

The vault structure ensures these entries don't conflict.
