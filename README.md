# drawtree-skill — AI client contract for the Draw Tree Protocol

This repository holds the **AI-client behavioural contract** that the Draw
Tree Protocol expects clients to follow. The same instructions ship in three
wrappers so different runtimes can install whichever shape they understand:

| File | For |
|---|---|
| [`SKILL.md`](./SKILL.md) | Anthropic Skills format (Claude Code, Claude.ai, Claude Desktop, Perplexity Computer skills) |
| [`AGENTS.md`](./AGENTS.md) | Codex CLI and any tool that follows the `agents.md` convention |
| [`system_prompt.md`](./system_prompt.md) | ChatGPT custom GPTs, Claude project instructions, any "paste these instructions" surface |

The instruction body is identical across the three files. The wrappers only
differ in:

- Front-matter (Anthropic Skills require `---` YAML; AGENTS.md does not; a
  system prompt is just text).
- A short trailer in `AGENTS.md` explaining where Codex auto-loads it from.

## Why this is in its own repo

The protocol spec (Draw-Tree/drawtree-protocol) defines *what* a tree must
look like. This repo defines *how* an AI client behaves while building one
— the entry gate, the strict ordering of Phase 1 stages, the
never-chain-stages rule, the "never paraphrase the user" rule, etc.

You can change models (Grok 4, Claude Sonnet, GPT-5, Gemini), change AI
clients (ChatGPT, Claude, Perplexity, Codex), or even change the server
implementation, and as long as the client honours this skill, the
research-design behaviour is the same.

## Install

### Claude Code

```bash
mkdir -p ~/.claude/skills/drawtree
cp SKILL.md ~/.claude/skills/drawtree/SKILL.md
```

### Claude.ai / Claude Desktop

Zip the file and upload through **Settings → Skills → Upload skill**.

### Codex CLI

```bash
mkdir -p ~/.codex
cp AGENTS.md ~/.codex/AGENTS.md
```

### ChatGPT / Perplexity / generic system prompt

Paste the contents of `system_prompt.md` into your custom GPT's
instructions, your Project's instructions, or your client's system prompt.

## Compatibility

This skill is written against the **Draw Tree Protocol v0.3**. It will be
versioned alongside the protocol.

## License

MIT.
