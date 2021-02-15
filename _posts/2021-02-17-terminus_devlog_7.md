---
title: Terminus Devlog Week 7
layout: post
image: https://www.dropbox.com/s/3trtgjbml73wesb/postimage-20210217.png?raw=1
excerpt: Graphics, spawners, healing and more! ![Progress](https://www.dropbox.com/s/7vim4m70pn9jtbp/terminus_devlog_7.gif?raw=1)
---

> ## Items Completed
> - [#20 - Items can be equipped](https://github.com/tredfern/terminus/issues/20)
> - [#18 - Random name generation](https://github.com/tredfern/terminus/issues/18)
> - [#32 - LoadCoreData is dispatchable](https://github.com/tredfern/terminus/issues/32)
> - [#21 - Combat rolls use equipped item for attacks](https://github.com/tredfern/terminus/issues/21)
> - [#19 - Skill checks can be opposed](https://github.com/tredfern/terminus/issues/19)
> - [#23 - Sounds on menu](https://github.com/tredfern/terminus/issues/23)
> - [#22 - Defensive skills for combat system](https://github.com/tredfern/terminus/issues/22)
> - [#7 - Enemy Spawners](https://github.com/tredfern/terminus/issues/7)
> - [#52 - Use weapon damage](https://github.com/tredfern/terminus/issues/52)
> - [#34 - Use "unarmed" weapon if no weapon equipped](https://github.com/tredfern/terminus/issues/34)
> - [#35 - Health stat is calculated](https://github.com/tredfern/terminus/issues/35)
> - [#27 - Items can be used](https://github.com/tredfern/terminus/issues/27)
> - [#12 - Health kit](https://github.com/tredfern/terminus/issues/12)
> - [#57 - Count of enemies and spawners](https://github.com/tredfern/terminus/issues/57)
> - [#25 - Graphics for wall and floor](https://github.com/tredfern/terminus/issues/25)

## So Much Stuff!
This is likely the most single productivity week on the project so far. I believe this is because enough pieces have
started to come together and this is making it possible to start focusing on the work of connecting things together
instead of just laying down foundation and building blocks.

### Using and Equipping Items
There isn't much yet on the UI for this functionality. There is the ability to define, `equipSlots` on the inventory
that will be places that the character can attach their weapons, armor and other items. For now, this is only
the Melee weapon slot and this is hardcoded to use the default sword provided to the character. The combat system
is aware of this weapon and will utilize the appropriate skill and damage when making attacks with the weapon.

The other development was to provide a way to `use` and item. Using an item does not consume it yet, but it performs
some action. When creating / describing a new item for the game, there will be the possibility of saying that the
item is usable and specifying a handler for what should happen when the character uses the item. This is passed
a `dispatch` function that can be used for updating the state with whatever action is appropriate. 


### Sounds and Graphics
A big development this week was to introduce some elements of sounds and some visuals to the map.

For sounds, there is some sounds played on hover and clicking on buttons. These sounds, are honestly, not very good
right now. The volume is out of sync and the playback code is not well organized. This is resulting in some weird
click sounds when rewinding sounds at inappropriate times. However, the key thing is that this is now in there which
uncovered some better investments that could be made before introducing more sounds to the game itself.

For graphics, I have been taking some pixel art tutorials. [Udemy](https://www.udemy.com/course/learn-to-create-pixel-art-for-your-game/learn)
The course from Udemy has been very informative. It's short and too the point, but effective and easy to apply the
concepts. In fact, it encouraged my to create the first concept art for the game!

_Not how the game will look but something to get inspiration from._
![Concept Art](https://www.dropbox.com/s/13muv93r2z11g4w/concept-art-1-zoom.png?raw=1)


The output of this so far though, is some very rough floors and walls that help start to give some texture to the game.
There is so much work to do on the graphical front, that it's important for me to get started and make sure I'm
investing at least a portion of each week into improving the visuals.


## Final Stats
- **104** Files Update
- **2088** Additions
- **366** Deletions
- **97.244%** [Code Coverage](https://coveralls.io/builds/37148605)

_Test coverage went down, but with ~1700 new lines of code the drop was not that bad. Also, the majority of this drop
was in `game/ui/widgets/combat_map.lua` from adding the new graphics to the game. Because that is all scratch code,
there isn't anything concerning to me in the numbers._

[All the changes](https://github.com/tredfern/terminus/compare/poststream-20210207...20210215)

![Progress](https://www.dropbox.com/s/7vim4m70pn9jtbp/terminus_devlog_7.gif?raw=1)