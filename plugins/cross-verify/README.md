# Cross-Verify Skill

AI cross-verification skill for Claude Code. Compares answers from Codex (GPT), Gemini, and Claude to provide synthesized analysis.

## Requirements

- Claude Code (latest)
- At least one of:
  - **Codex CLI**: `npm install -g @openai/codex`
  - **Gemini CLI**: `npm install -g @google/gemini-cli`

## Installation

Copy the `cross-verify/` folder to your Claude Code skills directory:

```bash
cp -r cross-verify/ ~/.claude/skills/cross-verify/
```

## Agent Teams (Optional, Recommended)

For parallel execution of CLI agents, enable Agent Teams in your `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Without Agent Teams, agents run sequentially (slower but functional).

## Usage

```
/cross-verify Is this authentication implementation secure?
/cross-verify Review the architecture of this module
/cross-verify What's the best approach for caching in this service?
```

Or use natural language:
- "cross check this code"
- "ask other AIs about this approach"
- "multi AI comparison on this bug"

## How It Works

1. Checks which CLIs are available (codex, gemini)
2. Spawns reviewer agents in parallel (or sequentially as fallback)
3. Lead (Claude) also analyzes the same question
4. Synthesizes all results into: Consensus / Unique Insights / Conflicts / Final Conclusion

## File Structure

```
cross-verify/
├── SKILL.md              # Main orchestration logic
├── agents/
│   ├── codex-reviewer.md # Codex CLI agent definition
│   └── gemini-reviewer.md# Gemini CLI agent definition
└── README.md             # This file
```
