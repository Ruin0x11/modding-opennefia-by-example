# Items

Items, like characters, are also really important.

Items have type `base.item`. Here is an example of adding a new item.

<span class="filename">Filename: mod/example/data/item.lua</span>

```lua
data:add {
   _type = "base.item",
   _id = "rare_doodad",
   
   -- Image of this item, of type `base.chip`.
   image = "elona.item_crystal",
   
   -- Base value of this item.
   value = 450,
   
   -- Base weight of this item. 1000 means "1.0s".
   weight = 1600,
   
   -- These two parameters control how rare this item is when randomly
   -- generating items. The default formula is:
   -- floor(rarity / (1000 + abs(wanted_level - item_level) * coefficient) + 1)
   rarity = 1000000,
   coefficient = 100,
   
   -- List of gods this item can be offered to. (optional)
   gods = { "elona.jure", "elona.opatos" },
   
   -- Color of this item, specified as `{r, g, b}`. (optional)
   color = { 100, 255, 175 },

   -- The item's categories, of type `base.item_category`. These are often used
   -- for filtering when randomly generating items, and can also be used in
   -- many other ways. For example, items with the `elona.ore` category can be
   -- randomly generated when digging through walls.
   categories = { "elona.ore" },
   
   -- Light source of this item (optional). Use this to make items behave like
   -- the campfire, where getting closer to them makes the screen brighter.
   light = {
       chip = "light_item",
       bright = 35,
       offset_y = 4,
       power = 1,
       flicker = 40,
       always_on = true
   },
}
```

Now you can create your new item in the game world.

```lua
> Item.create("example.rare_doodad", Map.current())
<map object `base.item:example.rare_doodad`>
```

## Localization

Localize your new item under the `_.elona.item` namespace. You can also add some
optional flavor text which gets shown in the item's examine menu if it's fully
identified.

<span class="filename">Filename: mod/example/locale/en/item.lua</span>

```lua
local item = {
   rare_doodad = {
      desc = {
        main = {
          text = "An extremely rare and precious doodad.",
          footnote = "~Category: Ores~"
        },
        _0 = {
          text = "It sells for a lot more than regular doodads.",
          footnote = "~Extra description~"
        }
        -- You can also keep adding entries _1, _2, and so on.
      },
      name = "rare doodad",
      unidentified_name = "a rocky thing" -- optional
   }
}

return {
   _ = {
      elona = {
         item = {
            example = item
         }
      }
   }
}
```

## Item Types

This section will cover some common types of items you might want to add.

You can also check out the prototypes in `mod/elona/data/item.lua` to get a
better idea of how the base game's items are defined. The examples below were
taken directly from there.

### Weapons and Armor

Any item that has a nonzero `dice_x` property and can be equipped in a hand
equipment slot counts as a melee weapon.

```lua
data:add {
   _type = "base.item",
   _id = "long_sword",

   dice_x = 2,
   dice_y = 8,
   hit_bonus = 5,
   damage_bonus = 4,
   pierce_rate = 5,
   equip_slots = { "elona.hand" },
   skill = "elona.long_sword",

   -- Randomly chooses a metallic material like bronze or iron.
   material = "elona.metal",

   image = "elona.item_long_sword",
   level = 1,
   value = 500,
   weight = 1500,
   coefficient = 100,
   categories = {
      "elona.equip_melee_long_sword",
      "elona.equip_melee"
   }
}
```

Any item that has at a nonzero `dice_x` property and can be equipped in a
ranged/ammo equipment slot property counts as a ranged weapon/ammo.

