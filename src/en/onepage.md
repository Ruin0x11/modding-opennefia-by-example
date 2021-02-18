# Document Title

## 1000-foot view
The below diagram shows an extremely high-level overview of the engine.

And here's a somewhat more detailed version.


## The main loop

OpenNefia runs on the LÖVE game engine under the hood. LÖVE comes with an update-and-render loop using the `love.update()` and `love.draw()` functions. Inside each function, OpenNefia uses a coroutine to yield from updating to drawing in a loop, and vice versa. In other words, *almost all of the engine and mod code* runs inside a coroutine of some sort. This is what makes code like the following possible:

```lua
-- this runs inside love.update(dt).
local draw_callback_1 = Anim.load("elona.anim_smoke", x, y)
local draw_callback_2 = Anim.load("elona.anim_smoke", x, y)

Gui.add_async_draw_callback(draw_callback_1)
Gui.add_async_draw_callback(draw_callback_2)

Gui.wait_for_draw_callbacks()

Gui.mes("Finished drawing.")
```

In this example, `Gui.wait_for_draw_callbacks()`, running inside `love.update(dt)`, would call `coroutine.yield()` internally at a 60FPS cadence to yield over to `love.draw()`, which would render the animations to the screen. `love.draw()` would then itself yield back to `love.update(dt)`, which would continue to update the time elapsed for each callback until they finish. Both animations would be rendered to the screen at the same time. After both animations have finished playing through, code execution resumes after `Gui.wait_for_draw_callbacks()` and the message `"Finished drawing."` is printed.

A more common usage of the coroutine system is found in code like the following:

```lua
Gui.mes("You like putits, don't you?")

local prompt = Prompt:new({
   "Scut!",
   "Rut."
})
local result = prompt:query() -- <== this right here

if result.index == 1 then
   Gui.mes("I knew it!")
else
   Gui.mes("Aww.")
end
```

`:query()` refers to the extremely important method `IUiLayer:query()`, and `Prompt` is a thing that implements `IUiLayer`. This and the rest of the UI layer system is examined in depth below, but basically what this does is enter a series of loops for displaying something like a GUI window where `coroutine.yield()` will get called between the `love.draw()` and `love.update(dt)` functions, polling for keybinds and doing the rest of the boring game loop bookkeeping you'd expect.

The usage of coroutines like this means you don't have to worry about things like update-draw loops, and can focus on what the intent of the code actually is.

*In vanilla:* There is no distinction between drawing and updating in the HSP version of Elona. Drawing code like `gcopy` would run alongside code like `await` that refreshes the screen and polls for keypresses. This makes sense since HSP's graphics are primarily based around modifying bitmap buffers, not clearing and redrawing the screen each frame.

## Map Objects and Ownership
In trying to best model the state of the vanilla engine's game world, while also designing the architecture to be futureproof, the below design has been converged on. It can be tricky to understand at first.

1. There is one currently loaded map, `field.map`, of type `InstancedMap`. This single global is what contains almost all of the game object state.
2. `InstancedMap` implements the interface `ILocation`, meaning it can contain things that implement `IMapObject`. "Map object" is OpenNefia's terminology for game objects with a 2D tile position. Each map object contains an immutable field, `location`, that points to the `ILocation` that contains it.
3. Changing the position of a map object means using a method on `ILocation`, `:move_object(x, y)`, which updates the object's position inside its `location`. Implementers of `ILocation` can optionally declare that they're "positional" with the method `:is_positional()`, meaning that they will keep track of objects on an arbitrarily sized 2D tile grid. Otherwise, methods like `:move_object(x, y)` become no-ops.
4. `ILocation:iter()` iterates through every object the location contains. For the purposes of deserialization, it is **critical** that this contract is followed, or the deserializer will not be able to restore the `location` backreference on each map object that it contains. For example, since characters can have *both* an inventory and equipment slots, both of which hold map objects that need to be properly deserialized, the implementation of `IChara:iter()` returns an iterator that chains both the iterators of the inventory and equipment slots together.
5. Map objects can *themselves* implement `ILocation`. This is used for things like characters, which have inventories of items, and items that act like containers for other items.

*In vanilla:* Flat global arrays are used to keep track of the pool of all objects. All the objects in each map and character's inventory are held in a massive `inv(ci, x)` array, and the maximum number of items for each character and map are hardcoded. For example, index 0-400 could be the list of items on the map, 401-600 could be the inventory of the player character, 601-620 could be for the next character, and so on. However, it was a highly desired feature to be able to remove these arbitrary limits, something which would be impossible to do with the original engine's code without rewriting a lot of the inventory management logic.

Also, the rendering code for each "thing on a 2D grid" was specialized for each type of object. You had the arrays `cdata` for characters, `inv` for items, `mef` for map effects, and `map(6, x, y)` for map features. None of them shared anything in common in the rendering or update code except the fact that they had an X and Y position. In OpenNefia, any object that can be displayed on the map's grid can have its own rendering logic, and is updated and displayed in a uniform way. You could in theory create an entirely new object type that uses this system (but there's no real point since map features can already satisfy such a need).

