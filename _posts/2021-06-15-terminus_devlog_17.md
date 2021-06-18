---
title: Terminus Devlog 17
layout: post
image: https://www.dropbox.com/s/n9l705127syfri1/postimage-devlog17.png?raw=1
excerpt: Dice Mechanics, Updated Milestones ![Video](https://www.dropbox.com/s/1qjb6004nlodfmp/terminus_devlog_17.gif?raw=1)
---

> ## Items Completed
> - [#107 - Refactor Map state to be a simple data store and that accessing it is handle directly by selectors](https://github.com/tredfern/terminus/issues/107)
> - [#126 - Full Screen Panel, Panel, Section title should have actions/toolbar area](https://github.com/tredfern/terminus/issues/126)
> - [#103 - Split out Inventory data from Character State](https://github.com/tredfern/terminus/issues/103)
> - [#128 - Skills should be tracked in separate rules section](https://github.com/tredfern/terminus/issues/128)
> - [#130 - Build the "official" attributes and skill check system](https://github.com/tredfern/terminus/issues/130)
> - Custom thunk assertions

## Dice and Randomization Mechanics

The biggest change was to rework the dice mechanics. Originally, I had not given much thought and went with an idea of what I wanted, but it seemed like it was going to be a too complicated system. I simplified things down significantly and went with the following:

`2d6 + modifiers >= 8`

Every skill check, attack role, etc... will follow this formula.

* If the task is more challenging, add negative modifiers
* If the task trivial, add positive modifiers
* If the character is skilled, add positive modifiers
* If the opposing creature resists, add negative modifiers

### Implementation

```lua
-- excerpt from game/rules/skills/actions.lua
function Actions.taskCheck(modifiers, successAction, failAction)
  return Thunk(Actions.types.TASK_CHECK, function(dispatch)
    local roll = skillCheckCup() -- 2d6
    local modScore = tables.sum(modifiers)
    dispatch(MessageLog.actions.add(Messages.skills.taskCheck, { roll = roll, modifiers = modScore }))
    if roll + modScore >= TARGET_NUMBER then
      dispatch(successAction)
    else
      dispatch(failAction)
    end
  end)
end
```

The modifiers is a table list of numbers to add/subtract from the roll. Creating selectors to pull these
values together will be the primary approach to isolate what modifiers impact an action.

For any task check that gets created, provide the modifiers for the action and the actions that should
be performed on success or failure. It's possible that a failure or success does nothing, which
is straightforward, just pass in `nil` or `{}`.

The action can be literally anything which allows the skills system to be streamlined an unconcerned
with circumstances.


### Long term considerations

There needs to be a way to apply temporary modifiers to characters. Such as startled, afraid, hungry, etc...
These modifiers might impact all skill checks or maybe the impact a subset of skills. As the complexity of 
the game expands those situations will need to be considered more in depth.

## Streamlining the Project

Milestones for the project have been modified to make more sense and remove some useless refactoring work that
was planned. 

## Statistic Report
<pre>
───────────────────────────────────────────────────────────────────────────────
Tag: devlog-17
───────────────────────────────────────────────────────────────────────────────
Files Update:        <strong>156</strong>
LOC Added:           <strong>2057</strong>
LOC Deleted:         <strong>1511</strong>
Code Coverage:       <strong>97%</strong>

───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Lua                        623     28835     3970      2870    21995       1422
Markdown                    22      1236      323         0      913          0
ReStructuredText            13       584      180         0      404          0
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
Total                      679     32911     4640      2916    25355       1427
───────────────────────────────────────────────────────────────────────────────
</pre>

[All the changes](https://github.com/tredfern/terminus/compare/devlog-17...devlog-16)

![Video](https://www.dropbox.com/s/1qjb6004nlodfmp/terminus_devlog_17.gif?raw=1)