```lua
data:add {
   _type = "base.item",
   _id = "long_bow",

   dice_x = 2,
   dice_y = 7,
   hit_bonus = 4,
   damage_bonus = 8,
   material = "elona.metal",
   equip_slots = { "elona.ranged" },
   pierce_rate = 20,

   -- Ranged weapons and ammo must have the same `skill` to be used together, so
   -- you can't for example mix guns with arrows.
   skill = "elona.bow",

   -- Accuracy modifier to apply to this weapon depending on the distance to the
   -- target. The first entry is a distance of 0 tiles, the second 1 tile, and
   -- so on.
   effective_range = { 50, 90, 100, 90, 80, 80, 70, 60, 50, 20 },

   image = "elona.item_long_bow",
   value = 500,
   weight = 1200,
   coefficient = 100,
   gods = { "elona.lulwy" },

   categories = {
      "elona.equip_ranged_bow",
      "elona.equip_ranged"
   }
}

data:add {
   _type = "base.item",
   _id = "arrow",

   dice_x = 1,
   dice_y = 8,
   hit_bonus = 2,
   damage_bonus = 1,
   material = "elona.metal",
   equip_slots = { "elona.ammo" },
   skill = "elona.bow",

   image = "elona.item_bolt",
   value = 150,
   weight = 1200,
   coefficient = 100,

   categories = {
      "elona.equip_ammo",
      "elona.equip_ammo_arrow"
   }
}
```

Any item that has a `dice_x` property of `0` and can be equipped somewhere
counts as armor.

```lua
data:add {
   _type = "base.item",
   _id = "magic_hat",

   pv = 4,
   dv = 6,
   equip_slots = { "elona.head" },

   -- Randomly chooses a soft material like cloth or silk.
   material = "elona.soft",

   image = "elona.item_magic_hat",
   level = 15,
   value = 1400,
   weight = 600,
   coefficient = 100,
   categories = {
      "elona.equip_head_hat",
      "elona.equip_head"
   }
}
```

See the section on [item materials](item_materials.md) for more information on
defining new item material types.

**Note**: The rules regarding what items are melee/ranged/ammo are subject to
change in the future. For example, the `omake_overhaul` variant has a feature
where ranged weapons can also be wielded in a melee equipment slot. The engine's
code would have to be changed in the future to support mods that add special
behaviors like these.

### Magic Items

You can define new potions, rods, scrolls, and spellbooks. Each one has a few
extra properties to set up. Here are some examples taken from the base game.

See the section on [magic](magic.md) for information on defining new spells.

#### Potions

```lua
data:add {
    _type = "base.item",
    _id = "potion_of_healing",

   -- These fields are necessary for unidentified magic items to be generated
   -- with a randomly generated color/name.
    knownnameref = "potion",
    originalnameref2 = "potion",
    has_random_name = true,
    color = "Random",

    on_drink = function(item, params)
        return Magic.drink_potion(item, "elona.heal_critical", 300, params)
    end,

    image = "elona.item_potion",
    value = 3000,
    weight = 120,
    level = 15,
    category = 52000,
    subcategory = 52001,
    rarity = 700000,
    coefficient = 50,

    categories = {
        "elona.drink",
        "elona.drink_potion"
    }
}
```

#### Rods

```lua
data:add {
   _type = "base.item",
   _id = "rod_of_teleportation",

   knownnameref = "staff",
   originalnameref2 = "rod",
   has_random_name = true,
   color = "Random",

   has_charge = true,
   charge_level = 12,

   on_init_params = function(self)
      self.charges = 12 + Rand.rnd(12) - Rand.rnd(12)
   end,

   on_zap = function(self, params)
      return Magic.zap_wand(self, "elona.teleport_other", 100, params)
   end,

   image = "elona.item_rod",
   value = 840,
   weight = 800,
   category = 56000,
   coefficient = 0,

   categories = { "elona.rod" }
}
```

#### Scrolls

```lua
data:add {
   _type = "base.item",
   _id = "scroll_of_incognito",

   knownnameref = "scroll",
   originalnameref2 = "scroll",
   has_random_name = true,
   color = "Random",

   image = "elona.item_scroll",
   value = 3500,
   weight = 20,
   category = 53000,
   rarity = 70000,
   coefficient = 0,

   on_read = function(self, params)
      return Magic.read_scroll(self, {{ _id = "elona.buff_incognito", power = 300 }}, params)
   end,

   categories = { "elona.scroll" }
}
```

#### Spellbooks

