---
title: Terminus Devlog 15 - UI and Performance
layout: post
image: https://www.dropbox.com/s/ecg8ynwvyyp3x75/postimage-devlog15.png?raw=1
excerpt: Caching queries, Github Actions, Zoomed out maps, Screen design ![Video](https://www.dropbox.com/s/ltp4204mcdoumr4/terminus_devlog_15.gif?raw=1)
---

> ## Items Completed
> - Caching for commonly used selectors
> - Github Actions for CI
> - [#37 - Zoomed out map view](https://github.com/tredfern/terminus/issues/37)
> - [#118 - Create basic screen layouts](https://github.com/tredfern/terminus/issues/118)

## Caching Queries

I spent a lot of time investigating some of the performance opportunities in _Terminus_. The code design I've
settled on is highly modular, but that can lead to some inefficiencies as there is more communication or duplicate
data retrieval. 

Roguelikes seem to need most of their optimization in how the process a **turn** more than how should each frame
behave. General animations and things like that are usually not that demanding, and simplifying graphics load
is usually an obvious optimization. (Don't draw what won't be seen)


### Selectors and Actions

State in [Moonpie](https://github.com/tredfern/moonpie) Apps are managed through actions/reducers + selectors.
This is similar to the Redux implementation or any other segmentation of Pub-Sub/State Management. This allows
a clear entry point into when data is accessed and when data is changed. 

This can be leveraged to cache certain responses instead of searching the state tree every time a request is made for
some data. This cache can be flushed and refreshed in response to changes to the state.


### Implementation

The `Cache` implementation is as follows:
1. Register the `Cache` with an _ID_, _Query_ that populates the cache, _Events_ that flush the cache
2. On first request the Cache will populate the results in the `Cache`
3. From then on, any request to the Cache will return the data in the `Cache`
4. If an event that the `Cache` is monitoring triggers then previously stored results are cleared.
5. The next query will behave as in Step 2.

```lua
-- Excerpt from game/cache.lua
local flushCallbacks = {}
local results = {}

function Cache.lookup(cacheDef)
  local name = cacheDef.name

  -- Configure when cache should clear
  if flushCallbacks[name] == nil then
    Cache.addFlushCallbacks(cacheDef)
  end

  -- Check for cached results
  if results[name] == nil then
    results[name] = cacheDef.source()
  end
  return results[name]
end
```

### Outcome

The implementation of the cache has improved some of the turn processing. When processing a turn there might be
multiple duplicate lookups and this simplifies the queries down. Missing optimized queries based on position still
limits the performance. I made a decision to hold off on that a bit longer until I'm convinced of the performance
gains as well as feeling that the position implementation is not going to change.

### Optimizations
Some potential optimizations:

1. The cache instead of flushing could monitor what was changed. This would require an update to the store to broadcast
  the state that had been updated.
2. More advanced caching that handle particular query values: Right now, it is difficult to cache with a certain query.
  For example, requesting entities in a specific position is difficult to cache because it requires knowing what position
  is being cached.


## Github Actions

Travis-CI over the past 6 months or so, switched to a pay model for all repositories that build on their platform.
I have no complaints with a service that costs money, charging money to stay supported. The problem for me was that
the price point was so much greater than the return for having a couple of repositories run automated tests. If there
had been some sort of offering that was more inline with my use case, I would have stuck around.

There is a simple alternative though and that was to switch to [Github Actions](https://github.com/features/actions).
Seems to be in response to some of the offerings that Gitlab has been putting out. Anyway, I've set up automated
build steps that will run the unit tests for [Terminus](https://github.com/tredfern/terminus).

Included in this, I've updated [Moonpie](https://github.com/tredfern/moonpie) and the 
[Moonpie Template](https://github.com/tredfern/moonpie-template) projects. Using the template project should mean
that any future Moonpie project or game I create will automatically get configured with continuous integration including
code coverage reporting. This will simplify the process greatly going forward.

_Here is the configuration file I created if it helps anyone else to build Love2D/Lua projects with Coveralls reporting._
```yml
# Build configuration for Terminus
name: build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      COVERALLS_REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    steps:
    - uses: actions/checkout@v2
      with:
        submodules: true

    - uses: leafo/gh-actions-lua@v8.0.0
      with:
        luaVersion: "luajit-2.1.0-beta3"

    - uses: leafo/gh-actions-luarocks@v4.0.0

    - name: build
      run: |
        luarocks install luacheck
        luarocks install busted
        luarocks install luacov
        luarocks install luacov-coveralls
        luarocks install luafilesystem

    - name: test
      run: |
        luacheck main.lua ./game
        busted --run=travis
        luacov-coveralls --exclude ext --exclude /home
```

## Zoomed out Map View

This was not a challenging change, but one that provides a lot of value. Rogue games are all about maps. That
sense of exploration is possible because the map changes each time. And, keeping track of everything requires the
ability to zoom out and see things from a zoomed out perspective.

This implementation just loops through all the tiles and renders out the tiles for the map.

```lua
-- Excerpt from game/ui/widgets/world_map_detail.lua
-- Demonstrates custom drawing within Moonpie Components
    drawComponent = function(self)
      local state = store.getState()
      local tileWidth = math.floor(self.box.width / self.mapSize.width)
      local tileHeight = math.floor(self.box.height / self.mapSize.height)

      for x = 1, self.mapSize.width do
        for y = 1, self.mapSize.height do
          local p = Position(x, y, self.currentLevel)
          local t = Map.selectors.getTile(state, p)
          if FogOfWar.selectors.get(state, self.player, p) and t then
            if t.terrain.key == "wall" then
              love.graphics.setColor(Colors(Colors.gray))
              love.graphics.rectangle("fill", (x - 1) * tileWidth, (y - 1) * tileHeight, tileWidth, tileHeight)
            elseif t.terrain.key == "room" then
              love.graphics.setColor(Colors(Colors.light_accent))
              love.graphics.rectangle("fill", (x - 1) * tileWidth, (y - 1) * tileHeight, tileWidth, tileHeight)
            elseif t.terrain.key == "corridor" then
              love.graphics.setColor(Colors(Colors.light_shade))
              love.graphics.rectangle("fill", (x - 1) * tileWidth, (y - 1) * tileHeight, tileWidth, tileHeight)
            end
          end
        end
      end

      if self.player.position.z == self.currentLevel then
        local x, y = self.player.position.x -1, self.player.position.y - 1
        love.graphics.setColor(Colors(Colors.playerBlip))
        love.graphics.rectangle("fill", x * tileWidth, y * tileHeight, tileWidth, tileHeight)
      end
    end,
```


## User Interface

The other key step I worked on was mapping out what screens I thought would be most important for the game. Some
of these will change and some will be broken down in more detail. For example, the character creation screen
will need some more sub-screens to help guide the process. There will also likely be some sort of encyclopedia or
other information collection system that collates data together.

I've created the basic structure for all the non-dialog based screens to make it easier to add new functionality
going forward.

![Screen Map](https://www.dropbox.com/s/x9paaufp1ay5fkp/Screen%20Navigation%20Map.png?raw=1)

---
## Statistic Report
<pre>
───────────────────────────────────────────────────────────────────────────────
Tag: devlog-15
───────────────────────────────────────────────────────────────────────────────
Files Update:        <strong>83</strong>
LOC Added:           <strong>1038</strong>
LOC Deleted:         <strong>270</strong>
Code Coverage:       <strong>97%</strong>

───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Lua                        662     27588     3746      3009    20833       1343
Markdown                    21      1163      313         0      850          0
ReStructuredText            13       464      142         0      322          0
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
Total                      717     31471     4368      3055    24048       1348
───────────────────────────────────────────────────────────────────────────────
</pre>

[All the changes](https://github.com/tredfern/terminus/compare/devlog-15...devlog-14)

![Video](https://www.dropbox.com/s/ltp4204mcdoumr4/terminus_devlog_15.gif?raw=1)