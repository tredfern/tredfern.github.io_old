---
title: Summary of January 9-10
layout: post
date:   2021-01-10
---

> **In this session...**
> - Adding turns and turn counter
> - Basic Enemy AI
> - Entering a character name / basic UI work

## Introducing Turns

The first major [commit](https://github.com/tredfern/terminus/commit/cfc16704532119046d3cdb48026b1466bbb9be1a) introduced
turns into the game. 

```lua
-- game/rules/turn/actions/process.lua
return function(player_action)
  return function(dispatch)
    dispatch(player_action)
  end
end 
```

Not a complicated piece of logic, we create a new action (process a turn) that dispatches the player's action. We tested out that this could be extensible by adding a [turn counter](https://github.com/tredfern/terminus/commit/d3cbccacd971b1d997e3c85ccc6251e8dc0eb25a) to the action.

```lua
-- game/rules/turn/actions/process.lua
local increment = require "game.rules.turn.actions.increment"
return function(player_action)
  return function(dispatch)
    dispatch(increment())
    dispatch(player_action)
  end
end 
```

## Enemies moving

Next, we [added the ability for enemies](https://github.com/tredfern/terminus/commit/041d04270a4e242eaee34fa429de4d246c68df4f) to move around. They just move a random direction, but it's a start for some more complex behaviors.

```lua
-- game/rules/turn/actions/process.lua
local increment = require "game.rules.turn.actions.increment"
local character = require "game.rules.character"
local enemy = require "game.rules.enemy"

return function(player_action)
  return function(dispatch, get_state)
    dispatch(increment())
    dispatch(player_action)

    local enemies = character.selectors.get_enemies(get_state())
    if enemies then
      for _, e in ipairs(enemies) do
        dispatch(enemy.actions.think(e))
      end
    end
  end
end

-- game/rules/enemy/actions/think.lua
local character = require "game.rules.character"

return function(enemy)
  return function(dispatch)
    local randx = math.random(-1, 1)
    local randy = math.random(-1, 1)

    dispatch(character.actions.move(
      enemy,
      enemy.x + randx,
      enemy.y + randy
    ))
  end
end 
```

## Sunday

This was a more basic day where the focus was to introduce some new [UI Elements](https://github.com/tredfern/terminus/commit/bc8cf9732badac4c48556776a2a485081a484d11)

```lua
-- game/ui/screens/create_character.lua
local components = require "moonpie.ui.components"
local store = require "moonpie.redux.store"
local connect = require "moonpie.redux.connect"
local app = require "game.app"
local character = require "game.rules.character"
local panel = require "game.ui.widgets.panel"

local create_character = components("create_character", function(props)

  local edit_character = props.character
  local character_name = components.textbox {
    id = "character_name",
    click = function(self) self:set_focus() end
  }
  character_name:set_text("Papageno")

  return {
    id = "create_character_screen",
    panel {
      width = "50%",
      style = "align-middle align-center",
      title = "Create Character",
      contents = {
        character_name,
        components.button {
          id = "button_done",
          caption = "Done",
          click = function()
            store.dispatch(character.actions.set_name(
              edit_character,
              character_name:get_text()
            ))
            app.combat()
          end
        },
      }
    }
  }
end)

return connect(create_character, function(state)
  return {
    character = character.selectors.get_player(state)
  }
end) 
```