```lua
data:add {
   _type = "spellbook_of_return",
   _id = "spellbook_of_return",

   knownnameref = "spellbook",
   originalnameref2 = "spellbook",
   has_random_name = true,
   color = "Random",

   -- Indicates that this item can be recharged.
   has_charge = true,

   -- Controls the number of charges - higher means more charges.
   charge_level = 3,

   -- Set the number of charges when this item is generated.
   on_init_params = function(self)
       self.charges = 3 + Rand.rnd(3) - Rand.rnd(3)
   end,

   on_read = function(self, params)
       return Magic.read_spellbook(self, "elona.spell_return", params)
   end,

   level = 8,
   image = "elona.item_spellbook",
   value = 8900,
   weight = 380,
   category = 54000,
   rarity = 300000,
   coefficient = 0,

   elona_type = "book",
   categories = {
       "elona.spellbook"
   }
}
```

### Food

Food items have the category `item.food`. Any item with this category will show
up in the `Eat` menu. To make the food spoilable, set its material to
`elona.fresh` and specify the `spoilage_hours` property.

To make the food cookable using a cooking tool, specify `params.food_type`. Here
are the currently available food types (these could be made moddable in the
future):

- `elona.bread`
- `elona.egg`
- `elona.fish`
- `elona.fruit`
- `elona.meat`
- `elona.pasta`
- `elona.sweet`
- `elona.vegetable`

To change the food's quality, specify `params.food_quality`. For now it must be
a number between 0 and 9, with 9 being the highest quality. This is what gets
changed when the food is cooked to display the subtext like "charred putit meat"
or "\<Zeome\> the false prophet hamburger", provided that `params.food_type` is
also specified on the item.

```lua
data:add {
   _type = "base.item",
   _id = "green_pea",

   material = "elona.fresh",
   params = { food_type = "elona.vegetable" },
   spoilage_hours = 72,

   image = "elona.item_green_pea",
   value = 260,
   weight = 360,
   coefficient = 100,
   gods = { "elona.kumiromi" },

   categories = {
      "elona.food_vegetable",
      "elona.food"
   }
}
```

### Useable Items

Some items have a "use" action, like the sewing kit or barbecue set. Here is how
to implement this, with the `on_use` callback:

```lua
data:add {
   _type = "base.item",
   _id = "sewing_kit",

   on_use = function(self, params)
      return Crafting.query("elona.sewing", self, params)
   end,

   image = "elona.item_sewing_kit",
   value = 780,
   weight = 500,
   category = 59000,
   coefficient = 100,
   color = Resolver.make("elona.furniture_color"),

   categories = { "elona.misc_item" }
}
```

### Cargo

Cargo items have the `elona.cargo` category.

```lua
data:add {
   _type = "base.item",
   _id = "cargo_whisky",

   cargo_weight = 16000,
   is_cargo = true,
   params = { cargo_quality = 2 },

   image = "elona.item_whisky",
   value = 1400,
   rarity = 600000,
   coefficient = 100,

   categories = { "elona.cargo" }
}
```

### Complex Items

Some items need special behavior defined on them that can't be specified in an
entirely declarative manner. A good example of this - in fact, the only real
example in the vanilla items - is the fruit tree. It has the following
behaviors:

- When generated, it is set up to produce fruits of a random type.
- Bashing it produces a fruit, if there are any left on the tree.
- The number of fruit left on the tree is added to when the map gets restocked.

The below item definition implements these behaviors with callbacks and event
handlers. First, let's take a look at the entire thing, then break down the
interesting parts one at a time.

```lua
data:add {
   _type = "base.item",
   _id = "tree_of_fruits",

   image = "elona.item_tree_of_fruits",
   value = 2000,
   weight = 42000,
   category = 80000,
   rarity = 100000,
   coefficient = 100,
   originalnameref2 = "tree",

   params = {
      fruit_tree_amount = 0,
      fruit_tree_item_id = "elona.apple"
   },

   on_init_params = function(self)
      local FRUITS = {
         "elona.apple",
         "elona.grape",
         "elona.orange",
         "elona.lemon",
         "elona.strawberry",
         "elona.cherry"
      }
      self.params.fruit_tree_amount = Rand.rnd(2) + 3
      self.params.fruit_tree_item_id = Rand.choice(FRUITS)
   end,

   events = {
      {
         id = "base.on_item_renew_major",
         name = "Fruit tree restock",
         priority = 100000,

         callback = function(self)
            if self.params.fruit_tree_amount < 10 then
               self.params.fruit_tree_amount = self.params.fruit_tree_amount + 1
               self.image = "elona.item_tree_of_fruits"
            end
         end
      },
      {
         id = "elona_sys.on_bash",
         name = "Fruit tree bash behavior",
         priority = 100000,

         callback = function(self)
            self = self:separate()
            Gui.play_sound("base.bash1")
            Gui.mes("action.bash.tree.execute", self)
            local fruits = self.params.fruit_tree_amount
            if self:calc("own_state") == "unobtainable" or fruits <= 0 then
               Gui.mes("action.bash.tree.no_fruits")
               return "turn_end"
            end
            self.params.fruit_tree_amount = fruits - 1
            if self.params.fruit_tree_amount <= 0 then
               self.image = "elona.item_tree_of_fruitless"
            end

            local x = self.x
            local y = self.y
            local map = self:current_map()
            if y + 1 < map:height() and map:can_access(x, y + 1) then
               y = y + 1
            end
            Item.create(self.params.fruit_tree_item_id, x, y, {amount=0})

            return "turn_end"
         end
      }
   },

   categories = {
      "elona.tree"
   }
}
```

