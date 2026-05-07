# NotePlan Import Plugin

Save Claude's long-form markdown research output into NotePlan 3 on macOS.

## Usage

Natural language (auto-invokes the skill):

```
> save this to noteplan
```

Explicit slash command:

```
/noteplan-import:import
```

With a folder override:

```
/noteplan-import:import Architecture
```

## What it does

1. Identifies the most recent substantial markdown research output in the current conversation
2. Resolves the NotePlan documents directory (iCloud Drive, sandbox container, or legacy fallback)
3. Detects the note file extension in use (defaults to `.txt`)
4. Writes the note under `Notes/Research/` (or your overridden folder) with `# <Title>` and a `#research` tag prepended
5. Opens the note in NotePlan via `noteplan://x-callback-url/openNote`

## Installation

Enable it in your `~/.claude/settings.json`:

```json
"enabledPlugins": {
  "noteplan-import@markupboy-claude": true
}
```
