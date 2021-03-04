---
title: Terminus Devlog Week 9
layout: post
image: https://www.dropbox.com/s/ojswr9rzcp7n13u/postimage-20210304.png?raw=1
excerpt: Winning condition, refactors, better inventory ![Video](https://www.dropbox.com/s/u5blqwpieetdurm/terminus_devlog_9.gif?raw=1)
---

> ## Items Completed
> - [#58 - Show indicator of enemy damage](https://github.com/tredfern/terminus/issues/58)
> - [#28 - Add quantity for inventory items](https://github.com/tredfern/terminus/issues/28)
> - [#59 - Add a way to remove spawners](https://github.com/tredfern/terminus/issues/59)
> - [#74 - BUG: Items showing up in walls](https://github.com/tredfern/terminus/issues/74)
> - [#60 - Victory screen if all spawners are eliminated](https://github.com/tredfern/terminus/issues/60)

## Overview

As I approach the end of milestone 1, which is a very simple and basic game some insights are surfacing.
- The code design is flexible, but requires some reorganizing
- Turning the general mechanics into a game requires some significant design work
- Graphics need work sooner than later to start to bring game to life
- Is a spaceship the correct setting, or would a planet be a better environment?

## Better Debugging

One small change that isn't listed here is the ability to dump out the entire game state to a file <F10>. This helps
with debugging. It also helps with looking at the game state and making sure it makes sense.

```lua
-- First few lines of a dump file
{
  camera = {
    height = 24,
    width = 24,
    x = 11,
    y = 33
  },
  characters = { {
      attributes = {
        AGILITY = 9,
        EDUCATION = 7,
        ENDURANCE = 9,
        SOCIAL = 9,
        STRENGTH = 13,
        WIT = 9
      },
      health = 19,
      inventory = { {...
``` 

Viewing the state has highlighted some opportunities for improvement. If I am thinking about state as a place to store
and link information together, it would be better to have keys for data and group similar data types together.
Thinking like a database for example.

Some thoughts for the future:
- Position data or location makes sense to centralize. This would make rendering easier, as we could go to each position
  and find what to render as opposed to pulling positions from lots of states
- Graphics/Animation data will likely need it's own state management
- Inventory could be pulled out. Each character might have inventory, but what about chests, and things like that?

Also, saving and loading broke somewhere along the way. Cleaning up state should simplify that.

## Inventory Management

```lua
-- game/rules/character/reducer.lua
local function createInventorySlot(item)
  return {
    item = item,
    quantity = 1
  }
end

local function findItemInInventory(inventory, item)
  return tables.findFirst(inventory, function(i) return i.item.key == item.key end)
end

...

-- handling add action
 [types.character_add_item_to_inventory] = function(state, action)
    local character = action.payload.character
    local slot = findItemInInventory(character.inventory, action.payload.item)
    if slot then
      slot.quantity = slot.quantity + 1
    else
      character.inventory[#character.inventory + 1] = createInventorySlot(action.payload.item)
    end
    return state
  end,

```

The key for inventory management was changing how it was stored in `state`. Modifying the state to organize
inventory by the type of item, and then create new slots when an item is added. 

When removing items from the inventory, we check if it is zero which removes the entire slot. Some consideration
should be give to whether all inventory items should be stackable, and if so what is the stackable limit. This again
is a good example of where moving inventory management to a separate reducer makes sense. If I have a quiver of arrows
do I stack more arrows in a quiver than if they are just loose? The quiver than is one item, that can then handle
stacking items in it. But should only handle certain item types, etc...


## Removing spawners from the map

```lua
-- data/items.lua
Items.describe {
  key = "sprayBottle",
  name = "Anti-Spawn Spray",
  consumeOnUse = false,
  useHandler = function(_, dispatch, user)
    local Map = require "game.rules.map"
    local store = require "game.store"
    local spawner = Map.selectors.getEnemySpawnerAt(store.getState(), user.x, user.y)
    if spawner then
      dispatch(Map.actions.removeEnemySpawner(spawner))
    end
  end,
  image = Image.load("assets/graphics/spray-bottle.png")
} 
```

This was a simple but important change. It also highlights some challenges in how much logic is necessary within
each item and whether those can be effectively tested. 

This new item has a handler that checks if there is a spawner at our current location and then removes it. Overall,
this is a good foundation that shows items interacting within context of the position of the character.

Opportunities are to consider context awareness for items. How to provide queues to user that certain actions are
available? How might items work differently in different circumstances? Maybe items can work together at different times?


## Game Over
Another change was to add a victory condition of removing all the spawners. This now has a check each turn process
to evaluate whether the game over state should trigger. Because turn processing is going to trigger so many
updates and checks in the future, it makes sense to change to an event driven methodology. This would allow
anything to process based on turns happening. (For example, what about an item that auto-heals the character, or
an injured state that causes the player to take damage each turn).


## Milestone 1 nearing completion
My goal is to finish milestone 1 this weekend. While not a fun game yet, there is enough going on that the next few
milestones could be reorganized to drive the game to a more interesting state more quickly. Better graphics and UI
certainly show up as opportunities right now, as well as a more engaging game situation.

## Final Stats
- **43** Files Update
- **424** Additions
- **59** Deletions
- **85.11%** [Code Coverage](https://coveralls.io/builds/37579247)

_Code coverage has an error where it's reporting on missing lines from external libraries_
_that started a few builds ago. This is the code coverage from the game folder._

[All the changes](https://github.com/tredfern/terminus/compare/20210215...devlog-8)

![Progress](https://www.dropbox.com/s/up279nlfg726e5o/terminus_devlog_8.gif?raw=1)