First, we have the `params` property.

```lua
   params = {
      fruit_tree_amount = 0,
      fruit_tree_item_id = "elona.apple"
   }
```

This table will hold properties specific to this item that don't make sense to
define on all other items. Here we define the number of fruits remaining and the
kind of fruit that will be generated when the player bashes the tree.

Next, we have the `on_init_params` callback:

```lua
   on_init_params = function(self)
      local FRUITS = {
         "elona.apple",
         "elona.grape",
         "elona.orange",
         "elona.lemon",
         "elona.strawberry",
         "elona.cherry"
      }
      self.params.fruit_tree_amount = Rand.rnd(2) + 3
      self.params.fruit_tree_item_id = Rand.choice(FRUITS)
   end
```

This function gets called when the item is first generated to set up randomly
generated properties, like the number of charges on rods or the skill trained by
a textbook. Here, we use it to add some fruits to the tree and set which fruit
gets generated. In this case, `FRUITS` is a list of `base.item` identifiers.

Finally, the meat - or should I say fruit - of the item is contained in the
`events` table:

```lua
   events = {
      {
         id = "base.on_item_renew_major",
         name = "Fruit tree restock",
         priority = 100000,

         callback = function(self)
            if self.params.fruit_tree_amount < 10 then
               self.params.fruit_tree_amount = self.params.fruit_tree_amount + 1
               self.image = "elona.item_tree_of_fruits"
            end
         end
      },
      -- (other events...)
   }
```

This table defines a set of local event handlers that will get bound when the
item is created or loaded from a save. Any data type which backs a map object
can also use this table, such as `base.chara` and `base.feat`.

When the object gets created, the equivalent of this code is run on it:

```lua
if object.proto.events then
   for _, event in ipairs(object.proto.events) do
      object:connect_self(event.id, event.name, event.callback, event.priority or 100000)
   end
end
```

See `api/IEventEmitter.lua` for more details.

It's useful to note that callbacks like `on_init_params` are really just
wrappers around event handlers, which make it easier to just define a function
on the item without having to add the `events` table and set an event priority
and name. Here is the code in the engine that actually implements
`on_init_params`:

```lua
local function apply_item_on_init_params(item, params)
   if item.proto.on_init_params then
      item.proto.on_init_params(item, params)
   end
end
Event.register("base.on_item_init_params", "Default item on_init_params callback", apply_item_on_init_params)
```

That's it. If you *really* wanted to, you could define `on_init_params` itself
as an event handler on the item's `events` table:

```lua
   events = {
      {
         id = "base.on_item_init_params",
         name = "initialize fruit tree parameters",
   
         callback = function(self)
            local fruits = {
               "elona.apple",
               "elona.grape",
               "elona.orange",
               "elona.lemon",
               "elona.strawberry",
               "elona.cherry"
            }
            self.params.fruit_tree_amount = rand.rnd(2) + 3
            self.params.fruit_tree_item_id = rand.choice(fruits)
         end,
      }
   }
```

However, in this case it's more convenient and readable to just define the
`on_init_params` callback. `events` should be used for defining handlers for
uncommon events that don't have callback wrappers like `on_init_params` does.

**Note**: All of this is subject to change, in case a better way of defining
things in the engine is thought of.
