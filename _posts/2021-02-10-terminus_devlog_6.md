---
title: Terminus Devlog 6 - Skills, Inventory, and Items
layout: post
date: 2021-02-10
image: https://www.dropbox.com/s/1w5dqpdg0u8kw2s/postimage-20210210.png?raw=1
excerpt: Adding the framework for skills, inventory and items ![Progress](https://www.dropbox.com/s/lbeaghjgskk3895/terminus_devlog_6.gif?raw=1)
---

[Release Download](https://github.com/tredfern/terminus/releases/tag/poststream-20210207)

> **In this session**
> - First active development on Milestone 1!
> - Ability scores and skills on character
> - Skills Refactoring
> - Character inventory and placeholder items
> - Displaying character items


## Attributes and Skills

For this game, I've selected a foundation of 6 attributes to define a character. This is a fairly standard RPG
format and seems like copying dozens of other RPGs is a safe bet for a foundation. I focused on 3 physical
and 3 mental attributes.

For building this into the game, I started with just placeholder values. It was more important to get them wired
in than to worry about any specific values or generation algorithm.


### Attributes
```lua
-- snip from game/rules/character/helper.lua
function characterHelper.createDefaultAbilities()
  return {
    strength = 10,
    agility = 10,
    endurance = 10,
    wit = 10,
    education = 10,
    social = 10
  }
end

local Components = require "moonpie.ui.components"

local function createLabelPair(label, value)
  return {
    Components.text { text = label },
    Components.text { id = label .. "Stat", text = value, style ="align-right" }
  }
end

-- game/ui/widgets/character_attributes.lua (By end of stream)
return Components("character_attributes", function(props)
  local attributes = require "game.rules.character.attributes"
  local characterAttributes = props.attributes or {}

  return {
    width = "15%",
    createLabelPair("Strength", characterAttributes[attributes.strength]),
    createLabelPair("Agility", characterAttributes[attributes.agility]),
    createLabelPair("Endurance", characterAttributes[attributes.endurance]),
    createLabelPair("Wit", characterAttributes[attributes.wit]),
    createLabelPair("Education", characterAttributes[attributes.education]),
    createLabelPair("Social", characterAttributes[attributes.social]),
  }
end)
```

[Hard coding](https://en.wikipedia.org/wiki/Hard_coding) your values into the program is acceptable when we are
establishing the framework from which the code will run. It makes no sense to figure out whether we should roll dice,
create a point-buy system, or some other generation method, if we haven't first gotten the program executing with
the values. They are two very different concerns in this case.

Especially in this context, since we want skills to be dependent on certain attributes for their basic skill values.

### Skills
```lua
-- game/rules/character/skills.lua
local Skills = {}

function Skills.getScore(skill)
  return skill.character.abilities[skill.ability]
end

function Skills.newSkill(name, character, ability)
  return {
    name = name,
    getScore = Skills.getScore,
    character = character,
    ability = ability
  }
end

return Skills 

-- Snip from game/rules/character/helper.lua
function characterHelper.createDefaultSkills(character)
  local skills = require "game.rules.character.skills"
  return {
    throwing = skills.newSkill("Throwing", character, "agility"),
    blade = skills.newSkill("Blade", character, "agility"),
    unarmed = skills.newSkill("Unarmed", character, "strength"),
    handgun = skills.newSkill("Handgun", character, "agility")
  }
end
```

This first implementation ran into some problems. It did produce a set of skills and it was possible to display data
to the screen that looked like a skills list. But it felt cumbersome and not well designed. 

*How do we add skills to the game?*, *How does a skill can points?*, *Can skills be trained?*

But there is value, we have a system. And from this system we can **refactor** to a better one!

### Skills Refactoring

```lua
local Skills = {}

function Skills:define(props)
  Skills[props.key] = setmetatable(props, {
    __call = Skills.calculate
  })
end

function Skills.calculate(skill, character)
  local untrained = skill.untrained or 0
  return (character.attributes[skill.attribute] or 0) + (character.skills[skill.key] or untrained)
end

local function setupDefault()
  local attributes = require "game.rules.character.attributes"
  Skills:define { name = "Unarmed", key = "unarmed", attribute = attributes.strength }
  Skills:define { name = "Blade", key = "blade", attribute = attributes.agility }
  Skills:define { name = "Handgun", key = "handgun", attribute = attributes.agility }
  Skills:define { name = "Throwing", key = "throwing", attribute = attributes.agility }
end

setupDefault()

return setmetatable(Skills, {
  __call = Skills.define
})


-- Snip from game/rules/character/helper.lua
function characterHelper.createDefaultSkills()
  local skills = require "game.rules.character.skills"
  return {
    throwing = 0,
    blade = 0,
    unarmed = 0,
    handgun = 0,
  }
end
```

So what changed here? First off, we change the way we think about skills. We want to **define** what skills
are available in the game. In the future, these might **describe** certain actions that are available. Items might
link to these skills to define how they are used or interacted with. This separates out the direct link to the
character, other than, what is the attribute that best defines how the skill is used. We also are able to define what 
it means to be **untrained** in a skill! 

Next, we change the character to link to skills based on the key to specify how well trained the character is. A
value of 0 means that the character has a basic level of knowledge in the skill. Additional points improve
the characters ability in the skill. This seems like a clean elegant system that should provide a lot of opportunity
in the future to make some interesting mechanics.


## Inventory and Items

Another key foundational piece that to be introduced was the concept of inventory and items in the game. 
```lua
-- game/rules/character/actions/add_item_to_inventory.lua
local types = require "game.rules.character.actions.types"

return function(character, item)
  return {
    type = types.character_add_item_to_inventory,
    payload = {
      character = character,
      item = item
    }
  }
end 

-- snip from game/rules/character/reducer.lua
  [types.character_add_item_to_inventory] = function(state, action)
    local character = action.payload.character
    character.inventory[#character.inventory + 1] = action.payload.item
    return state
  end,

-- game/rules/items/init.lua
local Items = {}
Items.list = {}

function Items.describe(props)
  Items.list[props.key] = {
    name = props.name
  }
end

Items.describe { key = "sword", name = "Sword" }
Items.describe { key = "healthPack", name = "Health Pack" }
Items.describe { key = "keycard", name = "Keycard" }


return Items 
```

To start building the foundation, the code was kept very light. Just enough to create the **skeleton** of the system.
At this time, it's not worth defining a dozen different attributes for items. Instead, it's just enough to link
some rules area for *Items* to *Characters*.

The action, *AddItemToInventory* is associated with a *Character* because ultimately it is changing the character state
and not the item state. This is a great sign that our code is well laid out because we can be specific about where
state changes are occurring and who is responsible for dealing with these kinds of changes.

```lua
local Items = {}
Items.list = {}

function Items.describe(props)
  Items.list[props.key] = {
    name = props.name,
    skills = props.skills
  }
end

function Items.canUseItem(item, character)
  if item.skills == nil or #item.skills == 0 then return true end

  for _, v in ipairs(item.skills) do
    if character.skills[v] then return true end
  end

  return false
end
```

With a little bit of work, it was possible to add a check to determine whether a character could use an item or not.
This check could be improved. For example, we should like have some mechanism for checking if a character is
trained in a skill, or whether an untrained skill is possible for an item. For example, nearly anybody can swing
a sword but only those actually taught how to use it can do it effectively.

## Data Folder and Final Touches

A small change, but critical step was the creation of the `game/data/` folder. This starts the foundations of
separating out the information that should be easily changeable, and the rules themselves. There is now an
action that is called from `game/rules/game_state/actions/load_core_data.lua` which will set up the core rules.

In the future, there will likely be an action to load mods, etc... that will change the data in the game. This
separation allows that to occur. Currently, `Skills` and `Items` are the two data points loaded from the data
folder and these are mostly dummy values. But they can be modified and changed over time. For example, different
tiles and terrain can be moved here, different monsters, character types.... this opens a lot of possibilities for
the future.

## Final Stats
- **44** Files Update
- **674** Additions
- **106** Deletions
- **98.26%** [Code Coverage](https://coveralls.io/builds/36934442)

[All the changes](https://github.com/tredfern/terminus/compare/milestone-0...poststream-20210207)

![Progress](https://www.dropbox.com/s/lbeaghjgskk3895/terminus_devlog_6.gif?raw=1)