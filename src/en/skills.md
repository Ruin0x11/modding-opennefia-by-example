# Skills

You can define new skills of type `base.skill` and have your character learn
them, alongside tracking their potential and experience.

<span class="filename">Filename: mod/example/data/skill.lua</span>

```lua
data:add {
   _type = "base.skill",
   _id = "embezzlement",

   -- Use "skill" for a regular skill, like Investing.
   -- Use "weapon_proficiency" for a weapon proficiency, like Polearm.
   type = "skill",

   -- The related stat that this skill trains.
   related_skill = "elona.stat_perception",

   -- If true, new characters will be spawned with this skill already learned.
   is_main_skill = false
}
```

Here is a basic overview of getting and modifying skill info for a character.

```lua
> player = Chara.player()
nil
> player:has_skill("example.embezzlement")
false

> inital_level = 10
nil
> player:gain_skill("example.embezzlement", initial_level)
nil
> player:has_skill("example.embezzlement")
true

-- Apply a temporary buff. Cleared when calling player:refresh().
> player:mod_skill_level("example.embezzlement", 15, "add")
nil

-- Get the skill level with buffs applied.
> player:skill_level("example.embezzlement")
25
-- Get the skill level with no buffs applied.
> player:base_skill_level("example.embezzlement")
10

-- Get the skill's potential and experience.
> player:skill_experience("example.embezzlement")
0
> player:skill_potential("example.embezzlement")
100

-- Modify the skill's potential.
> player:mod_skill_potential("example.embezzlement", 50, "add")
nil
> player:skill_potential("example.embezzlement")
150
```

Here is how the skill could be used somewhere in a mod:

```lua
local player = Chara.player()
local target = Chara.find("elona.shopkeeper")
local skill_level = player:skill_level("example.embezzlement")

if skill_level > Rand.rnd(100) then
   Gui.mes("You commit financial fraud.")
   
   local moolah = math.min(skill_level * (Rand.rnd(50) + 50), target.gold)
   target.gold = target.gold - moolah
   player.gold = player.gold + moolah
   
   -- Gain experience according to vanilla's experience gain formula, taking
   -- potential into account.
   local exp_gained = 100
   local related_stat_divisor = 5 -- Optional, applied to trained stat experience
   local chara_exp_divisor = 10 -- Optional, applied to gained character experience
   player:gain_skill_exp("example.embezzlement",
                         exp_gain,
                         related_stat_divisor,
                         chara_exp_divisor)
   
   -- Bypass vanilla's formula and gain a fixed amount of experience.
   player:gain_fixed_skill_exp("example.embezzlement", exp_gain)
end
```

**Note**: Stats are also defined with the skill system, but defining new stats
is not well-supported at the moment. More investigation should be done into how
this could be implemented, if this is a desirable feature.

## Trainers

To make your skill teachable by a trainer, update the `trainer_skills` property
of the map containing the trainer.

<span class="filename">Filename: mod/example/exec/data_edits.lua</span>

```lua
local function make_skill_trainable(town)
   table.insert(town.properties.trainer_skills, "example.embezzlement")
end

data:edit("base.map_archetype", {"elona.derphy"},
      "Add Embezzlement as a trainable skill in Derphy", make_skill_trainable)
```

## Localization

Skills are localized under the namespace `_.base.skill`.

<span class="filename">Filename: mod/example/locale/en/skill.lua</span>

```lua
local skill = {
   embezzlement = {
      name = "Embezzlement",
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
