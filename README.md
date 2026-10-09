# 🔁🛡️ Recurring Issues AI Skill

![Recurring Issues AI Skill Banner](banner.png)

> An AI coding skill that permanently quashes repeat bugs.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE.txt) [![Agent Skill](https://img.shields.io/badge/Agent%20Skill-SKILL.md-8A2BE2.svg)](https://github.com/anthropics/skills) [![Version](https://img.shields.io/github/v/release/adriangrantdotorg/recurring-issues-ai-skill?color=orange&label=Version)](https://github.com/adriangrantdotorg/recurring-issues-ai-skill/releases) [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/adriangrantdotorg/recurring-issues-ai-skill/pulls)

---

## ⬇️ Why Install?

- 🔁 **No third report** — the second mention ends the bug for good
- 💸 **$0 added cost** — runs inside the AI you already use
- 🛠️ **Nothing to set up** — one folder of plain Markdown, no dependencies

| | 😩 Without this Skill | 😌 With this Skill |
| --- | :---: | :---: |
| 🔁 Times you report the same bug | 🧪 3+ | **2** |
| 🩹 Places a fix lands | one screen | **the source of the bug** |
| 🚨 Tests that catch a comeback | 0 | **1+** |
| 📒 Record of past fixes | none | **`RECURRING.md`** |

<sub>🧪 estimate</sub>

---

## ✨ Features

Before fixing any bug, the AI checks whether it has been fixed before, and a repeat gets a fix that can't quietly come back.

![The same repeat bug report with and without the skill: without it, the bug is patched, comes back and is patched again; with it, the AI checks history, fixes the source and adds a guard, and the bug is fixed for good](docs/media/with-vs-without-skill.svg)

- 🔎 **Spots repeats on its own** — checks the ledger, project notes and git history first
- 🗣️ **Hears the word "again"** — also "still", "keeps happening", "I already asked"
- 🎯 **Fixes the source, not the screen** — one change where the wrong thing is made
- 🧹 **Catches every copy** — sweeps the codebase, including the twin code path
- 🚨 **A guard that fails on regression** — a test or check, never just a note
- 🔬 **Reproduces before fixing again** — a fix that didn't hold means a wrong theory
- 📒 **A ledger for next time** — one greppable line per fix in `RECURRING.md`

---

## 🚀 Installation

Needs an AI assistant that supports [Agent Skills](https://github.com/anthropics/skills). Works best in a git repo, where past fixes are easy to find.

```bash
# Claude Code
git clone https://github.com/adriangrantdotorg/recurring-issues-ai-skill.git ~/.claude/skills/recurring-issues-ai-skill
# Cursor
git clone https://github.com/adriangrantdotorg/recurring-issues-ai-skill.git ~/.cursor/skills/recurring-issues-ai-skill
# ChatGPT & Codex
git clone https://github.com/adriangrantdotorg/recurring-issues-ai-skill.git ~/.agents/skills/recurring-issues-ai-skill
```

| **Platform** | **Skills folder** |
| --- | --- |
| **[Claude Code](https://code.claude.com/docs/en/skills)** | `~/.claude/skills/` |
| **[Cursor](https://cursor.com/docs/skills)** | `~/.cursor/skills/` |
| **[ChatGPT & Codex](https://learn.chatgpt.com/docs/build-skills)** | `~/.agents/skills/` |

---

## 💡 Usage

Report bugs the way you normally do; the skill kicks in on its own.

| You say | The skill makes |
| --- | --- |
| "The save button does nothing." | A **normal fix**, plus a first line in `RECURRING.md` so a repeat is caught |
| "The dropdown is cut off again." | A **permanent fix**: the source fixed, every copy swept, a test that fails if it returns |
| "Find a permanent fix for the focus bug." | A **guard test** and a written rule that tells future sessions how to spot it |

---

<div align="center">
  <sub>Built with ❤️ for the AI coding community & everyone tired of fixing the same bug twice ✌🏾</sub>
</div>
