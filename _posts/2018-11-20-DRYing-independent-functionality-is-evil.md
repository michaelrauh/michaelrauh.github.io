---
layout: post
title:  "DRYing Independent Functionality is Evil"
date:   2018-11-20 20:59:30 -0400
categories: jekyll update
---

# DRY
Don't repeat yourself is a good idea. It means that if you change code to refactor, or even change functionality, you only need to do it in one place. This raises maintainability and makes bugs easier to fix and less likely. It also shrinks the code base. This is a good thing as there is then less to maintain.

# Points of individual configurability
Most refactors that I see people do at the beginning of a new feature are the opposite of what I see at the end of a feature. At the end of a feature, people seem to spend a lot of time DRYing things up and making the logic tidy. This is an admirable practice. The funny thing is, I see a lot of people go on to do the opposite refactor at the beginning of a later feature. The reason for this is that they realize that they need to write new code, and there is no context for that code to run in that is sufficiently isolated from existing code. Therefore the code must be broken apart so that there is a place to put the new feature. This is the stealth goal of the "if is evil" idea. Removing all `if` statements drives architectures that have many, many points of configurability in them so it is easy to add code to the codebase. It is closely related to the Unix philosophy.

# How do I know when I need to DRY something up?
This is an impossible task. They handwavey way to do this is to ask why code is repeating itself. Is it doing it because the code should change together, or is it doing it by coincidence? Is it desirable for there to be a rock solid connection between the two things, or should they be able to drift apart over time?

# An example
Below is an agent from the game Screeps. Screeps is awesome. You should play it. It is making me realize that I am bad at Javascript. The important part here is the mode. setMode is a function that tells it whether it should be harvesting or depositing energy.

```
var modes = require("./modes")
var util = require("./util")

var roleHarvester = {
  run: function(creep, spawn) {
    var mode = setMode(creep, spawn)
    util.approachAndAct(creep, mode)
  }
};

function setMode(creep, spawn) {
  if (creep.carry.energy < creep.carryCapacity) {
    return modes.harvest(creep.room.find(FIND_SOURCES)[0], creep)
  } else {
    return modes.deposit(creep, spawn)
  }
}

module.exports = roleHarvester;
```
Here is a different kind of creep role. It is a room upgrader. If you look at setMode, it looks eerily similar. The only difference is the actual mode that is selected in the else clause. This is clearly a bit of code that needs to be DRY'd up.

```
var modes = require("./modes")
var util = require("./util")

var roleUpgrader = {
    run: function(creep, spawn) {
      util.approachAndAct(creep, setMode(creep, spawn))
    }
};

function setMode(creep, spawn) {
  if (creep.carry.energy < creep.carryCapacity) {
    return modes.harvest(creep.room.find(FIND_SOURCES)[0], creep)
  } else {
    return modes.upgradeRoom(creep, creep.room.controller)
  }
}

module.exports = roleUpgrader;

```
Here is the upgrader creep a couple of commits down the line. There is a difference I hadn't noticed at first. The harvester spends most of its time in harvest mode. Once it flips into deposit mode, it calls deposit only one time. Then it goes back into harvest mode. The upgrader doesn't work this way. It needs to call upgrade many times until it is out of energy. The issue is that this code is called once per tick, so it isn't possible to run a loop. Instead, some state must be held.

```
var modes = require("./modes")
var util = require("./util")

var roleUpgrader = {
    run: function(creep, spawn) {
      setBias(creep)

      util.approachAndAct(creep, setMode(creep, spawn))
    }
};

function setBias(creep) {
  if (creep.carry.energy == creep.carryCapacity) {
    creep.memory.bias = "upgrade"
  }

  if (creep.carry.energy == 0) {
    creep.memory.bias = "harvest"
  }
}

function setMode(creep, spawn) {
  if (creep.memory.bias == "harvest") {
    return modes.harvest(creep.room.find(FIND_SOURCES)[0], creep)
  } else {
    return modes.upgradeRoom(creep, creep.room.controller)
  }
}

module.exports = roleUpgrader;
```
If this functionality had been DRY'd out, I would have ended up essentially undoing the refactor very shortly afterward. While it is arguable that the harvester could work under the bias system that is laid out for the upgrader, this is overkill for the current approach, and is likely to diverge again as soon as a new creep is introduced (probably a builder). On the other hand, `approachAndAct` has been very useful to have split out. It has also introduced a new collaborator (which should not be called util) which shrinks the responsibility of the creep roles. As it stands, the creep roles are mostly responsible for selecting modes. These modes are another bit of code that should move together. All harvesting should benefit from any improvements done to the harvest code so this makes sense to break out.

# Conclusion
DRY is good. Always do it. Except when it is not good. Then it is evil. Never do it.
