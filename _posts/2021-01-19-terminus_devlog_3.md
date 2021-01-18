---
title: January 16-17 Session Review
layout: post
date:   2021-01-19
excerpt: Setting up the map, camera and some initial stats for characters. Plus simple refactoring adventures.
---

[![Release](https://img.shields.io/badge/Download-20210117-informational)](https://github.com/tredfern/terminus/releases/tag/poststream-20210117)

> **In this [session](https://github.com/tredfern/terminus/blob/main/docs/sessionnotes/20210116.md)**
> - Refactoring
> - Making a map
>   - Adding boundary checks to character movement
> - A basic camera
>   - Centering and tracking the player
> - Adding health to characters with damage and removal
> - Fixing little bugs!

## Refactoring
### [Changes](https://github.com/tredfern/terminus/commit/89cea461ed69972057dce4616b8763a547b8e255)

1. We added a **game_state** ruleset under `game/rules`. 
2. We created a new action and **test harness** for game setup
3. We transitioned identical logic to the new action
4. We called the new action from `app.lua` and removed the old function

### Summary
Before we could make more advanced logic for setting up games, we needed to move the logic out of the `game/app.lua` file.
This file is meant to connect the various game states and hold the application together, 
not have any specific logic.

```lua
-- Function that we need to remove
local function set_up_the_game()
  local character = require "game.rules.character"

  store.dispatch(character.actions.add(character.create { x = 2, y = 1, is_player_controlled = true }))
  for _=1,4 do
    store.dispatch(
      character.actions.add(
        character.create { is_enemy = true, x = math.random(10), y = math.random(10) }))
  end
end
```

This function made sure there was a player and added a couple of enemies. Nothing too fancy, but it was in the wrong
location and before adding in logic for map generation, it made sense to move.

The [structure](/terminus#structure) of our game suggest we should have _rule_ and _action_ for setting up the game. 
Think of a board game, and usually there is a section of the rulebook with a big heading **Set Up**. 
Set up will likely be evolving, could be dependent on what settings a player would like to set, etc... 
This allows us a clean entry point to initiate the configuration of our game. And, we can test it!


```lua
-- New action in game/rules/game_state/actions/setup.lua
return function()
  return function(dispatch)
    dispatch(character.actions.add(
      -- The values we are inputting here are placeholders for the future
      character.create {
        x = 2,
        y = 1,
        is_player_controlled = true
      }
    ))

    for _=1,4 do
      dispatch(character.actions.add(
        character.create {
          is_enemy = true,
          x = math.random(10),
          y = math.random(10)
        }
      ))
    end
  end
end 
```

## Map
### [Changes](https://github.com/tredfern/terminus/compare/89cea4...8cc004)

1. [Create new map ruleset](https://github.com/tredfern/terminus/commit/46eed4) and added to setup rules
2. [Update UI](https://github.com/tredfern/terminus/commit/fc14b0) to use the map to render
3. [Added random generated terrains](https://github.com/tredfern/terminus/commit/3059ba) for dirt, water and grass
4. [Limit character movement](https://github.com/tredfern/terminus/commit/8cc004) to remain in map boundaries

With the update to how we set up the game, we can start to consider adding a new ruleset around _maps_. 
This ruleset would contain any map specific information in a coordinate-based fashion. For example, what
is the terrain in a specific square. 

#### Create the map
```lua
-- game/rules/map/map.lua
local class = require "moonpie.class"

local map = {}

function map:constructor(props)
  self.width = props.width
  self.height = props.height
end


return class(map) 

-- game/rules/map/actions/set.lua
local types = require "game.rules.map.actions.types"

return function(map)
  return {
    type = types.set,
    payload = map
  }
end 

-- game/rules/map/reducer.lua
local create_slice = require "moonpie.redux.create_slice"
local types = require "game.rules.map.actions.types"

return create_slice {
  [types.set] = function(_, action)
    return action.payload
  end
} 
```

The first block of code describes the map entity. We consider it a `class` that can be instantiated and passed
some properties. Currently, width and height are the only ones supported.

The second block describes the only action we needed to get maps off the ground and that is to set the map.
The current implementation plans that we would only have one map at a time. In the future, this could and probably
will change. We might have multiple maps in state and swap between them. For now, we start with a single map 
at the time implementation first.

The final block is a good chance to look at the `reducer` framework. We use the `create_slice` helper provided 
by [Moonpie](/moonpie/) to configure the reducer and define who what actions we want to handle in this state. 
A table is provied with various action type values as the key, we can add a handler for what to do at this state time. 
This particular action is simple, whenever you set the map, replace the state with the current map.

When using `create_slice`, the state is considered scoped to just this slice of the state. Updating these state values should
not impact the state managed by a different part of the state. This provides us with isolation in our state management
and should reduce the potential for unexpected bugs.

#### Rendering and Validating

The rendering and terrain logic is not terrible complex. We create a list of terrain values to use and randomly
input them into our grid. When rendering we ask for the color of the rectangle we'd like to render out.

```lua
-- snippet of game/rules/character/actions/set_position.lua
    validate = function(self, state)
      local dims = map.selectors.get_dimensions(state)
      return self.payload.x >= 1 and self.payload.x < dims.width and
        self.payload.y >= 1 and self.payload.y < dims.height
    end
```

Simple actions support adding a `validate` method to the table. This is called before passing to the reducer. If
an action determines it is invalid, it will be skipped. It makes a handy way of preventing state changes from
occurring that we want to prevent and localizes them to the action. We can just validate that the x and y values
are within the range we expect.

## A Basic Camera

### [Changes](https://github.com/tredfern/terminus/compare/8cc004...40778e)
1. Created new camera ruleset and it [follows player](https://github.com/tredfern/terminus/commit/75a62fc)
2. [Centered](https://github.com/tredfern/terminus/commit/40778ede) camera around the player

With a more interesting map that is larger than our ability to see on the screen at any on time, we need
to introduce some mechanism to figure out which part of the map to display.

```lua
-- game/rules/camera/actions/set_position.lua
local types = require "game.rules.camera.actions.types"

return function(x, y)
  return {
    type = types.camera_set_position,
    payload = {
      x = x,
      y = y
    }
  }
end 

-- snippet of game/ui/widgets/combat_map.lua
  draw_component = function(self)
    for x = 1, self.map.width do
      for y = 1, self.map.height do
        draw_tile(
          x - self.camera.x, 
          y - self.camera.y, 
          self.map:get_terrain(x, y).color)
      end
    end


    for _, v in ipairs(self.characters) do
      draw_character(
        v.x - self.camera.x, 
        v.y - self.camera.y, 
        v.is_enemy)
    end
  end
```

We provide an action that allows us to move the camera around. The main adjustment was to offset all the drawing
by the camera location. Notice that this implementation is very simple. We just want something that will allow
us to move what we are rendering.

For centering the character in the camera, we need to figure out what is the width and height of the screen in tiles.
That logic is determined by the UI. We created a new action to handle updating the dimensions of the camera.

```lua
-- game/rules/camera/actions/set_dimensions.lua
local types = require "game.rules.camera.actions.types"

return function(width, height)
  return {
    type = types.camera_set_dimensions,
    payload = {
      width = width,
      height = height
    }
  }
end
```

## Character Health
### [Changes](https://github.com/tredfern/terminus/compare/40778e...4cb815)
1. Characters [have health](https://github.com/tredfern/terminus/commit/4cb815) and are removed at 0


```lua
-- game/rules/character/actions/set_health.lua
local types = require "game.rules.character.actions.types"

return function(character, health)
  return {
    type = types.character_set_health,
    payload = {
      character = character,
      health = health
    }
  }
end 

-- snippet game/rules/character/actions/attack.lua
    dispatch(set_health(target, target.health - 1))

-- game/rules/character/selectors/get_dead.lua
local tables = require "moonpie.tables"

return function(state)
  return tables.select(state.characters, function(c)
    return c.health <= 0
  end)
end 

-- Snippet of game/rules/turn/actions/process.lua
    -- Check for dead characters
    local dead = character.selectors.get_dead(get_state())
    if dead then
      for _, e in ipairs(dead) do
        dispatch(character.actions.remove(e))
      end
    end
```

The first basic step for health was just adding a value to `game/rules/character/character.lua` to set a property
for health to a default value (10). Where things start getting interesting is adding an action that allows
us to set the health to a different value, combined with a selector that returns all the entities that have
health less than or equal to 0. 

Within `game/rules/character/actions/attack.lua` we changed a line to dispatch a new health value that is 1 lower
than it was before.

Finally, each time we process the turn, we search for any dead characters and then remove them.

The great thing about this approach is we changed behavior with little impact on the rest of the system and
still were able to create the desired effect. From this we are allowing ourselves the opportunity to create
new actions that deal damage in various ways to reduce health. In the future we will likely need to create an
action for dying that will be different from just removing to allow for more complex steps. Such as, dropping
loot on the ground, add experience points or something to the player, these kinds of things. Again, the overall
impact to the system adding these changes should be minimal.

## Fixing Bugs
1. Character would [attack themselves](https://github.com/tredfern/terminus/commit/4dea045)

This was a minor bug, but still a funny one. When characters would select their same square to move into, they
would see themselves as moving into a character and then issue an attack command. There were lots of ways to solve this, 
but the most logical was that one should not be able to dispatch the attack actions to themselves.

Because attacks are complex actions, it creates an interesting decision about what to do. Should we error? Should
we add logic within the system to handle this case? The approach I went with is that the action will return, essentially 
an empty action. This would then just pass through the system with no side-effects. It's less ideal because it 
adds a tiny bit of work, but it also is low risk.

The key point is that we created a test for this scenario first and proved it was failing and then fixed the error.

```lua
-- snippet from game/rules/character/actions/attack_spec.lua
  it("returns empty action if source and target", function()
    local source = {}
    local target = source

    assert.same({}, attack(source, target))
  end)
```

#### Final Stats
- **56** Files Update
- **897** Additions
- **120** Deletions
- **(97.59%)** Code Coverage

[All the changes](https://github.com/tredfern/terminus/compare/c64078...4cb815)

![Screenshot](https://www.dropbox.com/s/m7yo1xbv4o8gr1k/Screenshot%202021-01-17%20211714.png?raw=1)
*End of stream screenshot shows the magenta "player" character in the middle, the red enemies running around, and the basic health statistics displayed on the side.*

