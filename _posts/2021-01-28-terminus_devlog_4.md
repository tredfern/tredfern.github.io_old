---
title: January 23-24 Session Review
layout: post
date:   2021-01-28
image: https://www.dropbox.com/s/empi3ofceze1t2t/postimage-20210127.png?raw=1
excerpt: Generating dungeon maps using binary space partitioning. ![Screenshot](https://www.dropbox.com/s/u9iclyqabcprh55/Screenshot%202021-01-24%20132733.png?raw=1)
---

> **In this [session](https://github.com/tredfern/terminus/blob/main/docs/sessionnotes/20210123.md)**
> - Procedural map using BSP trees
> - Assigning player/enemy start positions
> - Restricting movement
> - Game over!

# Procedural Maps Using BSP Trees

There are many great write ups on how to generate dungeons using BSP trees. Some of
my favorite ones are:
* [Grid Sage Games/Cogmind](https://www.gridsagegames.com/blog/2014/06/procedural-map-generation/) gives a great overview with some extra links to other articles.
* [RogueBasin](http://roguebasin.roguelikedevelopment.org/index.php?title=Basic_BSP_Dungeon_generation) provides a simple to understand and read approach.

Both of these sections are great to read through if you need an overview of generating dungeon maps based on BSP trees. 

## Implementation
[Commit](https://github.com/tredfern/terminus/commit/d17d1afae8c2a790e5cd7c3dd54032c48551f507)

For organizing the files, I placed this new generator in `game/rules/map/generators/dungeon.lua`. I am assuming that there will eventually be other generators that I want to use to make maps, so keeping them nicely bundled together makes sense. I want these to be accessible and easy to manage.

### TDDing Procedural Generation
When trying to write unit tests for code that will generate random and complex data, it can be hard to identify what exactly to test. For this, I recommend testing specific functions in the implementation and make sure they are following the steps that you expect them to. This makes sense because in procedural generation, these steps will repeat over and over again to generate the information. Making sure they are behaving to the rules defined is important. 

Also, limiting the number of functions that actually do random steps and instead try to put the logic in functions that take the values from random generators. This allows you to remove randomness as a key component of your tests. You aren't
testing that something is random, you are testing that given a value, it behaves in an expected way. (_I did not do the best job in separating this out on the live stream. But in reviewing the generator, this would be ideal to clean up sometime_)


### Nodes

```lua
function generator.create_node(x, y, width, height)
  return {
    x = x,
    y = y,
    width = width,
    height = height,
    pick_room = function(self)
      if self.room then return self.room end

      if self.left and self.right then
        if math_ext.coinflip() then
          return self.left:pick_room()
        else
          return self.right:pick_room()
        end
      end
    end
  }
end
```
A key component in this process was creating nodes that contains the position and size of the node. This is the fundamental structure of the generator. As nodes divide each tree will contain 2 children that subdivide the parent into different. 

The pick_room function was added to so that later when figuring out how to assign players and enemies to rooms, we can grab a random room out of the tree. !! This is an inefficient method of adding this function, we repeat a new function for each node. It's not a big deal, but it's not the best.

```lua
function generator.generate(map)
  local root = generator.create_node(1, 1, map.width, map.height)
  generator.divide(root, 1, DEPTH)
  generator.create_rooms(root)

  -- generator.add_border_walls(map, root)
  generator.add_rooms(map, root)
  generator.add_corridors(map, root)
end

function generator.divide(node, current_level, max_levels)
  if current_level > max_levels then return end
  if node.width < MIN_SIZE_TO_DIVIDE and node.height < MIN_SIZE_TO_DIVIDE then return end

  if node.width >= node.height then
    generator.divide_on_x(node)
  else
    generator.divide_on_y(node)
  end

  generator.divide(node.left, current_level + 1, max_levels)
  generator.divide(node.right, current_level + 1, max_levels)

  return node
end

function generator.create_rooms(node)
  if node.left or node.right then
    generator.create_rooms(node.left)
    generator.create_rooms(node.right)
    return
  end


  local width = math.random(math.ceil(node.width / 2), node.width - 1)
  local height = math.random(math.ceil(node.height / 2), node.height - 1)
  local x = node.x + math.random(node.width - width)
  local y = node.y + math.random(node.height - height)

  node.room = {
    x = x, y = y, width = width, height = height
  }
end
```

The key to generating the dungeon is really these 3 steps. `generate` is the entry point into the process and outlines our strategy for creating the dungeon. `divide` and `create_rooms` are working in the BSP structure and provide the foundation for the data that will be used to fill in on the map.

Overall, there is nothing that is that odd about these steps. The only trick we add was to always divide on the longer side to make sure that we are shrinking in a way that is going to leave the most space for rooms.

```lua
function generator.add_rooms(map, node)
  if node.left or node.right then
    generator.add_rooms(map, node.left)
    generator.add_rooms(map, node.right)
    return
  end


  for x = 0, node.room.width - 1 do
    for y = 0, node.room.height - 1 do
      map:set_terrain(node.room.x + x, node.room.y + y, terrain.room)
    end
  end
end

function generator.add_corridors(map, node)
  if node.left or node.right then
    generator.add_corridors(map, node.left)
    generator.add_corridors(map, node.right)
  end

  --Connect the left and right nodes together...
  if node.left and node.right then
    local start_room = node.left:pick_room()
    local end_room = node.right:pick_room()
    -- pick 2 connecting points
    local start_x = math.random(start_room.x, start_room.x + start_room.width - 1)
    local start_y = math.random(start_room.y, start_room.y + start_room.height - 1)

    local end_x = math.random(end_room.x, end_room.x + end_room.width - 1)
    local end_y = math.random(end_room.y, end_room.y + end_room.height - 1)

    generator.build_corridor(map, start_x, start_y, end_x, end_y)
  end
end
```

The final 2 routines of `generate` add data to the map to represent the rooms.
Adding rooms, will traverse the tree and fill in the map data with room information.

**Corridors** were more interesting. In this implementation, I only added the corridor data
to the map. This was done by going all the way down to the _parent_ of the leaf nodes (`max_depth - 1`)
and joining it's two child rooms together. We then step up one level, choose 2 random rooms (using `pick_room`) from the tree and joining them together. When we get to the top and complete our final join, all the rooms should be connected by a path at this point.

The corridors themselves are implemented by picking a random point in each room, and connecting those points together. We only change the terrain when the terrain is empty and this avoids making weird crossing lines over the map. Sometimes corridors can natural intersect creating crossroads. For such a lightweight implementation, it does create some reasonable map designs.

## Player / Enemies using map data
- [Starting Positions](https://github.com/tredfern/terminus/commit/28ff06d1242a4c32b87fe3145cf71650f6620877)
- [Movement](https://github.com/tredfern/terminus/commit/3eac298a19931668a85495ad534221a993246684)


Next step was to position the player and enemies within random rooms.

```lua
-- Updated game/rules/game_state/actions/setup.lua
local character = require "game.rules.character"
local map = require "game.rules.map"
local tables = require "moonpie.tables"

return function()
  return function(dispatch, get_state)
    dispatch(map.actions.set(
      map.create(50, 50)
    ))
    local rooms = map.selectors.get_rooms(get_state())

    local player_start_room = tables.pick_random(rooms)

    dispatch(character.actions.add(
      character.create {
        x = player_start_room.x + math.floor(player_start_room.width / 2),
        y = player_start_room.y + math.floor(player_start_room.height / 2),
        is_player_controlled = true
      }
    ))

    for _ = 1,8 do
      local r = tables.pick_random(rooms)
      dispatch(character.actions.add(
        character.create {
          is_enemy = true,
          x = r.x + math.floor(r.width / 2),
          y = r.y + math.floor(r.height / 2),
        }
      ))
    end
  end
end
```

This modified the action so that instead of just picking random coordinates on the map, it picks a random room, from our tree data, and places the character into the middle square.


```lua
-- snip of game/rules/map/terrain.lua
  blank = {
    color = colors.oxford_blue,
    blocks_movement = true
  },


-- updated game/rules/character/actions/set_position.lua
return function(character, x, y)
  return {
    type = types.character_set_position,
    payload = {
      character = character,
      x = x,
      y = y
    },
    validate = function(self, state)
      local terrain = map.selectors.get_terrain(state, self.payload.x, self.payload.y)
      return not terrain.blocks_movement
    end
  }
end
```

What really turns this into a "game" is restricting the movement to only valid squares. To accomplish this, we added a flag to the terrain that says whether the terrain blocks movement. Simple enough. For each movement action, we changed the validation to check for this flag. 

And that was all it took for movement to be restricted to only the squares that define the map!

## Game over man, Game over
[Commit](https://github.com/tredfern/terminus/commit/694094d8a3bfed75bf809f7bc8cd4d3fbe6e0a77)

The final change from the stream was to return to the title screen when the character died. Hey, it has all the game states now!

```lua
-- snip from game/rules/turn/actions/process.lua
if dead then
  local player_died = false
  for _, e in ipairs(dead) do
    player_died = player_died or e.is_player_controlled
    dispatch(character.actions.remove(e))
  end

  if player_died then
    dispatch(game_state.actions.game_over())
    return
  end
end

-- game/rules/game_state/actions/game_over.lua
return function()
  return function()
    local app = require "game.app"
    app.title()
  end
end 
```

Every time the turn is processed, if the character has died then we end the game. It's a primitive check, but it is sufficient to placehold for the logic we want to have.

I have some questions about this because this is an action and it will change the screen. And, it is also clearly encapsulating updating that state and is easy to manage. Keeping the various screens in the app makes sense to me at this time. Because the app has so little to manage, making it the central area for what screen we are on, and what the game state is, makes the most sense to me.

#### Final Stats
- **57** Files Update
- **705** Additions
- **143** Deletions
- **97.713%** Code Coverage (increased)

[All the changes](https://github.com/tredfern/terminus/compare/poststream-20210117...poststream-20210124)

![Screenshot](https://www.dropbox.com/s/u9iclyqabcprh55/Screenshot%202021-01-24%20132733.png?raw=1)
*End of stream screenshot. The player has moved to the boundary of a room and cannot move further. Map is connecting a series of rooms with various corridors connecting them together*

