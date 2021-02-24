---
title: Terminus Devlog Week 8
layout: post
image: https://www.dropbox.com/s/a23pp0kkw1i6vzu/postimage-20210224.png?raw=1
excerpt: More placeholder images, music, and additional AI! ![Video](https://www.dropbox.com/s/up279nlfg726e5o/terminus_devlog_8.gif?raw=1)
---

> ## Items Completed
> - [#26 - A little flashiness on the title screen](https://github.com/tredfern/terminus/issues/26)
> - [#30 - graphics for characters/monsters](https://github.com/tredfern/terminus/issues/30)
> - [#11 - AI that moves towards player](https://github.com/tredfern/terminus/issues/11)
> - [#65 - Items can be "cloned"](https://github.com/tredfern/terminus/issues/65)
> - [#29 - Items can be picked up on the map](https://github.com/tredfern/terminus/issues/29)
> - [#24 - Music to title screen](https://github.com/tredfern/terminus/issues/24)
> - [#31 - graphic for health kit/items](https://github.com/tredfern/terminus/issues/31)
> - [#64 - Items are consumed on use](https://github.com/tredfern/terminus/issues/64)
> - [#66 - Terrain is defined in data folder](https://github.com/tredfern/terminus/issues/66)


## Graphics, Flashiness, and Music

At an early stage, I don't want to make things too polished. It's too early and many things could change. But I did
want to start giving more character to the game. Adding some more visual ideas will help guide the project to
having the kind of character and theme that I want.

```lua
-- 
local starField = love.graphics.newParticleSystem(Image.load("assets/graphics/particles/circle.png"))
  starField:setParticleLifetime(2, 5) -- Particles live at least 2s and at most 5s.
	starField:setEmissionRate(25)
  starField:setSizes(0.05, 0.1)
	starField:setSizeVariation(0.5)
	starField:setLinearAcceleration(-220, -220, 220, 220) -- Random movement in all directions.
	starField:setColors(c[1], c[2], c[3], 1, c[1], c[2], c[3], 0)
  starField:setEmissionArea("borderrectangle", 50, 50)
  starField:setTangentialAcceleration(10, 100)

events.beforeUpdate:add(function()
  local dt = love.timer.getDelta()
  starField:update(dt)
end)
```

Nothing fancy here, but if you are curious about setting up a particle system in Love2D. This just gives some moving
bits on the title screen. It's not quite to starfield settings but it feels ok. Definitely an improvement. I also 
added a few images this release for the character, the aliens, and health packs on the ground. While simple
they just add a bit of character.

Most importantly, it highlights the work necessary to get graphics rendering properly. Currently, in
`game/ui/widgets/combat_map.lua` there is a cluster of functions and routines to draw the objects and it
is not easily extendable. I've added [Issue 71](https://github.com/tredfern/terminus/issues/71) to track
work necessary to improve this.

### Music

I also took some time to put together a [brief track](https://soundcloud.com/trevorredfern/terminus-game-title-track-placeholder) for the title screen. It was a lot of fun to do some research on various [soundtracks and albums](https://open.spotify.com/playlist/3Y3iXAfwGYXyDS884v9xcY?si=i_RJCOf-T1mkMIwtc8cpNw).

## Character AI

```lua
-- game/rules/enemy/actions/check_spawn_enemylua
local aiRoutines = {
  randomMove = require "game.rules.enemy.ai.random_movement"
  require "game.rules.enemy.ai.move_towards_player",
  require "game.rules.enemy.ai.random_movement"
}
local tables = require "moonpie.tables"
local randomChance = 2

return function(spawner)
  return function(dispatch)
    local check = love.math.random(100)
    if check < randomChance then
      dispatch(Character.actions.add(
        Character.create {
          x = spawner.x,
          y = spawner.y,
          isEnemy = true,
          ai = tables.pickRandom(aiRoutines)
        }
      ))
    end
  end
end
```

Added a new AI that moves directly towards the character. Monsters are created with a random AI module
attached. This gives a different interaction with the aliens to make the game feel a bit more interesting.
It also opens the possibility to code up some more complex AI interactions.


## Improvement to Items

Items went through the biggest change in this past week.

- Items can be `cloned`. This creates a duplicate copy of an item that can be added to the game.
- Items can be placed on the map. The character can pick them up and add these items to the inventory.
- Items that are used are consumed from the inventory and removed from the game.







## Final Stats
- **68** Files Update
- **867** Additions
- **207** Deletions
- **97.21%** [Code Coverage](https://coveralls.io/builds/37369994) 

_Code coverage has an error where it's reporting on missing lines from external libraries_
_that started a few builds ago. This is the code coverage from the game folder._

[All the changes](https://github.com/tredfern/terminus/compare/20210215...devlog-8)

![Progress](https://www.dropbox.com/s/up279nlfg726e5o/terminus_devlog_8.gif?raw=1)