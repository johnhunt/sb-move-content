# sb-move-content

A [SilverBullet](https://silverbullet.md) **v2** Space Lua library that adds a **Selection: Move Content** command.

> Written by [Claude Code](https://claude.com/claude-code) (Anthropic's CLI for Claude) on behalf of the repo owner. The owner reviewed and published it but did not hand-write the code.

## What it does

Highlight any content on the current page — prose, a list, a code block, Space Lua, anything inside a contiguous selection — then run **Selection: Move Content** from the command palette. You'll be prompted for a target page name. The selection is:

- **appended** to the bottom of the target page (creating the page if it doesn't exist), and
- **removed** from the source page.

Similar in spirit to SilverBullet's built-in **Page: Extract**, but instead of creating a new page and leaving a `[[wikilink]]` behind, this moves the content into an existing (or newly-created) page and cleans up the source entirely.

## Requirements

- SilverBullet **v2** (Space Lua). This will not work on v1.

## Installation

### Recommended — built-in `Library: Install`

1. Open the command palette (`Ctrl-/` or `⌘-/`) and run **Library: Install**.
2. When prompted for **URI**, paste:

   ```
   https://github.com/johnhunt/sb-move-content/blob/main/PLUG.md
   ```
3. SilverBullet downloads the library, writes it to `Library/MoveContent` in your space, and reloads. Done.

To update later, run **Library: Update All** (or **Update** from the Library Manager). To remove, use the Library Manager's **Remove** button.

### Alternative — copy the file manually

1. Copy [`PLUG.md`](PLUG.md) into your space (anywhere under the space root works; `Library/MoveContent.md` matches the convention).
2. Run **System: Reload** (`Ctrl-Alt-R` / `⌃⌥R`) or refresh the browser tab.

## Usage

1. Select some content on any page.
2. Open the command palette and run **Selection: Move Content**.
3. A fuzzy-filter page picker appears listing every page in your space (most-recently-modified first, current page excluded). Either:
   - Type to filter, then pick an existing page — the selection is appended to it, or
   - Pick **✨ New page…** at the top of the list to create a new target, then enter its name.

A flash notification confirms the move and references the target page.

### Edge cases

- Empty selection → error notification, no change.
- Empty target name (when creating) → error notification, no change.
- Target page is the current page → rejected (would be a no-op at best, destructive at worst).
- Existing target content → separated from the appended selection by a blank line.

## License

[MIT](LICENSE)

## Attribution

Code written by [Claude Code](https://claude.com/claude-code) on model Claude Opus 4.6. Commit messages include co-author attribution.
