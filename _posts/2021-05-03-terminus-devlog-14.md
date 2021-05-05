---
title: Terminus Devlog Week 14
layout: post
image: https://www.dropbox.com/s/qmhnmoh1kcvaxxs/postimage-devlog14.png?raw=1
excerpt: Field of View / Fog of War ![Video](https://www.dropbox.com/s/aqw6v4c3egi1m71/terminus_devlog_14.gif?raw=1)
---

> ## Items Completed
> - [#110 - Commands can accept directional parameters](https://github.com/tredfern/terminus/issues/110)
> - [#14 - Doors for rooms](https://github.com/tredfern/terminus/issues/14)
> - [#36 - Message log supporting colored text](https://github.com/tredfern/terminus/issues/36)
> - [#98 - LoS calculations for characters](https://github.com/tredfern/terminus/issues/98)
> - [#112 - Field of view should be blocked by walls](https://github.com/tredfern/terminus/issues/112)
> - [#111 - Fog of war for previously visited tiles](https://github.com/tredfern/terminus/issues/111)

## Implementing Fog of War

Fog of war systems are a key component of Roguelike games. Or really, any strategy/tactical computer game. Computers allow an impartial referee (_though most of us assume the computer is cheating_), to track and display only the
information we should know. We can also have the computer randomize some statistics that eliminates the potential
for meta-gaming where we know and understand more about the world than our characters do.

There are 2 key components to a Field of View/Fog of War system.
1. What can I see right this moment?
2. What am I aware of? Where have I been?

#### LOS

Before visiting how Terminus handles tracking vision, I want to point out a [great resource](http://www.adammil.net/blog/v125_Roguelike_Vision_Algorithms.html) about various vision algorithms. It gives a good sample of various approaches
for calculating visibility in Roguelikes. 

Terminus to start is using a `raycasting` implementation. It has some artifacts in the sight lines at this point.

```lua
-- excerpt from game/rules/field_of_vision/calculate.lua
return function(state, origin, radius)
  local vm = VisibilityMap:new()
  vm:setVisible(origin)

  local list = createTestList(origin, radius)

  -- iterate each one, retrieving whether from origin to test point it can be seen
  for _, testPoint in ipairs(list) do
    local lineList = getLineList(origin, testPoint)
    for _, v in ipairs(lineList) do
      if Position.distance(origin, v) > radius then
        break
      end

      vm:setVisible(v)
      if Helper.blocksSight(state, v) then
        break
      end
    end
  end

  return vm
end 
```
This file handles the loop that will check for what tiles are visible.

1. Generates a list of test points based on the sight radius for the player
2. For each test point, generate a list of points in a line from origin to test
3. Flag current point location as visible
4. Check if that point blocks sight, either break out and move to next test point or continue on line

All of the logic is pretty self explanatory. 


#### Retrieving Visible Tiles

With the redux/store style implementation, it makes it easy for any point to ask for a list of visible tiles.

```lua
-- game/rules/field_of_view/selectors/get_visible_positions.lua
local Position = require "game.rules.world.position"

return function(state, view)
  if state.fieldOfView then
    local vm = state.fieldOfView[view]
    local out = {}

    for k, v in pairs(vm) do
      -- TODO: This is required because of the OOP model of VisibilityMap #115
      if type(k) == "number" and v then
        table.insert(out, Position.fromKey(k))
      end
    end
    return out
  end
end 
```
This file retrieves the visible positions from state for a particular view. Nobody needs to be concerned
with how that is calculated or where it is stored. You get just this list of points and can work from it.

### Tracking Fog of War

Fog of war is an interesting problem. 

1. How do we keep track of tiles that have been visited?
2. Can our map change without the player seeing those changes?
3. What about features/items that don't move? Should they be visible?
4. Should items/features not update until the character refreshes sight again?

Basically, this means we need to keep track of the state of various tiles when the player was last there.
And that this could be different from the real state.


_3 files drive the majority of the fog of war implementation right now_
```lua
--
-- game/rules/fog_of_war/actions/update_position.lua
--
return function(perspective, position, tile)
  local key = nil
  if type(position) == "number" then
    key = position
    position = nil
  end

  return {
    type = actionTypes.UPDATE_POSITION,
    payload = {
      perspective = perspective,
      position = position,
      positionHashKey = key,
      tile = tile
    }
  }
end 

--
-- game/rules/fog_of_war/actions/update_perspective.lua
--
local Thunk = require "moonpie.redux.thunk"
local actionTypes = require "game.rules.fog_of_war.actions.types"
local FieldOfView = require "game.rules.field_of_view"
local updatePosition = require "game.rules.fog_of_war.actions.update_position"
local Map = require "game.rules.map"

return function(perspective)
  return Thunk(
    actionTypes.UPDATE_PERSPECTIVE,
    function(dispatch, getState)
      local state = getState()
      local visiblePoints = FieldOfView.selectors.getVisiblePositions(state, perspective)

      if visiblePoints then
        for _, pos in ipairs(visiblePoints) do
          local tile = Map.selectors.getTile(state, pos)
          dispatch(updatePosition(perspective, pos, tile))
        end
      end
    end
  )
end

--
-- game/rules/fog_of_war/reducer.lua
--
local createSlice = require "moonpie.redux.create_slice"
local actionTypes = require "game.rules.fog_of_war.actions.types"

return createSlice {
  [actionTypes.UPDATE_POSITION] = function(state, action)
    local perspective = action.payload.perspective
    local pos = action.payload.position
    local key = action.payload.positionHashKey or pos.hashKey
    local tile = action.payload.tile

    if not state[perspective] then state[perspective] = {} end

    state[perspective][key] = {
      tile = tile
    }

    return state
  end
}
```

I LOVE THIS IMPLEMENTATION! Not because it's particularly good, because it's not that. But because it
uses the redux/store implementation to update the fog of war position. This can be improved to track other
data in the state and not cause additional friction. 

1. Dispatch to the fog of war to update it's perspective
2. Fetch all the currently visible positions for that perspective
3. Dispatch an update about what is visible in each spot
4. Reducer collects final action and stores the information

It's straightforward to implement and it took little effort and fairly testable.

## Improvements to Consider

1. Reading about **redux sagas**, would fog of war implementation benefit from that. So basically we have an action to update visibility and a side effect is refreshing fog of war?
2. There are some OOP implementations for certain entities in the system. That is ok in general, sometimes that approach is best, but for things that are stored in state, I'd like the most basic objects possible. In fact, I'd like just pure tables stored in state.
3. Features like doors aren't yet tracked by FoW. It wouldn't be hard to add but they are missing today.
4. Eliminate artifacts by choosing a better LoS algorithm
5. Entity lookups are becoming the most costly, need to refactor character and items to use world/entity storage.


---
## Statistic Report
<pre>
───────────────────────────────────────────────────────────────────────────────
Tag: devlog-14
───────────────────────────────────────────────────────────────────────────────
Files Update:        <strong>119</strong>
LOC Added:           <strong>1913</strong>
LOC Deleted:         <strong>160</strong>
Code Coverage:       <strong>88%</strong>

───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Lua                        638     25725     3542      2868    19315       1223
Markdown                    21      1098      296         0      802          0
ReStructuredText            13       412      124         0      288          0
Plain Text                   5       658      108         0      550          0
CSV                          4      1175        0         0     1175          0
JSON                         4       153        0         0      153          0
License                      2        42        8         0       34          0
YAML                         2        73        1         0       72          0
Batch                        1        35        8         1       26          5
Makefile                     1        20        4         7        9          0
Python                       1        56       15        29       12          0
Shell                        1        11        1         1        9          0
gitignore                    1        51       10         9       32          0
───────────────────────────────────────────────────────────────────────────────
Total                      694     29509     4117      2915    22477       1228
───────────────────────────────────────────────────────────────────────────────
</pre>

[All the changes](https://github.com/tredfern/terminus/compare/devlog-13...devlog-14)

![Video](https://www.dropbox.com/s/aqw6v4c3egi1m71/terminus_devlog_14.gif?raw=1)