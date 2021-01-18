---
layout: page
title: The Terminus Project
permalink: /terminus/
---

[![Build Status](https://travis-ci.com/tredfern/terminus.svg?branch=main)](https://travis-ci.com/tredfern/terminus)
[![Coverage Status](https://coveralls.io/repos/github/tredfern/terminus/badge.svg?branch=main)](https://coveralls.io/github/tredfern/terminus?branch=main)

> ### The Beginning
> After a tiring 2020, I decided to do something different for myself in 2021. Each weekend, I will be streaming 
> live development for 3-4 hours on this project and talking about what steps I'm taking. I will provide insights 
> into how to write clean tests, different ways to solve programming challenges, and at some point, 
> maybe even have an interesting product.
> This is not a tutorial on how to approach the project. My hope is that streaming a non-tutorial project will help demonstrate solving challenges that tutorials tend to avoid which such as:
> * How to approach *Test Driven Development* and use it to write better code
> * When and how to *refactor*
> * How to *evaluate solutions* when multiple are possible
> * When to write *scaffolding/temporary code* and when to invest in your architecture

## Terminus

Terminus is at this point just a project code name to work under.

## Goals
* 100% Test Coverage
* Easy to mod and extend
* Repeatable game play
* A feeling of a story

## Structure

#### Rules
The game is organized around the concept of _rules_. Rules combine the concepts of:
- __Entities__ - What this rule is represented by. Think of the pieces of a board game
- __State management__ - How is the data for this rule stored in the database. This controls how these entities change.
- __Actions__ - The behaviors that can be initiated in relation to this rule that can update __state__. 
Actions can be either simple, updating a value in an entity controlled by this rule. Or, complex, triggering
multiple other actions, including actions from other rules if appropriate.


#### User Interface
The user interface of the game is dependent on the rules and the state values that are updated within. The UI might
trigger actions based on __User Input__. This allows the future separation of the display of the game, from the data
and ruleset of the game.

## Development

Development is at least partially being performed on [Twitch](https://twitch.tv/turtlecoder). 
Streams are currently scheduled for:
Saturdays 13:00-15:00 EST
Sundays 11:00-12:00 EST

## Game Design

Game design and additional detailed documentation will be maintained within the 
[docs folder](https://github.com/tredfern/terminus/tree/main/docs) on the repository

## Inspiration

I'm focusing my inspiration on games from the past. Certainly there will be ideas that will leverage current games 
including things like Dwarf Fortress/Rimworld or Caves of Qud, or Fall Guys (inspiration can and should come 
from anywhere). These old games captured my mind when I started playing them and finding what made that spark 
happen is what I'm curious about exploring with this project.

**[The Magic Candle](https://en.wikipedia.org/wiki/The_Magic_Candle)** - I played this game when it first came 
out in 1989. It has been a while since I last played it, but I remember being impressed by the mechanics of 
exploration. This game felt huge. And also the ways that you managed your party. 

**[Darklands](https://en.wikipedia.org/wiki/Darklands_%28video_game%29)** - A very interesting RPG that allowed 
a lot of freedom in how you approached the game. But one of my favorite features was that you didn't have to move 
your character through each individual square when walking through a town. How much time did I waste in most RPG's 
just clicking from one side of the map to another just to complete a quest? Being able to navigate to locations 
was fantastic.

**[Ultima V](https://en.wikipedia.org/wiki/Ultima_V%3A_Warriors_of_Destiny)** - The main Ultima game I played. 
I loved the variety of towns and quests that linked things together. Getting the magic carpet, was awesome. 

**[Everything Epyx](https://en.wikipedia.org/wiki/Epyx)** - I grew up looking at everything Epyx released. 
I loved their motto even as a kid of combining strategy with action. Some of their games that are influencing 
my thinking on this project are, Temple of Apshai Trilogy, Dragonriders of Pern (I'm serious, 
I played that game a ton),

An interesting tidbit that I learned from research is that Epyx did the port of Rogue to non-Unix platforms.
