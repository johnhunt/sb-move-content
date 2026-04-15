---
name: "Library/MoveContent"
tags: meta/library
---

# Move Content

A [SilverBullet](https://silverbullet.md) v2 Space Lua library that adds a **Selection: Move Content** command. Select any text on a page, run the command, pick a target page from a fuzzy-filter page picker (or create a new one), and the selection is appended to that page and removed from the source.

Works with any content — prose, lists, code blocks, Space Lua, whatever is inside the selection range.

## Implementation

```space-lua
-- priority: 10

moveContent = moveContent or {}

-- Sentinel name used for the "create a new page" option in the picker.
moveContent.newPageOption = "✨ New page…"

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

-- Build the filterBox options: a pinned "new page" entry, then all existing
-- pages sorted by lastModified desc (most recent first), excluding the
-- current page.
--
-- Note: p.lastModified is a string (ISO timestamp). ISO 8601 strings sort
-- lexicographically in chronological order, so we compare them directly
-- with `>` — do NOT try to negate them (attempting `-string` blows up with
-- "attempt to unm a 'string'").
function moveContent.buildPickerOptions(currentPage)
  -- Collect and sort pages (most recently modified first).
  local pages = {}
  for _, p in ipairs(space.listPages()) do
    if p.name != currentPage then
      table.insert(pages, p)
    end
  end
  table.sort(pages, function(a, b)
    return (a.lastModified or "") > (b.lastModified or "")
  end)

  -- orderId is used by the filterBox as a tie-breaker on fuzzy-match score,
  -- with lower values ranked higher. Use a large negative integer for the
  -- "new page" sentinel so it pins to the top; then positive integers
  -- starting at 1 preserve the sort order for real pages.
  local options = {
    {
      name = moveContent.newPageOption,
      hint = "Create",
      orderId = -1000000000
    }
  }
  for i, p in ipairs(pages) do
    table.insert(options, {
      name = p.name,
      orderId = i
    })
  end
  return options
end

command.define {
  name = "Selection: Move Content",
  run = function()
    local sel = editor.getSelection()
    if not sel or sel.from == sel.to then
      editor.flashNotification(
        "No selection to move — highlight some text first.", "error")
      return
    end

    local fullText = editor.getText()
    -- Editor positions are 0-indexed (CodeMirror); Lua string.sub is 1-indexed
    -- and inclusive, so add 1 to `from` and leave `to` as-is.
    local selected = string.sub(fullText, sel.from + 1, sel.to)

    local currentPage = editor.getCurrentPage()

    -- Step 1: pick an existing page, or choose the "new page" sentinel.
    local options = moveContent.buildPickerOptions(currentPage)
    local picked = editor.filterBox(
      "Move to",
      options,
      "Pick a page to move the selection into, or choose '" ..
        moveContent.newPageOption .. "' to create a new one.",
      "Type to filter pages"
    )
    if not picked then
      -- User cancelled.
      return
    end

    -- Step 2: resolve the target name. If they picked "new page", prompt.
    local target
    if picked.name == moveContent.newPageOption then
      local entered = editor.prompt("New page name:")
      if not entered then return end
      target = string.gsub(entered, "^%s+", "")
      target = string.gsub(target, "%s+$", "")
      if target == "" then
        editor.flashNotification("New page name cannot be empty.", "error")
        return
      end
      if target == currentPage then
        editor.flashNotification(
          "Target page is the current page — nothing to do.", "error")
        return
      end
    else
      target = picked.name
    end

    -- Step 3: append (or create) and remove from source.
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
