---
name: autocad
description: Safe working method for driving AutoCAD through the autocad MCP server — reading a drawing, drawing, editing, saving. Use whenever the user asks to read, inspect, draw, annotate, measure or modify a DWG in AutoCAD (« plan », « calques », « dessine », « cote », « AutoCAD »).
---

# Working in AutoCAD with Claude

The `autocad` MCP server drives the AutoCAD instance running on this Windows PC through COM.
It needs full AutoCAD (or GstarCAD / ZWCAD). AutoCAD LT cannot be driven this way.

## Before anything else

1. Read the `drawing://current` resource: it gives the name of the drawing the server is
   working on. This is **not always** the drawing in front of the user in AutoCAD.
2. Tell the user which drawing you are about to read or modify.

## Reading

- Start with `list_layers`, then `list_entities` (filter by `entity_type`, keep `limit` small
  on large drawings), then `get_entity_properties` on the handles you need.
- Use `screenshot` to show the user what AutoCAD displays. It captures the AutoCAD window even
  when another window is in front.
- Objects exported from Revit often appear as proxy entities (`AcDbZombieEntity`): their
  geometry cannot be read reliably. Say so instead of guessing.

## Writing — protect the user's work

- **Never write into the user's original file.** Before any edit, ask for (or make) a copy of
  the DWG and open the copy with `open_drawing`.
- Re-check `drawing://current` right before writing and right before `close_drawing`: if the
  user clicked another drawing in between, stop and say so.
- Put everything you create on dedicated layers (`create_layer`, e.g. `CLAUDE_…`), never on the
  user's layers, so your work can be hidden or removed in one step.
- Verify what you drew by reading it back (`get_entity_properties`, `list_entities`), then a
  `screenshot`.
- `save_drawing` overwrites an existing file at that path. Save only the copy, and only when the
  user asks.

## If AutoCAD seems unresponsive

AutoCAD rejects calls while it is busy (dialog open, command running). Ask the user to press
`Esc` in AutoCAD and close any open dialog, then retry.
