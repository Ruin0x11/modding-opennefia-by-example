# Skills

You can define new skills of type `base.skill` and have your character learn
them, alongside tracking their potential and experience.

<span class="filename">Filename: mod/example/data/skill.lua</span>

```lua
data:add {
   _type = "base.skill",
   _id = "embezzling",

   -- Use "skill" for a regular skill, like Investing.
   -- Use "weapon_proficiency" for a weapon proficiency, like Polearm.
   type = "skill",

   -- The related stat that this skill trains.
   related_skill = "elona.stat_perception",

   -- If true, new characters will be spawned with this skill already learned.
   is_main_skill = true
}
```

Here is a basic overview of getting and modifying skill info for a character.

```lua
local player = Chara.player()

player:has_skill("example.embezzling") -- false

local inital_level = 10
player:gain_skill("example.embezzling", initial_level)
player:has_skill("example.embezzling") -- true

-- Apply a temporary buff. Cleared when calling player:refresh().
player:mod_skill_level("example.embezzling", 15 "add")

-- Get the skill level with buffs applied.
local skill_level = player:skill_level("example.embezzling") -- 25
-- Get the skill level with no buffs applied.
local base_skill_level = player:base_skill_level("example.embezzling") -- 10

-- Get the skill's potential and experience.
local skill_exp = player:skill_experience("example.embezzling")
local skill_potential = player:skill_potential("example.embezzling")

-- Modify the skill's potential.
player:mod_skill_potential("example.embezzling", 50, "add")

if skill_level > Rand.rnd(100) then
   Gui.mes("You commit financial fraud.")
   
   -- Gain experience according to vanilla's experience gain formula, taking
   -- potential into account.
   local exp_gained = 100
   local related_stat_divisor = 5 -- Optional, applied to trained stat experience
   local chara_exp_divisor = 10 -- Optional, applied to gained character experience
   player:gain_skill_exp("example.embezzling",
                         exp_gain,
                         related_stat_divisor,
                         chara_exp_divisor)
   
   -- Bypass vanilla's formula and gain a fixed amount of experience.
   player:gain_fixed_skill_exp("example.embezzling", exp_gain)
end
```

Stats are also defined with the skill system, but defining new stats is not
well-supported at the moment. More investigation should be done if this is a
desirable feature.

## Trainers

To make your skill teachable by a trainer, update the `trainer_skills` property
of the map containing the trainer.

<span class="filename">Filename: mod/example/exec/data_edits.lua</span>

```lua
local function make_skill_trainable(town)
   table.insert(town.properties.trainer_skills, "example.embezzling")
end

data:edit("base.map_archetype", {"elona.derphy"},
      "Add Embezzling as a trainable skill", make_skill_trainable)
```

## Localization

Skills are localized under the namespace `_.base.skill`.

<span class="filename">Filename: mod/example/locale/en/skill.lua</span>

```lua
local skill = {
   embezzling = {
      name = "Embezzling",
      description = "Commit financial fraud, for fun and profit."
   }
}

return {
   _ = {
      elona = {
         skill = {
            example = skill
         }
      }
   }
}
```
