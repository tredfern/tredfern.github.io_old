---
title: Terminus Devlog Week 13
layout: post
image: https://www.dropbox.com/s/o97apoej04bs7se/postimage-devlog13.png?raw=1
excerpt: Multi-Level Dungeons and Animations! ![Video](https://www.dropbox.com/s/9kr0kwt7mjmd4bv/terminus_devlog13.gif?raw=1)
---

> ## Items Completed
> - [#75 - Graphics can animate](https://github.com/tredfern/terminus/issues/75)
> - [#13 - Ladders for dungeons](https://github.com/tredfern/terminus/issues/13)

## Big foundational changes

While only 2 issues total closed, the amount of work and impact of these changes is significant. It's taking time
to rework the infrastructure of the code to support what is needed for a more advanced game, but that work 
is starting to pay off. 

## #75 Animations

Animations are obvious for what you want and difficult to design a good API around. The challenge is what information
is necessary at the right level. For this implementation I've broken the animations into 3 different components.

Sprite
:  Sprites represent the basic image to draw. These know about how to render to the screen. They also understand how 
  to divide up spritesheets into quads to render a specific element in the sheet.

Animation
: Animations represent a specific animation to be played. Animations are made up of frames that have a Sprite and
  a time-delay for how long the frame should take to display.

Animator
:  Animators will hold multiple animations and can be told to play a specific animation. They keep track of 
  what frame a particular instance should be on.

```lua
--
-- Animation example from data/characters/character_idle.lua
--
local animation = require "game.graphics.animation"
local sprite = require "game.graphics.sprite"
local imageMgr = require "moonpie.graphics.image"

local imageData = imageMgr.load("data/characters/character_idle.png")
local frame1 = sprite.fromAtlas(imageData, 0, 0, 32, 32)
local frame2 = sprite.fromAtlas(imageData, 32, 0, 32, 32)
local frame3 = sprite.fromAtlas(imageData, 64, 0, 32, 32)
local frame4 = sprite.fromAtlas(imageData, 96, 0, 32, 32)

local character_idle = animation:new()
character_idle:addFrame(frame1, 0.3)
character_idle:addFrame(frame2, 0.3)
character_idle:addFrame(frame3, 0.3)
character_idle:addFrame(frame4, 0.3)

return character_idle 
```
```lua
--
-- Example of adding an animator to an entity (game/rules/player/actions/add)
--
  local c = characters.create { x = x, y = y, isPlayerControlled = true }
  c.animator = Animator:new()
  c.animator:addAnimation("idle", characterIdle)
  c.animator:play("idle")
```

The only animation in play right now is just an idle animation for the character. Adding more will obviously take more time on the sprite side and that's for a later point to invest more heavily into. 

## #13 Multilevel Maps (Ladders)

There were a few options to explore to create multilevel maps.

Generate a new map when going to a new level
: This had the advantage that x,y for ladders could be in different spots. The downside was that each level doesn't relate to each other.
: It was also a more simple implementation for generating maps as each map could just be made at the time that the character switched levels.

Generate maps in a 3-dimensional map
: The option I ended up going with. By adding a 3rd dimension to the coordinate system, it's possible to create maps that have multiple levels. This has a lot of interesting benefits for the game.
: Levels could be designed where there are sections closed off but require going up/down to another level to enter. Levels could have very different designs but interact with each other. For example, air ducts are in every Alien space survival movie. This allows create an "air duct" level that the character can move through and drop down into different rooms. That's kind of cool.

```lua
local function compare(start, dest)
  if dest.x then
    return start.x == dest.x and
      start.y == dest.y and
      start.z == dest.z
  else
    return start.x == dest[1] and
      start.y == dest[2] and
      start.z == dest[3]
  end
end

local function new(x, y, z)
  return {
    x = x or 0,
    y = y or 0,
    z = z or 0
  }
end

local function copy(position)
  return new(
    position.x,
    position.y,
    position.z
  )
end

local function add(position, delta)
  local x = delta[1] or 0
  local y = delta[2] or 0
  local z = delta[3] or 0

  return new(
    position.x + x,
    position.y + y,
    position.z + z
  )
end

local function northwest(position)
  return add(position, { -1, -1, 0 })
end

local function north(position)
  return add(position, { 0, -1, 0 })
end

local function northeast(position)
  return add(position, { 1, -1, 0 })
end

local function west(position)
  return add(position, { -1, 0, 0 })
end

local function east(position)
  return add(position, { 1, 0, 0 })
end

local function southwest(position)
  return add(position, { -1, 1, 0 })
end

local function south(position)
  return add(position, { 0, 1, 0 })
end

local function southeast(position)
  return add(position, { 1, 1, 0 })
end

local function up(position)
  return add(position, { 0, 0, 1 })
end

local function down(position)
  return add(position, { 0, 0, -1 })
end

return setmetatable({
  add = add,
  copy = copy,
  new = new,
  equal = compare,
  northwest = northwest,
  north = north,
  northeast = northeast,
  west = west,
  east = east,
  southwest = southwest,
  south = south,
  southeast = southeast,
  up = up,
  down = down
}, {
  __call = function(_, x, y, z)
    return new(x, y, z)
  end
}) 
```

The data model is simplistic { x , y , z }. But the helpers organized around this allow the map and movement systems to be able to figure out relative locations without having to do the math directly. Isolating this will keep details hidden. In the future it will help with maybe rendering multiple levels on the screen at the same time, etc... 

This change of removing `x` and `y` attributes from entities and replacing with `position` took the majority of the effort. It was a large scale refactor that touched many points. Once that was completed, it became possible to implement the changes to the dungeon generation.

### Dungeon Generation

The `game/rules/map/tile_map.lua` implementation saw the next biggest changes. Adding a z level index to the map. Also, all calls to the TileMap accept a `Position` argument instead of `x`, `y`, `z` directly. Simplifies what is passed in, but also standardizes the interface across the application.

The ``game/rules/map/outline.lua` saw limited change. Each room and corridor keeps track of what level it's on, but otherwise the details didn't need to change much.

The dungeon generation then performs a loop building each level. When it gets a level completed it attempts to connect the level to a previous level. That is done with a basic random/looping mechanism that selects a room and tries to find a corresponding overlapping room on the previous level. Because the area for the map is significantly filled in, there is limited risk of a level not being able to find at least 1 room that overlaps.

Ladders are entities in the game. A new adjustment here was adding them as `features` to a room.

## Next up

I'd like to explore adding some descriptions and more variance for rooms to give a bit more character to
the maps. Maybe some rooms with different shapes as well.

## Final Stats
- **109** Files Update
- **1354** Additions
- **293** Deletions
- **87%** [Code Coverage](https://coveralls.io/builds/39066031)

[All the changes](https://github.com/tredfern/terminus/compare/devlog-12...devlog-13)

![Progress](https://www.dropbox.com/s/9kr0kwt7mjmd4bv/terminus_devlog13.gif?raw=1)
