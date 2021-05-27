---
title: Terminus Devlog 16
layout: post
image: https://www.dropbox.com/s/nnbgxq4bld0vzm5/postimage-devlog16.png?raw=1
excerpt: Statistics, Locking Doors, Descriptions, and Ducks ![Video](https://www.dropbox.com/s/zna0pb7hrh6y2qo/terminus_devlog_16.gif?raw=1)
---

> ## Items Completed
> - [#15 - Locked doors that are unlocked by items](https://github.com/tredfern/terminus/issues/15)
> - [#122 - Refactor to Ducks style system](https://github.com/tredfern/terminus/issues/122)
> - [#79 - Statistics tracking section](https://github.com/tredfern/terminus/issues/79)
> - [#81 - Stats on the game over screen](https://github.com/tredfern/terminus/issues/81)
> - [#16 - Descriptions when entering room](https://github.com/tredfern/terminus/issues/16)
> - Custom thunk assertions

## Getting Ducks in a Row

My initial organization for the code led to a lot of routines being split into very tiny specific files. This was a great starting place because it forced me to consider single responsibility for each routine and minimize references.

The downside was it created a lot of files and became more problematic to extend functionality. Also, it made things
harder to read and track as the project continues to grow.

I _liked_ the organization of rules to manage specific entities and systems together. I _disliked_ each action being
in its own file and selectors being split out. To address this, I went through a large scale refactor and settled on a scheme similar to the [Ducks](https://www.freecodecamp.org/news/scaling-your-redux-app-with-ducks-6115955638be/) approach.

```bash
# Example from game/rules/stats
actions_spec.lua
actions.lua
init_spec.lua
init.lua
reducer_spec.lua
reducer.lua
selectors_spec.lua
selectors.lua
types.lua
```

This organization keeps `actions`, `selectors`, and `reducer` logic together along with the tests being specific. And
it reduces the total number of files to track and manage. Making things much simpler to edit. Also, actions can
much more easily reference each other. Allowing composing complex Thunks to be much more seamless.

## Statistics Section

Tracking statistics is so important for a roguelike game. I want to be able to score as many different things as possible.
This will lead into achievements in the future, but also it helps to create scores or ways for players to see how
their character is doing.

Code wise there is nothing that interesting. Actions contain both `set` and `count`. Reducer is straightforward and can
add new keys or increment values. Selectors pull the values out.

## Locked Doors

Locked doors was a more interesting implementation. The challenge is where to put the logic. In the end, I don't
think the location is the perfect spot, but it's a starting point.

I created a new `door.lua` file within the `map` rules section. It can add doors to map, lock, unlock, and open doors.
The messages are dispatched from the `player.actions` to unlock and open the door because of player input.

Actually, it feels pretty good, but not certain if the locking/unlocking should be handled by the player or by
the door. Should there be logic to handle different keys, etc... but those decisions can be put off for a bit longer.


```lua
-- From game/rules/player/actions.lua
function Actions.openDoor(orientation)
  return Thunk(
    Actions.types.openDoor,
    function(dispatch, getState)
      local player = Selectors.getPlayer(getState())
      local checkDoor = Position[orientation](player.position)
      local door = Door.selectors.getByPosition(getState(), checkDoor)

      if door then
        if door.locked then
          if Selectors.hasItemOfKind(getState(), "keycard") then
            dispatch(Door.actions.unlock(door))
            dispatch(MessageLog.actions.add(Messages.movement.door.unlocked))
          else
            dispatch(MessageLog.actions.add(Messages.movement.door.locked))
          end
        else
          dispatch(Door.actions.open(door))
        end
      end
    end
  )
end
```

## Room Descriptions

Room descriptions have been added and currently, it just picks random ones. The interesting side effect was tracking
that the player has visited rooms before. This was added as a player reducer to track player specific information. I
felt that this information was different from statistics as it's a little bit of a log.

```lua
-- from game/rules/player/actions.lua
function Actions.move(direction)
  local Map = require "game.rules.map"
  return Thunk(Actions.types.MOVE, function(dispatch, getState)
    local state = getState()
    local player = Selectors.getPlayer(state)

    local newPos = direction(player.position)
    dispatch(Characters.actions.move(player, newPos))

    local newTile = Map.selectors.getTile(state, newPos)
    if newTile then
      dispatch(Actions.enteredRoom(newTile.room))
    end
  end)
end

function Actions.enteredRoom(room)
  if room == nil then return end

  return Thunk(
    Actions.types.ENTERED_ROOM,
    function(dispatch, getState)
      if Selectors.hasVisitedRoom(getState(), room) then return end
      dispatch(MessageLog.actions.add(room.description))
      dispatch(Actions.trackRoomVisit(room))
    end
  )
end

function Actions.trackRoomVisit(room)
  return {
    type = Actions.types.TRACK_ROOM_VISIT,
    payload = {
      room = room
    }
  }
end
```

This bit of code flags whenever the character enters a room. If the character has already visited the room, it just
returns out. But if the player has not visited the room, a description is displayed and the room visit is tracked.

## Custom Thunk Assertions

Usually when testing `Thunks` I want to make sure that the thunk execute/dispatches other expected actions. I might want to test the parameters and validate them specifically, though that could be a bit brittle, but even just making sure the actions I anticipate are running provides a foundation for making sure that code is knitting together the way I expect.

To help with that, I create some custom assertions that poke the `thunk` and catch anything it dispatches. It makes it easy to test that it's working as expected and at the same time keep the `thunk` isolated from causing other changes.

_Example assertion_
![Assertions](https://www.dropbox.com/s/ands8sz6mcw3zxx/thunk-assertions.png?raw=1)

A cool side effect, is that for testing thunks, the simple action creators can be used within the tests to put
together the test parameters to validate. It's possible there will be a brittleness with this approach, but on the
other hand, this allows not having to remember exactly how an action looks in order to make sure that the `Thunk`
is executing the correct dispatches.

```lua
it("sets the map dimensions", function()
  local action = Actions.create(30, 20, generator)
  assert.thunk_dispatches(Actions.setDimensions(30, 20, 10), action)
end)
```

## Statistic Report
<pre>
───────────────────────────────────────────────────────────────────────────────
Tag: devlog-16
───────────────────────────────────────────────────────────────────────────────
Files Update:        <strong>262</strong>
LOC Added:           <strong>3737</strong>
LOC Deleted:         <strong>3421</strong>
Code Coverage:       <strong>97%</strong>

───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Lua                        610     28200     3884      2810    21506       1414
Markdown                    21      1163      313         0      850          0
ReStructuredText            13       534      164         0      370          0
Plain Text                   5       658      108         0      550          0
CSV                          4      1175        0         0     1175          0
JSON                         4       153        0         0      153          0
License                      2        42        8         0       34          0
YAML                         2        66       14         0       52          0
Batch                        1        35        8         1       26          5
Makefile                     1        20        4         7        9          0
Python                       1        56       15        29       12          0
gitignore                    1        51       10         9       32          0
───────────────────────────────────────────────────────────────────────────────
Total                      665     32153     4528      2856    24769       1419
───────────────────────────────────────────────────────────────────────────────
</pre>

[All the changes](https://github.com/tredfern/terminus/compare/devlog-16...devlog-15)

![Video](https://www.dropbox.com/s/zna0pb7hrh6y2qo/terminus_devlog_16.gif?raw=1)