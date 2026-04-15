---
name: "Library/MoveContent"
tags: meta/library
---

# Move Content

A [SilverBullet](https://silverbullet.md) v2 Space Lua library that adds a **Selection: Move Content** command. Select any text on a page, run the command, pick a target page, and the selection is appended to that page (creating it if needed) and removed from the source.

Works with any content — prose, lists, code blocks, Space Lua, whatever is inside the selection range.

## Implementation

```space-lua
-- priority: 10

moveContent = moveContent or {}

-- Ensure the existing page content ends with a blank line before appending,
-- so the new content is clearly separated.
function moveContent.withSeparator(existing)
  if not existing or existing == "" then
    return ""
  end
  -- Guarantee exactly one trailing blank line.
  local trimmed = string.gsub(existing, "[\r\n]+$", "")
  return trimmed .. "\n\n"
end

command.define {
  name = "Selection: Move Content",
  run = function()
    local sel = editor.getSelection()
    if not sel or sel.from == sel.to then
      editor.flashNotification("No selection to move — highlight some text first.", "error")
      return
    end

    local fullText = editor.getText()
    -- editor positions are 0-indexed (CodeMirror); Lua string.sub is 1-indexed
    -- and inclusive, so add 1 to `from` and leave `to` as-is.
    local selected = string.sub(fullText, sel.from + 1, sel.to)

    local currentPage = editor.getCurrentPage()
    local target = editor.prompt("Move selection to page:")
    if not target then return end
    target = string.gsub(target, "^%s+", "")
    target = string.gsub(target, "%s+$", "")
    if target == "" then
      editor.flashNotification("Target page name cannot be empty.", "error")
      return
    end
    if target == currentPage then
      editor.flashNotification("Target page is the current page — nothing to do.", "error")
      return
    end

    local existing = ""
    if space.pageExists(target) then
      existing = space.readPage(target) or ""
    end

    local newContent = moveContent.withSeparator(existing) .. selected
    space.writePage(target, newContent)
    editor.replaceRange(sel.from, sel.to, "")

    editor.flashNotification("Moved selection to [[" .. target .. "]]", "info")
  end
}
```
