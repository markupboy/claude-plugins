---
description: >
  Save the most recent long-form markdown research output from this
  conversation into NotePlan 3 on macOS as a searchable note.
  Use when the user asks to "save this to NotePlan", "send the research
  to NotePlan", "import this into NotePlan", "save the research note",
  "put this in NotePlan", "archive this in NotePlan", or any phrasing
  that asks to persist a deep-research / long markdown answer into
  NotePlan. Files under Notes/Research/ by default and auto-opens.
argument-hint: [folder]
---

You will save the most recent substantial markdown research output from this conversation into NotePlan 3 as a note, then open it via the NotePlan URL scheme. Follow these steps exactly.

## 1. Parse the argument

The optional argument is a destination subfolder name relative to `Notes/`.

- Trim whitespace.
- If empty or missing, use `Research`.
- Reject names that contain `/`, `..`, start with `@`, or start with `.`. If invalid, ask the user for a valid folder name and stop.

Record the result as `FOLDER`.

## 2. Identify the source markdown

Scan the conversation history newest-first and pick the candidate by these rules in order:

1. The **most recent assistant message** (excluding any output produced by this skill itself) that has substantial markdown content — meaning at least one of:
   - 400+ words of prose
   - An `# H1` plus 3+ `## H2` sections
   - Explicit "deep research" / "research report" framing

2. If the user's invoking message points to a specific earlier output ("save the architecture deep-dive", "save my last research"), prefer that disambiguation.

3. If no assistant message in history meets the bar, do not guess. Tell the user: "I don't see a research-style markdown output in this conversation to save. If you meant a specific earlier message, quote a phrase from it or re-paste the content." Stop.

4. If multiple plausible candidates exist and the user just said "save it", list each candidate by its first H1 (or first 80 chars of the first paragraph) and ask which one. Do not auto-pick.

The chosen text is `BODY`.

## 3. Resolve the NotePlan documents directory

Probe in order; use the first that exists:

```bash
for p in \
  "$HOME/Library/Mobile Documents/iCloud~co~noteplan~NotePlan/Documents" \
  "$HOME/Library/Containers/co.noteplan.NotePlan3/Data/Library/Application Support/co.noteplan.NotePlan3" \
  "$HOME/Library/Containers/co.noteplan.NotePlan/Data/Library/Application Support/co.noteplan.NotePlan"; do
  if [ -d "$p" ]; then echo "$p"; break; fi
done
```

If none resolve, abort with: "NotePlan 3 doesn't appear to be installed at a standard location. This plugin requires macOS with NotePlan 3."

Record the resolved path as `DOCS_DIR`.

## 4. Detect the note file extension

Sample the destination folder first, then fall back to `Notes/`:

```bash
ls -1 "$DOCS_DIR/Notes/$FOLDER" 2>/dev/null \
  | grep -E '\.(txt|md)$' \
  | sed -n 's/.*\.\(txt\|md\)$/\1/p' \
  | sort | uniq -c | sort -rn | head -1
```

If empty, repeat against `"$DOCS_DIR/Notes"`. If still empty, default to `txt` (NotePlan's default). Use the dominant extension. Record as `EXT` (with leading dot, e.g. `.txt`).

## 5. Extract and sanitize the title

Title source: the first line of `BODY` matching `^# +(.+?)\s*$`. If absent:

1. If the first non-empty line is short (< 100 chars) and not a code fence, use it.
2. Otherwise, ask the user for a title. Do not invent one.

Sanitize in this order:

1. Trim leading/trailing whitespace.
2. Replace `/` with `-`.
3. Replace `:` with ` -`.
4. Collapse newlines, tabs, and control chars to single spaces.
5. Collapse runs of whitespace to a single space.
6. Strip trailing dots and spaces.
7. Truncate to 120 chars at a word boundary if convenient.
8. If the result is empty, fall back to `Research <YYYY-MM-DD HHMM>` using the current local time.

The result is `TITLE`. The filename stem `STEM` is identical to `TITLE` (NotePlan uses the filename as the displayed title).

## 6. Ensure the destination folder exists

```bash
mkdir -p "$DOCS_DIR/Notes/$FOLDER"
```

The iCloud path contains spaces, so quote it. If `mkdir -p` fails (e.g. permission error or iCloud paused), surface the error verbatim and stop.

## 7. Handle filename collisions

Compute `OUT_PATH = "$DOCS_DIR/Notes/$FOLDER/$STEM$EXT"`.

- If `OUT_PATH` does not exist: use it.
- If it does: try `$STEM (2)$EXT`, then ` (3)`, … through ` (9)`. Beyond that, append a ` <YYYY-MM-DD-HHMMSS>` stamp.
- Never overwrite. Never ask.

## 8. Compose the file content

The body to write:

```
# <TITLE>
#research

<BODY>
```

Where `<BODY>` is the original `BODY` with the original H1 line removed *if* it matched the title chosen in step 5 (avoid duplication). Preserve everything else verbatim — tables, code fences, links, NotePlan task syntax (`- [ ]`, `* foo`, `+ foo`). Do not reflow. End the file with a single trailing newline.

Special cases:

- If `BODY` opens with YAML frontmatter (`---\n…\n---\n`), preserve it as-is and inject `# <TITLE>` and `#research` *after* the closing `---`.
- If `BODY` already has `^#research(\s|$)` within its first 5 lines, skip prepending the tag.

Write the file to `OUT_PATH`.

## 9. Open the note in NotePlan

URL-encode the relative path with Python (handles Unicode, `&`, `?`, etc.):

```bash
ENCODED=$(python3 -c 'import sys, urllib.parse; print(urllib.parse.quote(sys.argv[1]))' "Notes/$FOLDER/$STEM$EXT")
open "noteplan://x-callback-url/openNote?filename=${ENCODED}&openNote=yes"
```

If `python3` is unavailable, fall back to `perl -MURI::Escape -e 'print uri_escape(shift)' "Notes/$FOLDER/$STEM$EXT"`. If `open` exits non-zero, surface the error but treat the save itself as successful — the file is on disk.

## 10. Report to the user

One line:

```
Saved to NotePlan: Notes/<FOLDER>/<STEM><EXT>
```

If a collision suffix was applied, mention it (`Saved as "<STEM> (2)<EXT>" because the original name already existed.`). If `open` failed, print the absolute path so the user can open the file manually.
