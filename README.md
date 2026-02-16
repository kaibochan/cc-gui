# cc-gui

A UI library for ComputerCraft computers and monitors. Focused on being easy to use on-device: composition of elements, a single render pass, and event dispatching.

## Features
- Hierarchical `Element` model with local buffers and child composition.
- `Display` that composes element buffers and writes a single blit pass to the device.
- Text and widget primitives (`Text`, `Bar`, `Graph`) for common UI needs.
- Simple event dispatch helpers: `gui.doEvents()` and `gui.getEvents()`.

## Installation
Copy this repository into your ComputerCraft filesystem under `/apis/gui`. From an on-device program you can then `require("gui")`.

Example (from the computer):

```lua
-- assuming files are in /apis/gui
local gui = require("gui")
```

## Quick start

1. Attach/wrap your display device (monitor or terminal) and initialize a `Display`:

```lua
local monitor = peripheral.wrap("top")
local display = gui.initializeDisplay(monitor)
```

2. Create elements and add them to the root `window` (the `Display`'s root `Element`):

```lua
local title = gui.Text:new{
  x = 0, y = 0,
  width = display.width, height = 1,
  text = "Status: OK",
}
display.window:addElement(title)
```

3. Render to the device and run an event loop:

```lua
display:render()
while true do
  gui.doEvents()
  -- optionally update state and call display:render() again
end
```

## Handling input
Event dispatch uses the `Display:getSelectedElement(x,y)` helper and calls any handler on the element matching the event name. Common patterns:

- Mouse events on the terminal use `mouse_click` / `mouse_up` / `mouse_drag`.
- Monitor input uses `monitor_touch`.

Attach an event handler to an element like this:

```lua
local button = gui.Element:new{ x=2,y=2,width=10,height=1 }
button.mouse_click = function(self, e)
  -- e.button, e.x, e.y available
  print("Button clicked")
end
display.window:addElement(button)
```

For monitor touches:

```lua
title.monitor_touch = function(self, e)
  -- e.display, e.x, e.y
end
```

`gui.doEvents()` converts pulled OS events into a small table and routes them to the selected element. See [init.lua](init.lua#L1-L400) for details.

## API overview
- `gui.initializeDisplay(display_device)` : create a `Display` bound to a device.
- `display:render()` : compose the element tree and blit to the device.
- `gui.doEvents()` / `gui.getEvents()` : pull and dispatch input/timer events.

Core modules (source): [Element.lua](Element.lua#L1-L200), [Display.lua](Display.lua#L1-L200), [Text.lua](Text.lua#L1-L200), [Bar.lua](Bar.lua#L1-L200), [Graph.lua](Graph.lua#L1-L200), [DataSet.lua](DataSet.lua#L1-L200), [Utils.lua](Utils.lua#L1-L200).

## Tips & best practices
- Build UIs by composing small elements into `display.window` instead of making many direct `blit` calls.
- Update element properties and call `display:render()` once per frame (or after a batch of changes).
- Reuse element buffers when running tight, frequently-updating loops to reduce allocations.