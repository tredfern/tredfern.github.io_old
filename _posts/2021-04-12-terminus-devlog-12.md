---
title: Terminus Devlog Week 12
layout: post
image: https://www.dropbox.com/s/7he3ekhkbw7so29/postimage-20210413.jpg?raw=1
excerpt: Map Image Improvements ![Video](https://www.dropbox.com/s/z0o412veobm0y6y/terminus_devlog_12.gif?raw=1)
---

> ## Items Completed
> - [#97 - Refactor map generation](https://github.com/tredfern/terminus/issues/97)
> - [#67 - Clean up Label-Pair UI](https://github.com/tredfern/terminus/issues/67)

## Map generation improvements

One of the flaws in the original implementation is that there was no way to assign a specific image to a tile. This limited the ability to have variation or more detail on the map. The other challenge was that it was just
tracked as terrain, there was no bucket to add in additional data.

The new map information breaks the map into two different data structures.

An `outline` that represents the room and corridors that are within the map. This could be improved to highlight what rooms connect to which corridors etc... This could provide a general way of viewing
the map as a sequence of areas as opposed to just a collection of tiles. Things like making rooms link
together in a more logical sequence could then be managed by the outline.

A `tilemap` that represents the individual squares of the map. This is what the character actually moves on.

A future iteration will be to remove walls as "terrain/tilemap" features but instead replace them as entities
on the map. This would make destructible walls easier to manage. 


## Reason for these changes

The main reason is that drawing the map has become more data driven and simplified. The code to render the screen is able to quickly iterate over the map. If new terrains or information is added, it's easy to update
the information. Also, tiles are able to hold more information, which could allow for more variance or
information associated with particular parts of the map. Though ideally, I'd like for entities to drive
that interaction more than the tilemap itself. The tilemap should really be there to give a general layout
and get the screen rendered quickly. Those items and things in the world should make the actual map display.


## Next up

I'd like to explore adding some descriptions and more variance for rooms to give a bit more character to
the maps. Maybe some rooms with different shapes as well. 



## Final Stats
- **59** Files Update
- **623** Additions
- **522** Deletions
- **86%** [Code Coverage](https://coveralls.io/builds/38729029)

[All the changes](https://github.com/tredfern/terminus/compare/devlog-11...devlog-12)

![Progress](https://www.dropbox.com/s/z0o412veobm0y6y/terminus_devlog_12.gif?raw=1)
