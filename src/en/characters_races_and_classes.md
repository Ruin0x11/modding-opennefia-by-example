# Characters, Races and Classes

Characters are really important. They are defined with entries of type `base.chara`.

## Characters

Here is how to define a new character, and some interesting parameters you can
specify (there are lots more not covered in this example).

<span class="filename">Filename: mod/example/data/chara.lua</span>

```lua

data:add {
   _type = "base.chara",
   _id = "my_putit",
   
   -- Level of this character. Determines how powerful it is when it is
   -- spawned, and in what dungeons it gets spawned.
   level = 1,
   
   -- What alignment this character has.
   -- 
   -- This determines if it will act hostile toward the player on first sight.
   -- - base.enemy: hostile towards the player
   -- - base.citizen: shopkeepers, etc.
   -- - base.neutral: ignores the player, can swap places with them
   -- - base.friendly: acts like an ally
   faction = "base.enemy",
   
   -- Race and class of this character. These determine body parts and base
   -- attributes/skills.
   race = "elona.slime",
   class = "elona.tourist",
   
   -- The character's image, of type `base.chip`. Can be nil to use the
   -- race's default image.
   image = "elona.chara_race_slime",

   -- Color of this character, specified as `{r, g, b}`.
   color = { 255, 155, 155 },
   
   -- These two parameters control how common this character is when randomly
   -- generating characters. The default formula is:
   --   floor(rarity / (500 + abs(chara_level - dungeon_level) * coefficient))
   -- Higher rarity means more abundant. Higher coefficient means less of a
   -- chance to find the item if the dungeon map's level is too far from the
   -- character's level.
   rarity = 80000,
   coefficient = 400,
   
   -- A callback to be run when this character's corpse is eaten.
   on_eat_corpse = function(corpse, params)
      Gui.mes(params.chara.name .. " ate a corpse of " .. corpse.params.chara_id .. ".")
   end
},
```

Now you can create your new character in the game world.

```lua
> Chara.create("example.my_putit", Map.current())
<map object `base.chara:example.my_putit`>
```

In vanilla, there was a feature to add "Custom NPCs", or "CNPCs", by adding some
special text files to a user directory. In OpenNefia, there is no such
distinction - every character is defined the same, and hopefully that means you
should be able to add your own without too much hassle.

### Localization

Localize your character's name under the `_.elona.chara` namespace.

<span class="filename">Filename: mod/example/locale/en/chara.lua</span>

```lua
local chara = {
   my_putit = {
      name = "my putit",
   }
}

return {
   _ = {
      elona = {
         chara = {
            example = chara
         }
      }
   }
}
```

## Races

Most of the interesting parts of each character are controlled by `race` and
`class`. Let's go over these now.

Races have type `base.race`. Here is an example:

<span class="filename">Filename: mod/example/locale/en/race.lua</span>

```lua 
data:add {
   _type = "base.race",
   _id = "my_race",
   
   -- If true, this race will not show up in the character creation process
   -- unless extra races are turned on in the config.
   is_extra = false,

   -- A list of properties that will get copied to the character on creation.
   --
   -- Note that you *could* define these on the `base.chara` entry as well,
   -- if you wanted, and those values would override the ones here.
   properties = {
      -- Chance the creature breeds on a ranch. Higher is a better chance.
      breed_power = 180,

      image = Resolver.make("elona.by_gender",
                            {
                               male = "elona.chara_eulderna_male",
                               female = "elona.chara_eulderna_female"
      }),
      age = Resolver.make("base.between", { min = 16, max = 35 }),
      height = 175,
      gender = Resolver.make("elona.gender", { male_ratio = 52 }),

   },

   -- Skills and stats for this character, and their initial levels, of type
   -- `base.skill`.
   skills = {
      ["elona.stat_life"] = 100,
      ["elona.stat_mana"] = 100,
      ["elona.stat_strength"] = 6,
      ["elona.stat_constitution"] = 7,
      ["elona.stat_dexterity"] = 7,
      ["elona.stat_perception"] = 10,
      ["elona.stat_learning"] = 8,
      ["elona.stat_will"] = 10,
      ["elona.stat_magic"] = 12,
      ["elona.stat_charisma"] = 8,
      ["elona.stat_speed"] = 70,

      ["elona.martial_arts"] = 2,
      ["elona.casting"] = 5,
      ["elona.literacy"] = 3,
      ["elona.magic_device"] = 3,
   },

   -- Traits for this character, and their initial levels, of type `base.trait`.
   traits = {
      ["elona.perm_magic"] = 1,
   },

   -- Body parts for this character, of type `base.body_part`.
   body_parts = {
      "elona.body",
      "elona.body",
      "elona.back",
      "elona.leg"
   },
}
```

See the sections on [skills](skills.md), [traits](traits.md) and [body
parts](body_parts.md) for more details about the various data types here.

### Localization

Localize your new race under the `_.elona.race` namespace.

<span class="filename">Filename: mod/example/locale/en/race.lua</span>

```lua
local race = {
   my_race = {
      name = "My Race",
   }
}

return {
   _ = {
      elona = {
         race = {
            example = race
         }
      }
   }
}
```

## Classes

Defining classes is pretty similar to races. They have type `base.class`.

<span class="filename">Filename: mod/example/locale/en/class.lua</span>

```lua
data:add {
   _type = "base.class",
   _id = "my_class",

   -- If true, this class will not show up in the character creation process
   -- unless extra classes are turned on in the config.
   is_extra = false,

   -- Properties to copy to this character on generation. Same as with races.
   properties = {
      item_type = "elona.1",
      equipment_type = "elona.1"
   },

   -- Base skills/stats of this race. These get added to the ones for the
   -- character's race.
   skills = {
      ["elona.stat_strength"] = 10,
      ["elona.stat_constitution"] = 10,
      ["elona.stat_dexterity"] = 2,
      ["elona.stat_perception"] = 0,
      ["elona.stat_learning"] = 0,
      ["elona.stat_will"] = 3,
      ["elona.stat_magic"] = 0,
      ["elona.stat_charisma"] = 0,
      ["elona.stat_speed"] = 0,

      ["elona.long_sword"] = 6,
      ["elona.short_sword"] = 4,
      ["elona.blunt"] = 6,
      ["elona.axe"] = 6,
      ["elona.polearm"] = 4,
      ["elona.scythe"] = 5,
      ["elona.two_hand"] = 6,
      ["elona.tactics"] = 4,
      ["elona.evasion"] = 5,
      ["elona.healing"] = 5,
      ["elona.medium_armor"] = 4,
      ["elona.heavy_armor"] = 4,
      ["elona.shield"] = 5,
   },

   -- Callback run when a new save is created with this class selected.
   -- (optional)
   on_init_player = function(player)
      Item.create("elona.portable_cooking_tool", nil, nil, {}, player)
   end
}
```

### Localization

Localize your new class under the `_.elona.class` namespace.

<span class="filename">Filename: mod/example/locale/en/class.lua</span>

```lua
local chara = {
   my_class = {
      name = "My Class",
      description = "My custom class."
   }
}

return {
   _ = {
      elona = {
         class = {
            example = chara
         }
      }
   }
}
```
