# req-gpt Skill

GPT subagent skill for Claude Code. Sends questions to GPT models via Codex CLI.

## Requirements

- Claude Code (latest)
- **Codex CLI**: `npm install -g @openai/codex`

## Installation

```bash
cp -r req-gpt/ ~/.claude/skills/req-gpt/
```

## Usage

```
/req-gpt What's the best caching strategy here?
/req-gpt o3 Review this authentication code
/req-gpt gpt-4o Explain this error
```

Default model: `gpt-5.2`

## How It Works

1. Parses model name from first argument (or uses default)
2. Gathers code context if relevant (git diff, files)
3. Calls `codex exec -m {model} --full-auto "{prompt}"`
4. Returns GPT response as-is

## File Structure

```
req-gpt/
├── skills/req-gpt/
│   └── SKILL.md          # Skill orchestration logic
└── README.md             # This file
```
