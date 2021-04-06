---
title: Terminus Devlog Week 11
layout: post
excerpt: Reviewing architecture and design
---

> ## Items Completed
> - [#100 - Create Player Rules Area](https://github.com/tredfern/terminus/issues/100)
> - [#102 - Create NPC Area](https://github.com/tredfern/terminus/issues/102)
> - [#63 - Sound Management Improvements](https://github.com/tredfern/terminus/issues/63)
> - [#76 - Refactor Enemy Spawners](https://github.com/tredfern/terminus/issues/76)
> - [#67 - Add sounds for moving and fighting](https://github.com/tredfern/terminus/issues/67)

## Refactors and Planning

For the past few weeks I've been evaluating different strategies to improve the way state
is managed. In particular, while the rules and actions feel very clean, testing sometimes
is a bit problematic. Specifically, anytime using thunk style actions that called other
thunk style actions, it was cumbersome to write the tests.

### Entities

Entity management was the first main concern. The characters section is a bit clunky with
how characters are created and added. Also, what components should characters have? What
should the process be for adding or modifying skills? How should inventory management be
coordinated? How to keep track of the position of characters, items, in relation to
each other? Spawners were added to the "map" data store, do they belong there?

I researched implemented various Entity-Component-System patterns. I think this is the right
foundation going forward, which is to split some of the state management from the rules management.
The rules representing more of the system behaviors in an ECS. But one thing I like about
the current model, is while actions are problematic to test at times, they provide great 
encapsulation between when/how state is updated and where logic is happening.

To start the refactor I've created a `world` rule area that provides an aggregate view of
all the entities in the game. This allows getting all entities that match certain component
requirements without concern about what `type` of entity they are. So, retrieving all the
entities that can be drawn can be simplified to a single selector.

In the long run, all entities will be stored and managed by the state, but for the initial
refactor, just splitting out the selector and aggregating together is helping entity management.

## Map Management

The big concern I'm working on now is how to organize map data more intelligently. This next milestone
is heavily dependent on creating more sophisticated map data. In particular:

* Maps need to have doors
* Maps need to have ladders/portal to connect to other maps
* Maps need to have descriptions attached to rooms, or corridors
* Tiles should be able to have a variety of tiles to break up the monotony of the grid

This means we need to break apart some of the map data and simplify state management into a very basic
storage mechanism and provide the right selectors, accessors into the data. 

For example, I should be able to ask, what room am I in right now? I should also be able to get a list of tiles
from x,y-w,h. 

These refactors are slow right now because I'm not sold on a particular solution. I'm keeping my desire for
easy to TDD the game and that is what is putting an additional constraint on finding an ideal pattern. Just
doing it in a fairly clean approach wouldn't be much concern, but how to do it in a way that is easy to test??
