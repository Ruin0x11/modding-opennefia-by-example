# Maps, Areas and Nefias

Map generation and management has been considerably overhauled in OpenNefia. It
is now possible to generate maps in many more ways than the original codebase
allowed for.

Our goal in this chapter is to create a static map like the Truce Ground and
create an entrance leading to it in North Tyris. To do this, we'll define the
following:

- A `base.map_archetype`, which defines the properties of the map like its
  geometry and if it's a town or dungeon. From this archetype a complete map (or
  multiple) can be generated.
- A `base.area_archetype`, which defines an "area", a structure which bundles
  one or more maps into a group organized by floor number. Think of an area as a
  complete dungeon with multiple floors and interconnecting stairs.

## Map Archetypes

First, let's define the map and indicate how its geometry should be generated.

<span class="filename">Filename: mod/example/data/map_archetype.lua</span>

```lua
local MapEntrance = require("mod.elona_sys.api.MapEntrance")
local InstancedMap = require("api.InstancedMap")
local Enum = require("api.Enum")

data:add {
   _id = "my_map",
   _type = "base.map_archetype",

   starting_pos = MapEntrance.south,

   properties = {
      music = "elona.ruin",
      types = { "shelter" },
      tileset = "elona.wilderness",
      level = 1,
      is_indoor = false,
      max_crowd_density = 10,
      has_anchored_npcs = true,
      default_ai_calm = 1
   },

   chara_filter = function(map)
      return {
         level = 20,
         quality = Enum.Quality.Normal,
      }
   end

   on_generate_map = function(area, floor)
      local map = InstancedMap:new(10, 10)
      map:clear("elona.grass")
      return map
   end
}
```

Let's break this down:

- `starting_pos` is a function that determines where the player will be placed
  when the map is entered. `MapEntrance` is a library containing common map
  starting positions. Here we set `starting_pos` to `MapEntrance.south`, which
  is a function that will place the player on the southern edge of the map.
  `MapEntrance.south` is defined as follows:
```lua
function MapEntrance.south(map, chara)
   local x = math.floor(map:width() / 2)
   local y = map:height() - 2

   return { x = x, y = y }
end
```
- `properties` is a table of key-value properties to copy to the map when it
  gets created. This will happen automatically after `on_generate_map` returns,
  or you can set it manually in that function by calling
  `map:set_archetype("example.my_map", { set_properties = true })`. Here are
  some of the more important properties that we define:
  + `level` determines how strong the enemies in the map will be, and also
    controls the quality of the items generated in it.
  + `types` determines how this map should be treated by the game. It's kind of
    like the `categories` property of items, in that it controls a wide set of
    behaviors in the engine related to the map. As an example, this is what
    determines if a map counts as an overworld like North Tyris, or a town like
    Vernis.
  + `is_indoor` determines if this map is affected by things like weather. It
    also controls if the map can be exited by moving to its edge.
  + `max_crowd_density` determines how many randomly spawned monsters can occupy
    this map at a time - higher means more monsters.
- `chara_filter` determines what kinds of characters will be randomly generated
  in this map. It gets called every time a character is generated. For example,
  the Tower of Fire uses this callback to spawn nothing but fire-related
  monsters.
- `on_generate_map` is the most important thing here. This function will return
  the fully generated map, given an area and the floor the map was generated on.
  In the above example, all we do is create a new map with a width and height of
  10 tiles, and then set all the tiles in it to grass.
  
### Localization

Here is how to localize a map based on its map archetype, using the
`_.elona.map_archetype` namespace.

<span class="filename">Filename: mod/example/locale/en/map_archetype.lua</span>

```lua
local map_archetype = {
   my_map = {
      name = "My Map"
   },
}

return {
   _ = {
      elona = {
         map_archetype = {
            example = item
         }
      }
   }
}
```


## Area Archetypes

Next, let's package our map into an area so it can be accessed from the world
map.

```lua
data:add {
   _type = "base.area_archetype",
   _id = "my_area",

   -- Image of this area in the world map, of type `base.chip`.
   image = "elona.feat_area_tent",

   -- The maps making up this area, each one a `base.map_archetype`.
   floors = {
      [1] = "elona.my_map"
   },

   -- Which world map and position to place the entrance to this area.
   parent_area = {
      -- Area archetype of the area to place the entrance in.
      _id = "elona.north_tyris",

      -- What floor of the parent area the entrance should be generated in.
      on_floor = 1,

      -- Position of the entrance in the parent map.
      x = 51,
      y = 6,

      -- Floor in this area the entrance should lead to. In this case it leads
      -- to `elona.my_map`.
      starting_floor = 1
   }
}
```

