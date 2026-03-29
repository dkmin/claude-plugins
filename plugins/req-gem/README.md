# req-gem Skill

Gemini subagent skill for Claude Code. Sends questions to Gemini models via Gemini CLI.

## Requirements

- Claude Code (latest)
- **Gemini CLI**: `npm install -g @google/gemini-cli`

## Installation

```bash
cp -r req-gem/ ~/.claude/skills/req-gem/
```

## Usage

```
/req-gem What's the best caching strategy here?
/req-gem gemini-2.5-pro Review this authentication code
/req-gem gemini-2.5-flash Explain this error quickly
```

Default model: `gemini-2.5-pro`

## How It Works

1. Parses model name from first argument (or uses default)
2. Gathers code context if relevant (git diff, files)
3. Calls `gemini -p "{prompt}" -m {model} -o text`
4. Returns Gemini response as-is

## File Structure

```
req-gem/
├── skills/req-gem/
│   └── SKILL.md          # Skill orchestration logic
└── README.md             # This file
```
