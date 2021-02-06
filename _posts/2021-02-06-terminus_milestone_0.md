---
title: Milestone 0 - Completed!
layout: post
date: 2021-02-06
image: https://www.dropbox.com/s/irggeabq3dusjlf/terminus_milestone_0.gif?raw=1
excerpt: Overview of Milestone 0, basic foundations loading, saving and options...
---

[![Release](https://img.shields.io/badge/Download-Milestone%200-informational)](https://github.com/tredfern/terminus/releases/tag/milestone-0)

> ### Milestone 0: Basic Foundations
> ##### [All Changes](https://github.com/tredfern/terminus/compare/poststream-20210131...milestone-0)
> - Options screen that allows setting resolution
> - Character Details Screen
> - Saving game to data file
> - Reload game from file

### Options Screen

Before initiating serious development, I wanted to provide a way to set different resolutions in order to test and verify the application. Also, having an options screen makes it easier to add additional options in the future.

```lua
local Components = require "moonpie.ui.components"
local FullScreenPanel = require "game.ui.widgets.full_screen_panel"
local VideoResolution = require "game.ui.widgets.video_resolution"

local OptionsScreen = Components("options_screen", function(props)
  local videoResolution = VideoResolution { id = "videoResolution" }
  return {
    id = "options_screen",
    FullScreenPanel {
      title = "Options",
      contents = {
        Components.text { text = "Screen Resolution: "},
        videoResolution,
        Components.button {
          id = "btnApply",
          caption = "Apply",
          click = function()
            local res = videoResolution:getResolution()
            love.window.setMode(res.width, res.height)
            moonpie.resize(res.width, res.height)
          end
        },
        Components.button { id = "btnClose", caption = "Close", click = props.returnScreen },
      }
    }
  }
end)

return OptionsScreen
```

This outlines the options screen component. It creates a component that has a widget for `VideoResolution`. That component handles picking the appropriate settings and allowing the user to select different options. (Details in next code block.) When the user clicks apply, the video resolution information is retrieved from the component and
then applied to the settings. The call to `moonpie.resize` is to let the framework know that the screen has changed. Love2D does not always notify via the resize event and this allows the screen to update.


```lua
local Components = require "moonpie.ui.components"
local tables = require "moonpie.tables"

local VideoResolution = Components("video_resolution", function()
  local curWidth, curHeight = love.window.getMode()
  local modes = love.window.getFullscreenModes()
  local _, index = tables.findFirst(modes, function(m) return m.width == curWidth and m.height == curHeight end)

  local component
  local updateResolution = function(nextIndex)
    if nextIndex < 1 then nextIndex = #modes end
    if nextIndex > #modes then nextIndex = 1 end

    component:update({
      modeIndex = nextIndex,
      resolutionWidth = modes[nextIndex].width,
      resolutionHeight = modes[nextIndex].height
    })
  end

  component = {
    modeIndex = index,
    resolutionWidth = curWidth,
    resolutionHeight = curHeight,
    getResolution = function(self)
      return { width = self.resolutionWidth, height = self.resolutionHeight }
    end,
    render = function(self)
      return {
        Components.button { id = "btnPrev", caption = "<<-",
          click = function() updateResolution(self.modeIndex + 1) end},
        Components.text {
          id = "displayText",
          text = string.format("%d x %d", self.resolutionWidth, self.resolutionHeight) },
        Components.button { id = "btnNext", caption = "->>",
          click = function() updateResolution(self.modeIndex - 1) end },
      }
    end
  }

  return component
end)

return VideoResolution 
```

The `VideoResolution` component is not particularly challenging. It gets the modes out of Love2D and then provides some buttons to cycle through the various modes. There is an accessor function that provides what resolution is selected and whether full-screen is checked.

### Character Details Screen

```lua
local Components = require "moonpie.ui.components"
local FullScreenPanel = require "game.ui.widgets.full_screen_panel"
local Character = require "game.rules.character"
local connect = require "moonpie.redux.connect"


local CharacterDetails = Components("character_details", function(props)
  local app = require "game.app"

  return {
    id = "character_details_screen",

    FullScreenPanel {
      id = "screenPanel",
      title = props.characterName,
      contents = {
        Components.button { id = "btnClose", caption = "Close", click = app.combat }
      }
    }

  }
end)

return connect(
  CharacterDetails,
  function(state)
    local player = Character.selectors.getPlayer(state)
    return {
      characterName = player.name
    }
  end
)
```

The `CharacterDetails` screen is mostly a placeholder for future development. When adding some more inventory and other options this will provide the framework to quickly add in new features. The screen is already wired up to the store which will allow quickly developing the rest of the components.

### Saving / Loading Game Files

Saving and Loading is one of the things that I've always put off for too long and when trying to implement it, becomes challenging. Fortunately, Lua already provides great libraries for serializing data. Also, with the `moonpie.redux` approach, it is easy to serialize out the state and replace state again.

The only tricky thing was figuring out which library to use and how to test it.

I chose [Binser](https://github.com/bakpakin/binser). It covered the use cases necessary and supported running in unit tests. Now, Love2D is LuaJit dependent and we could look at some custom LuaJit libraries, but for now this allows saving in Love2D and in the unit test suite. From that, it will be possible to make complex 

```lua
  it("can serialize and deserialize a realistic looking store", function()
    local character = require "game.rules.character"
    local map = require "game.rules.map"

    local state = {
      characters = {
        character.create(),
        character.create(),
        character.create()
      },
      map = map.create { width = 100, 200 }
    }


    local s = serializer.serialize(state)
    local out = serializer.deserialize(s)[1]
    assert.has_no_errors(function() out.map:get_terrain(30, 29) end)
  end)
  ```

  Here's a sample of the unit tests around saving and loading state. Characters and maps are the most complicated
  pieces so wrapping a unit test around these basic entities provide out that the entity was able to save and load properly. Now, this won't cover all scenarios, and likely the save data will break constantly. But now with it available, it will be possible to validate and make sure that it is working as expected.

#### Final Stats
- **51** Files Update
- **1476** Additions
- **168** Deletions
- **(97.926%)** Code Coverage (Increased)