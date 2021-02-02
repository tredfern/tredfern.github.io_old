---
title: Devlog Update 5 (Jan 30-31 Stream)
layout: post
date:   2021-02-03
image: https://www.dropbox.com/s/pzws4fngvcecai1/postimage-20210203.png?raw=1
excerpt: Displaying a message log and performing combat rolls
---

> **In this [session](https://github.com/tredfern/terminus/blob/main/docs/sessionnotes/20210130.md)**
> - Adding a message log to the user interface
> - Creating messages when certain actions occur
> - Combat "System" - Not advanced but adding some randomness
> - Offline: Skipped stream on Sunday for major refactor


# Adding a Message Log to the UI

In Rogue-like games, a common factor is some sort of message log giving updates to the player about what
is happening behind the systems. Rogue-likes usually contain some complex mechanics and the text display
is usually the key insight into what is actually occurring. This could be information about combat rolls,
extra details about what is happening on the screen, or other events that are occurring.


## Managing the State of the Message Log

The first step was to add a new _rules_ area to manage the message log. This would own a slice of the state. 
The responsibilities of this state management are adding new entries and providing access to the existing
entries to any consumers that are interested.

```lua
-- game/rules/message_log/init.lua
-- Defines the API for using the message log center
return {
  actions = {
    add = require "game.rules.message_log.actions.add"
  },
  selectors = {
    get_last = require "game.rules.message_log.selectors.get_last"
  }
} 

-- game/rules/message_log/reducer.lua
-- Defines how state is stored
local create_slice = require "moonpie.redux.create_slice"
local types = require "game.rules.message_log.actions.types"


return create_slice {
  [types.message_log_add] = function(state, action)
    state = state or {}
    state[#state + 1] = action.payload
    return state
  end
} 


-- game/rules/message_log/actions/add.lua
-- An action message that defines a new message to add to state
local types = require "game.rules.message_log.actions.types"

return function(message)
  return {
    type = types.message_log_add,
    payload = {
      message = message
    }
  }
end 

-- game/rules/message_log/selectors/get_last.lua
-- Retrieves the last XX number of messages from state
return function(state, count)
  if not state.message_log then return {} end

  local out = {}
  for i = #state.message_log, #state.message_log - count + 1, -1 do
    out[#out + 1] = state.message_log[i]
  end
  return out
end 
```

The code here is a great example of how to define a new state slice and perform some basic operations in 
[Moonpie](https://github.com/tredfern/moonpie). We set up a rules area and define the API we want consumers
to use it. There are actions that can be dispatched, and selectors that can retrieve appropriate state.

The reducer defines how we make changes to the state. It's based on the actions `type`. That `type` provides
a direct table lookup to which block of code to run. (`[types.message_log_add] = function(state, action)`)

Once we are able to add entries to the log, we are going to want to get them back out again. The selector, `get_last`
provides a simple API for getting the last XX number of entries. 


## Displaying the Message Log

The _Message Log Widget_ is an example of how this message log is consumed. 

```lua
-- game/ui/widgets/message_log.lua
local components = require "moonpie.ui.components"
local connect = require "moonpie.redux.connect"
local message_log = require "game.rules.message_log"
local tables = require "moonpie.tables"

local message_log_widget = components("message_log", function(props)
  return {
    messages = props.messages,
    render = function(self)
      return tables.map(self.messages, function(msg)
        return {
          components.text { text = msg.message }
        }
      end)
    end
  }
end)


return connect(message_log_widget, function(state)
  return {
    messages = message_log.selectors.get_last(state, 5),
  }
end) 
```

In [Moonpie](https://github.com/tredfern/moonpie) we have the ability to connect our UI components to the state.
Whenever state changes occur, it will call the connect routine and if those changes are important to the UI the
UI will update. _There might be some performance optimizations necessary in the future when lots of state changes
are occurring. However, these are deferred until I can see that such changes are necessary and what makes sense._

The `render` routine is called whenever the component updates, that is, when a new message is added. It will
loop through all the messages and return a new text display for each message. 

With a component defined, we can now `connect` this component to the store to monitor any state changes. The `connect`
routine provides a function to pull any state changes. We use our `get_last` routine to get the last 5 entries from
state.

To display the message log, we need to add it to the screen where this is visible

```lua
local message_log = require "game.ui.widgets.message_log"

-- Snip from game/ui/screens/combat.lua
local combat_screen = components("combat_screen", function()
  return {
    id = "combat_screen",
    {
      style = "main_screen",
      map_component(),
    }, {
      style = "stats",
      components.h3 { text = "Stats" },
      {
        style = "stats_content",
        turn_counter(),
        character_stats()
      },
      {
        components.h3 { text = "Messages" },
        message_log() -- << Added Message Log >>
      }
    },
```

We now have everything wired up for messages to be created, stored, retrieved and displayed.

## Adding Messages

```lua
-- game/rules/character/actions/attack.lua
local set_health = require "game.rules.character.actions.set_health"
local types = require "game.rules.character.actions.types"
local message_log = require "game.rules.message_log"

return function(source, target)
  if source == target then return {} end

  return setmetatable({
    type = types.character_attack,
  }, {
    __call = function(_, dispatch)
      dispatch(set_health(target, target.health - 1))

      -- Message Log Entry
      local src = source.name or tostring(source)
      local trg = target.name or tostring(target)
      local str = src .. " attacked " .. trg
      dispatch(message_log.actions.add(str))
    end
  })
end
```

We want to track whenever a character attacks another character to display a message. By going to the attack action
we can add in the message_log API and dispatch a message about the attack that is occurring. 

![Message Log](https://www.dropbox.com/s/pjp7cat4rbfzkar/message_log_example_20210202.png?raw=1)

_An interesting discovery occurred from this that I was not aware of before. And that is that enemies will attack
each other!_


## Combat System

With messages being displayed, we can look at what we are doing for the attacks and see if we could make something
more complex. This isn't to define _the_ combat system but to create _a_ combat system.

The first step, was actually to **refactor** the code. Currently, attacks were handled within `game/rules/character`.
This doesn't make sense. What if there are attacks that are not by characters? Should those be in a different section?
What if the general rules are the same? Like a trap that shoots a dart? How do we consolidate these rules together?

So the first step, was a modest refactor that created a `game/rules/combat/` center for managing all the combat rules.
An interesting thing here, is that we don't (at this time) have anything specific for state in the combat section.
Instead, we are focused on bringing the rules necessary for combat to operate together in a single area.

> When ever refactoring, always focus on __maintain__ your current level of functionality, and not adding the
> new exciting pieces that are causing you to want to refactor. This prevents long refactors that become
> confusing. Once you have stabilized the code base after the refactor, you are free to add the new
> awesomeness.

```lua
-- snip from game/rules/combat/actions/attack_spec.lua
  it("rolls to resolve the attack", function()
    local helper = require "game.rules.combat.helper"
    spy.on(helper, "resolve_attack")

    local source = { attack = 100 }
    local target = { defense = 0, health = 8 }
    local atk = attack(source, target)
    atk(mock_dispatch)

    assert.spy(helper.resolve_attack).was.called_with(100, 0)
  end)

-- game/rules/combat/helper_spec.lua
  describe("game.rules.combat.helper", function()
  local helper = require "game.rules.combat.helper"
  local mock_random = require "moonpie.test_helpers.mock_random"

  describe("attack rolls", function()
    it("if attack succeeds and defense fails, return true", function()
      mock_random.setreturnvalues { 25, 58 }
      assert.is_true(helper.resolve_attack(50, 50))
    end)

    it("returns false if attack misses", function()
      mock_random.setreturnvalues { 58, 58 }
      assert.is_false(helper.resolve_attack(50, 50))
    end)

    it("returns false if defend succeeds", function()
      mock_random.setreturnvalues { 58, 48 }
      assert.is_false(helper.resolve_attack(50, 50))
    end)

    it("returns the die roles", function()
      mock_random.setreturnvalues { 20, 30 }
      local success, atk, def = helper.resolve_attack(50, 50)
      assert.is_false(success)
      assert.equals(20, atk)
      assert.equals(30, def)
    end)
  end)
end) 
```

I want to show a small snip of the `spec` for the combat system. We were going to end up with some complexity
of random numbers introduced into our spec. Random numbers always introduced complexity into any TDD framework.

First off, we want to isolate the random number routines and make that code as specific as possible. The less
code surrounding the random number generators the better. For this end, we created a `helper` that can hold these
specific routines. 

Within the action, we just make sure you are passing the appropriate values into the helper. If you do that, then
we can make sure that the logic within the action is occurring properly.

For the helper, we mock out the random number generator and specify what values we expect to get back in certain
circumstances. This allows us to control the random number generator and make sure that when certain value ranges
occur, we know the outcomes from the test.

In our case, we want an attack to only hit if the Attack Roll succeeds and the Defense Roll fails. 

## Sunday Stream - Cancelled for Refactor

I skipped streaming on Sunday and instead focused on rewriting the code to use camelCase instead of snake_case. 
I like to flip back and forth between different styles and patterns, so this is kind of a thing I do. (And how I 
ended up in snake_case in the first place.) After giving it some thought though I figured camelCase would be
more accessible to other developers so I'm following that pattern going forward. The Moonpie library has also
been updated to use camelCase.


#### Final Stats
- **96** Files Update
- **1121** Additions
- **378** Deletions
- **97.86%** Code Coverage (increased)

[All the changes](https://github.com/tredfern/terminus/compare/poststream-20210124...poststream-20210131)

[Progress](https://www.dropbox.com/s/ipwl3gxs3h5yts2/terminus_20210202.gif?raw=1)