## Turn Actions
Let's try to understand some of the problems OpenNefia attempts to solve by walking through a common gameplay scenario. What happens when a character throws a potion?

First, we have to understand how the main gameplay loop in OpenNefia works. The most critical code for the main game loop is located in `game/field_logic.lua`.




Although for the purposes of a finished, shipped version of the game this works perfectly fine, it isn't ideal from an extensibility standpoint. Every time you'd want to add a new thing that's throwable, you'd have to insert your own logic in this part of the code to handle what happens when you throw the item.


The problem with this is that OpenNefia needs to be *moddable* to succeed. People who want to create mods should be able to add their own stuff, and toggle their changes on or off, without having to touch the base engine at all. This is historically one of the number one pain points when it comes to "modding" the old-world HSP variants of Elona - every time you wanted to add in a moderately complex feature, you had to edit the base source code and recompile it.

The way that OpenNefia allows you to create your own custom throwable/useable items is through *event hooks*. You define a new item, then attach a *function callback* to it that gets run whenever a character tries to throw that item:


Although this paradigm does make it way easier to mod the game, the tradeoff is a loss of maintainability. Now that a lot of the important logic that determines what happens when some event gets triggered is scattered throughout numerous different event hooks, it can be a challenge to determine *exactly* what happens when a character throws something. Fortunately, OpenNefia comes with a way to print the list of event handlers registered for an object, so you can see what behaviors could potentially get fired.


Although primitive, this debugging feature could potentially be expanded further. With some Lua hackery we might be able to display the exact line where each event handler was defined, so you don't have to go searching the whole codebase for things. This could even be wrapped in an editor extension that would, for example, give you a list of exact source locations of defined event handlers that you could jump between on the fly.

## Serialization
Saving and loading the engine's data is absolutely one of OpenNefia's biggest weaknesses, and it needs to be improved further if the game is to be stable enough to use in practice.

At present, serialization works as follows:
1. Take the top-level globals, like the current map, and filter them through a special version of `table.deepcopy` that ignores some hardcoded fields like backreferences to parent objects or internal OOP bookkeeping things. This is a really terrible way of going about this, and I wish it were better.
2. Punt the stripped table through [binser]().
3. Compress the thing and save it to disk.
4. Later on, load it from disk, decompress it, and restore things that couldn't be serialized like backreferences, fields pointing to functions like event handlers, and OO class metatables.
5. Pray to Ehekatl that nothing breaks.

Real-world serialization performance on a save from actual gameplay isn't well-tested at present, and that's very worrying. Most of the engine is tested through the lens of porting individual features in a "laboratory" environment intended for isolated testing. A full playthough of the game would certainly be more enlightening, and more likely than not, disheartening, for seeing if the performance is satisfactory. If we can at least aim for similar serialization performance to vanilla while also adding on all the neat and weird things that OpenNefia provides, I'd call it a win.

A few other details about serialization: the OO system allows you to define `:serialize()` and `:deserialize()` methods that control what happens when a class instance gets saved to disk. These are useful for removing things like references to stuff from `data` or map objects that aren't desirable to save. The serializer is configured to restore all the class metatables when the data gets loaded again, thanks to built-in support from binser.

What is sorely needed is a good paradigm for saving references to map objects on arbitrary class instances, like an `origin` field on potion puddles that points to the character that originally threw the potion for purposes of aggro. Currently serialization of map object references is only well-supported for map objects contained in something implementing `ILocation`, like `InstancedMap` or `EquipSlots`. Maybe a weak wrapper around an `ILocation` and an UID that can be lazily loaded would work. I have to mention that there's a *lot* of hackish code surrounding the serialization of map objects, mainly involving the restoration of the `location` field pointing to the object's containing `ILocation`. See `api.MapObject` for details - if you're some kind of masochist, that is.

## Editor environment
OpenNefia fires up a debug server on port 4567 when it starts, which accepts and returns JSON messages. This is used for communication with an external editor process for all sorts of debugging goodies.

The Emacs editor extension for the engine is *essential* to my current development process. I end up using the functionality it provides literally all the time for things like fixing import statements, sending snippets of code to the game, loading libraries into the game's REPL, and most importantly, hotloading in changes:


It makes development so much easier for me that it's easy to take it for granted. I think it's a good idea to consider the benefits of designing developer tooling specifically tailored for the project you're working on. You can imagine the ways those tools could accelerate the development process by taking advantage of the things you know about how the system is designed. For example, there's a "killswitch" function included in the extension, `open-nefia-reset-draw-layers`, that pops off any faulty UI layers by sending a command to the engine via a JSON message. Because input polling can get stuck if you accidentally break something while poking around, it's a crucial thing to have on hand to prevent from having to restart the engine and needing to get all the way back to where you were.