After the area is defined, you should be able to enter North Tyris and see your
new area placed slightly north of the Truce Ground.

### Localization

Here is how to localize an area based on its area archetype, using the
`_.elona.area_archetype` namespace.

<span class="filename">Filename: mod/example/locale/en/area_archetype.lua</span>

```lua
local area_archetype = {
   my_area = {
      desc = "You see an area handcrafted by a modder. You can't help but gravitate towards it out of curiosity.",
      name = "My Area"
   },
}

return {
   _ = {
      elona = {
         area_archetype = {
            example = item
         }
      }
   }
}
```

## Nefias (Random Dungeons)

It's also possible to leverage a `base.area_archetype` to generate new maps
programatically depending on the current floor. As it turns out, this is exactly
how random dungeons - also known as Nefias - are generated.

Let's create a new type of Nefia that only spawns putits. First, we'll create a
map archetype with the spawning logic.

```lua
local MapEntrance = require("mod.elona_sys.api.MapEntrance")
local Rand = require("api.Rand")

data:add {
   _id = "putit_nefia",
   _type = "base.map_archetype",

   starting_pos = MapEntrance.stairs_up,

   properties = {
      types = { "dungeon_tower" },
      is_indoor = true,
      has_anchored_npcs = true,
      default_ai_calm = 0,
   },

   chara_filter = function(map)
      return {
         id = Rand.choice({"elona.putit", "elona.red_putit"})
      }
   end
}
```

Hopefully this should be clear enough from reading the previous sections.
`MapEntrance.stairs_up` places the player on top of the stairs up in the map.

Next, let's create the area archetype, which will handle the dungeon floor
creation logic.

```lua
local DungeonMap = require("mod.elona.api.DungeonMap")
local DungeonTemplate = require("mod.elona.api.DungeonTemplate")

data:add {
   _type = "base.area_archetype",
   _id = "putit_nefia",

   image = "elona.feat_area_tower",

   deepest_floor = 5,

   parent_area = {
      _id = "elona.north_tyris",
      on_floor = 1,
      x = 47,
      y = 9,
      starting_floor = 1
   },

   on_generate_floor = function(area, floor)
      local gen, params = DungeonTemplate.nefia_tower(area, floor, { level = 1 })
      local map = DungeonMap.generate(area, floor, gen, params)
      map:set_archetype("elona.putit_nefia", { set_properties = true })

      for _ = 1, 20 do
         Chara.create("elona.putit", map)
      end

      return map
   end
}
```

We define `example.putit_nefia` as a static dungeon with 5 floors. Each time a
floor is generated, for example by traveling to an ungenerated floor via
stairs, the `on_generate_floor` callback will be called, with the dungeon's area
and the current floor number passed in as arguments. This function will return a
new map for that floor.

You can generate any arbitrary map in this function, but here we opted to re-use
the vanilla dungeon generation logic. This is slightly more complicated than
just generating an empty map as we did before, but what it means is that you can
generate your own random dungeons just like vanilla does and then do whatever
you want to the resulting map, like filling every room with platinum bells or
something.

Basically, here's what using the vanilla dungeon generator boils down to:

1. Call a function in the `DungeonTemplate` API and pass in the area and floor
   provided to you in the `on_generate_floor` callback. Here we call
   `DungeonTemplate.nefia_tower`, which provides the parameters for generating a
   single floor of a "tower"-type Nefia. This function returns two values: a
   function that generates a new dungeon floor, and a set of parameters to be
   used with the dungeon generation logic, like how big the dungeon's rooms
   should be or the strength of the monsters inside.
2. Pass in the area and floor along with the parameters you got back from the
   `DungeonTemplate` API to `DungeonMap.generate()`. What this function does is
   repeatedly call the dungeon generator function you passed in until it
   successfully generates a proper floor of the dungeon. This is because the
   random map generation can sometimes fail, for example if the generator can't
   connect all of the dungeon's rooms together with corridors. After it
   succeeds, it also generates random monsters and loot for the floor, adds some
   features like cobwebs and pots, and connects the stairs inside the floor to
   the previous and next floors in the area.
3. Set the map archetype of the generated map to hook up the putit spawning
   logic we defined earlier.
4. Go to town with changing things in your new dungeon.

### Custom Dungeon Generation Algorithms

If you want to do things like implement a new maze generation algorithm and have
it show up in your custom Nefias, what you would do here is replace the call to
`DungeonTemplate.nefia_tower()` with a function that returns your maze generator
and the dungeon generation parameters for that map, if any, and pass those to
`DungeonMap.generate()`. The following example demonstrates how to do this.

(TODO: finish this section)
