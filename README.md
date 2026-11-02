# rules-not-prompts

**I can't write Swift. I shipped a 62,299-line iOS app with AI in a year.**

Not by learning Swift. By writing the AI a set of rules it has to read before every single change.

```
6 red lines        — the ones that fail silently. Compile fine, pass tests, break on a user's device.
13 constraints     — how to make a change, verify it, and commit it.
Path-activated     — specialty rules load only when you touch a matching file.
A casebook         — the real accident behind every single rule.
```

The app won't be open source. This will be.

**中文 → [README.zh-CN.md](README.zh-CN.md)**

---

## Why rules, not prompts

Everyone who writes code with AI hits the same wall: you write a paragraph of instructions, and it still does the wrong thing.

A prompt asks. A rule constrains.

The difference is **verifiability**. A good rule can be checked:

- a command you can run (build, test)
- an artifact you can see (commit, diff)
- a failure you can name (compile error, crash log)

A rule that fails all three is just prose the agent reads and forgets.

Every rule in this repo comes with the incident that produced it. That's the whole point — [read the casebook](docs/CASEBOOK.md) and you'll see what a bad rule costs.

---

## Why only ONE rules file

This is the most important design decision here.

`CLAUDE.md` and `AGENTS.md` are both auto-loaded by agents. If both hold a copy of your rules:

1. **The same rules load twice** — wasted context budget
2. **The copies drift** — you edit one, forget the other, and the agent reads whichever it finds first
3. **After the drift, nobody knows which one is correct**

So: `AGENTS.md` is the single source of truth. `CLAUDE.md` is one line:

```markdown
@AGENTS.md
```

If you need to feed different tools differently, use an import or a symlink. **Never copy the content.**

---

## Why rules load by path

`.claude/rules/` files activate via frontmatter `paths`:

```yaml
---
paths:
  - "**/*.swift"
---
```

Swift rules load when you edit a Swift file. Dependency rules load when you edit the project file. Nothing else sits in context.

Always-on rules dilute each other. An agent reading 300 lines of rules will actually follow the first few.

**One trap worth knowing**: every `paths` entry is a file glob that must match at least one real file. Write a symbol name or a deleted filename and the rule **never activates — with no error**.

---

## Install

```bash
git clone https://github.com/<you>/rules-not-prompts
cd rules-not-prompts

# copy into your project
cp -r .claude  /path/to/your-project/
cp AGENTS.md  /path/to/your-project/
cp CLAUDE.md  /path/to/your-project/

# find every placeholder
grep -rn "⟨" /path/to/your-project/
```

Then replace the ⟨angle-bracket⟩ placeholders with your own paths, build commands, and test commands.

**Start with three rules, not thirteen.** Run a week. The more rules you have, the more the agent will read "not mentioned" as "allowed".

---

## Layout

```
rules-not-prompts/
├── README.md                    ← English
├── README.zh-CN.md              ← 中文
├── AGENTS.md                    ★ single source of truth (rules + project manual)
├── CLAUDE.md                    one-line import pointer
├── .claude/
│   └── rules/
│       ├── swift.md             activates on **/*.swift
│       ├── frontend.md          activates on your UI source dir
│       ├── security.md          activates on source + test dirs
│       └── dependencies.md      activates on your project file
├── docs/
│   ├── RULES.md                 numbering, sources, rule lifecycle
│   ├── ADAPTING.md              switching to TypeScript / Python / Web
│   └── CASEBOOK.md              8 real accidents
└── promo/
    └── content-calendar-30d.md  30-day content plan
```

---

## The six red lines

Each one targets an error that **fails silently** — compiles, passes tests, breaks for the user.

| # | Forbidden | Why |
|---|---|---|
| R001 | Traversing a framework's private navigation stack | Private type, no stable API, breaks on upgrade |
| R002 | Business logic in the view layer | Logic scatters, tests can't reach it |
| R003 | Mixing package managers | Two resolutions for one dependency |
| R004 | Hand-editing project file internals | The toolchain silently drops your change |
| R005 | Unapproved third-party SDKs | Supply chain and compliance risk |
| R006 | Debug logic inside live services | Debug code ships with the release |

Full numbering and lifecycle: [docs/RULES.md](docs/RULES.md). The accident behind each: [docs/CASEBOOK.md](docs/CASEBOOK.md).

---

## Cross-stack

The skeleton is language-agnostic. The four files in `rules/` are Swift/iOS examples — rewrite them per [docs/ADAPTING.md](docs/ADAPTING.md), which includes ready-to-adapt versions for TypeScript/React, Python, and generic build files.

| Tool | Reads |
|---|---|
| Claude Code | `CLAUDE.md` (imports `AGENTS.md`) + `.claude/rules/` |
| Codex / Cursor / Grok Build / … | `AGENTS.md` |

---

## License

MIT. Take it, change it, ship it. No attribution required.

One request: if a rule here stops a real bug for you, open an issue and tell me. That's worth more than a star.

---

*Extracted from a real one-person iOS project, then anonymized. At extraction: 1,111 commits / 333 Swift files / 62,299 lines / 96 test files / 0 third-party dependencies.*
