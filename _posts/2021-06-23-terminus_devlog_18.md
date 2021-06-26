---
title: Terminus Devlog 18 - Milestone 2 Completed
layout: post
image: https://www.dropbox.com/s/yaguudigx6ctpfc/postimage-devlog18.png?raw=1
excerpt: Finish the refactor, can we build a game now? ![Video](https://www.dropbox.com/s/7dkutjwjfd73exe/terminus_devlog_18.gif?raw=1)
---

> ## Items Completed
> - [#78 - Add hot-key slots for using items, powers, etc...](https://github.com/tredfern/terminus/issues/78)
> - [#84 - Hotslot UI that allows for quickly using items](https://github.com/tredfern/terminus/issues/84)
> - [#117 - Characters should be tracked within World entity state](https://github.com/tredfern/terminus/issues/117)
> - [#109 - Graphics behaviors are using traditional class/oop approach -> Research switching to simple tables + redux approach](https://github.com/tredfern/terminus/issues/109)

## Milestone 2 Completed!
<pre>
───────────────────────────────────────────────────────────────────────────────
March 10 - June 24
───────────────────────────────────────────────────────────────────────────────
Total Commits:       <strong>159</strong>
Issues Closed:       <strong>40</strong>
Files Update:        <strong>424</strong>
LOC Added:           <strong>9411</strong>
LOC Deleted:         <strong>4119</strong>
</pre>

## Hotkeys and Quick Action UI

The newest improvement was to add hotkeys that are dynamic along with UI buttons that update with the new items.


```lua
-- from game/rules/player/actions.lua
function Actions.setHotKey(key, name, image, handler)
  return {
    type = Actions.types.SET_HOT_KEY,
    payload = {
      hotkey = key,
      name = name,
      image = image,
      handler = handler
    }
  }
end

-- from game/rules/player/reducer.lua
  [types.SET_HOT_KEY] = function(state, action)
    state.hotkeys = state.hotkeys or {}
    state.hotkeys[action.payload.hotkey] = {
      name = action.payload.name,
      image = action.payload.image,
      handler = action.payload.handler
    }
    return state
  end,
```
_Hotkeys are managed by the `player` slice._

Adding hotkeys is an action dispatched to the player slice. No characters beside the player have hotkeys so this makes sense.
The hotkey takes information on the `name`, `image`, and `handler` for the item/action in the hotkey. This came together
quite quickly.

```lua
-- Entire game/ui/main_screen/quick_slots.lua
local Components = require "moonpie.ui.components"
local Player = require "game.rules.player"
local connect = require "moonpie.redux.connect"
local tables = require "moonpie.tables"
local SpriteImage = require "game.ui.widgets.sprite_image"

local Slot = Components("quick_slots_slot", function(props)
  return {
    id = "hotkey_" .. props.hotkey,
    SpriteImage { id = "HotKeyImage", sprite = props.action.image },
    Components.text { text = props.hotkey, style = "align-bottom align-right" },
    click = props.action.handler,
  }
end)

local QuickSlots = Components("quick_slots", function(props)
  local hotkeys = props.hotkeys or {}
  return {
    hotkeys = hotkeys,
    render = function(self)
      return tables.map(self.hotkeys, function(item)
        return Slot { action = item.value, hotkey = item.key }
      end)
    end
  }
end)


return connect(QuickSlots, function(state)
  local hotkeys = Player.selectors.getHotKeys(state) or {}
  local sortedKeys = tables.sortBy(hotkeys, function(k) return k end)
  return {
    hotkeys = sortedKeys
  }
end)
```

This is the entire UI file for rendering out the quick slots/hotkeys. As items are updated, the component will
refresh and re-render with the new widgets. Items are used and removed, the hotkeys are cleared out and the UI
updates. Very simple and very dynamic.

_Potential improvement would be to allow using icons provided from moonpie._

## Refactor Characters

Characters have been cleaned up some in how they are managed. Mostly on the file management side of things.
More refactoring is likely to occur as the character creation piece is built out. But I kept characters outside
of the `entities` are of the world ruleset. I'm not sure if this is good or bad, but it helps sort these
into their own domain. So, it's kind of handy for that. Just like leaving map entities tied to the map
directly as opposed to trying to generalize everything.

## Refactor Graphics / Sprites

Sprite and graphics went through a simple clean up that reduced code considerably and also made it easier to
save/load data, swap out graphics if new tilesets/images become a thing. The main change was to make a wrapper
that pulls the image when it wants to draw directly from a store as opposed to referencing image data directly
via a pointer/reference.

## Milestone 3: Characters and Bestiary

The characters and bestiary milestone will focus on bring the game world more to life with more choices for character
creation to make that part feel more complete. It will also build out the possible ecosystem of creatures in the world
to help give different strategies, levels of risk, and just new interactions. 

The other goal for this milestone is to invest more in the planning upfront to better plan what I expect the milestone
to focus on so at the end I can clearly demonstrate what has been added.

## Statistic Report
<pre>
───────────────────────────────────────────────────────────────────────────────
Tag: devlog-18
───────────────────────────────────────────────────────────────────────────────
Files Update:        <strong>65</strong>
LOC Added:           <strong>982</strong>
LOC Deleted:         <strong>873</strong>
Code Coverage:       <strong>97%</strong>

───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Lua                        619     29014     3995      2847    22172       1434
Markdown                    22      1236      323         0      913          0
ReStructuredText            13       591      182         0      409          0
Plain Text                   5       658      108         0      550          0
CSV                          4      1175        0         0     1175          0
JSON                         3       103        0         0      103          0
License                      2        42        8         0       34          0
YAML                         2        66       14         0       52          0
Batch                        1        35        8         1       26          5
Makefile                     1        20        4         7        9          0
Python                       1        56       15        29       12          0
gitignore                    1        51       10         9       32          0
───────────────────────────────────────────────────────────────────────────────
Total                      674     33047     4667      2893    25487       1439
───────────────────────────────────────────────────────────────────────────────
</pre>

[All the changes](https://github.com/tredfern/terminus/compare/devlog-18...devlog-17)

![Video](https://www.dropbox.com/s/7dkutjwjfd73exe/terminus_devlog_18.gif?raw=1)