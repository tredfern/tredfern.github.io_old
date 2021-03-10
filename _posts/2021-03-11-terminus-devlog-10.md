---
title: Terminus Devlog Week 10
layout: post
image: https://www.dropbox.com/s/2ks2etr99uhcxf7/postimage-20210311.png?raw=1
excerpt: Milestone 1 - DONE!, Milestone 2 Begins! ![Video](https://www.dropbox.com/s/t2kcd50a7edvm9f/terminus_devlog_10.gif?raw=1) 
---

> ## Items Completed
> - [#69 - Refactor some enums](https://github.com/tredfern/terminus/issues/69)
> - [#82 - Fix game saving](https://github.com/tredfern/terminus/issues/82)
> - [#73 - Graphics for spawners](https://github.com/tredfern/terminus/issues/73)
> - [#61 - Add some "Options" to Create Character](https://github.com/tredfern/terminus/issues/61)
> - [#62 - Add sounds for moving and fighting](https://github.com/tredfern/terminus/issues/62)

## Wrapping up Milestone 1

Milestone 1 was to get a collection of basic functionality together in the form of a game. It also was to prove
out the functionality of my [Moonpie](/moonpie) framework.

Currently enabled in the game:
- A way to win and lose
- Options to change the screen size resolution
- Loading/Saving the game
- Creating a character
- Random map generation
- Graphics for all the entities in the game
- Some sound effects
- Music

While the game mechanics are light at this point, the framework for implementing is in a good place. There is
a robust ability to create new actions into the game and have those dispatch cascading actions. This allows
for a way to define a flexible ruleset that can be used to create the game. 

The organization of the code feels particularly strong, at least at the root levels:
```
assets
├─ fonts
├─ graphics
├─ music
├─ sounds
├─ ..
data
├─ names
├─ items
├─ ..
game
├─ rules
    ├─ character
      ├─ actions
      ├─ selectors
    ├─ combat
    ├─ game_state
    ├─ ..
├─ settings
├─ ui
```

With this structure it's clear where certain bits of code needs to live. Need to add a different rule set, make a folder
and name it appropriately. Each of these rules can manage a slice of state. However, there is some refactoring
that is necessary. But that is where _Milestone 2_ makes an entrance.

## Milestone 2

With the basics of a game in operation, it's time to make it interesting and focus on the main loop. Some key observations:

- UI Improvements
  - Hot slots for items that are equipped
  - Better hotkeys that can be reconfigured
  - Message display should be more intuitive
  - Animating UI elements
- Graphics
  - Animations for moving, fighting etc...
  - Start getting more specific on palette for colors
  - Visual additions, blood trails on ground etc...
  - Shader effects, lighting or other kinds of things
  - Tighten up how resolution will work, and how the map should display that
- Sounds
  - Sounds kind of clip on replay, better management of them
  - Better control of sound volume
  - Sounds for pick up items, defeated enemy, etc...
- Music
  - Some soundtrack during game
  - Victory music/sounds for rewards
- Gameplay
  - Game needs to start taking on the elements that make it more fun
  - Items should have more interesting interactions
  - All ability scores should serve a purpose
  - Additional NPC's that are not enemies
  - Add more items
- Maps
  - Specific room types
    - Engineering, bridge, storage, medbay, quarters, etc...
    - Specific rooms should have different items
  - Rooms should be able to have predesigned layouts
  - Ladders that lead to new levels
  - Doors!
- Other stuff
  - Refactor some of the state management
    - Position of items, characters, etc... should be centralized
    - Graphics for animations might need a more central state
    - How to allow walking animations so there is kind of a play between the turns?
  - Line of Sight
  - Add a prototype for crafting mechanics? Because you know, everything needs a crafting mechanic these days.

This is a very large list right now, and some of these will be deferred to a later milestone I'm sure. But there
is no question that Milestone 2 will be much more significant as by the end of it, while it doesn't have to be a
polished game, it should feel like a game that can be played and feedback could be received.

## Final Stats
- **92** Files Update
- **394** Additions
- **189** Deletions
- **86%** [Code Coverage](https://coveralls.io/builds/37816703)

[All the changes](https://github.com/tredfern/terminus/compare/devlog-9...devlog-10.1)

![Progress](https://www.dropbox.com/s/t2kcd50a7edvm9f/terminus_devlog_10.gif?raw=1) 