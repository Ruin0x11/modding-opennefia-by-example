# Document Title

## 1000-foot view
The below diagram shows an extremely high-level overview of the engine.

And here's a somewhat more detailed version.


## The main loop

OpenNefia runs on the LÖVE game engine under the hood. LÖVE comes with an update-and-render loop using the `love.update()` and `love.draw()` functions. Inside each function, OpenNefia uses a coroutine to yield from updating to drawing in a loop, and vice versa. In other words, *almost all of the engine and mod code* runs inside a coroutine of some sort. This is what makes code like the following possible:

```lua
-- this runs inside love.update().
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

## Turn Actions
Let's try to understand some of the problems OpenNefia attempts to solve by walking through a common gameplay scenario. What happens when a character throws a potion?

First, we have to understand how the main gameplay loop in OpenNefia works. The most critical code for the main game loop is located in `game/field_logic.lua`.




Although for the purposes of a finished, shipped version of the game this works perfectly fine, it isn't ideal from an extensibility standpoint. Every time you'd want to add a new thing that's throwable, you'd have to insert your own logic in this part of the code to handle what happens when you throw the item.


The problem with this is that OpenNefia needs to be *moddable* to succeed. People who want to create mods should be able to add their own stuff, and toggle their changes on or off, without having to touch the base engine at all. This is historically one of the number one pain points when it comes to "modding" the old-world HSP variants of Elona - every time you wanted to add in a moderately complex feature, you had to edit the base source code and recompile it.

The way that OpenNefia allows you to create your own custom throwable/useable items is through *event hooks*. You define a new item, then attach a *function callback* to it that gets run whenever a character tries to throw that item:


Although this paradigm does make it way easier to mod the game, the tradeoff is a loss of maintainability. Now that a lot of the important logic that determines what happens when some event gets triggered is scattered throughout numerous different event hooks, it can be a challenge to determine *exactly* what happens when a character throws something. Fortunately, OpenNefia comes with a way to print the list of event handlers registered for an object, so you can see what behaviors could potentially get fired.


Although primitive, this debugging feature could potentially be expanded further. With some Lua hackery we might be able to display the exact line where each event handler was defined, so you don't have to go searching the whole codebase for things. This could even be wrapped in an editor extension that would, for example, give you a list of exact source locations of defined event handlers that you could jump between on the fly.

## Editor environment
The editor extensions for Emacs are *essential* to my current development process. I end up using the functionality it provides literally all the time for things like fixing import statements, sending snippets of code to the game, loading libraries into the game's REPL, and most importantly, hotloading in changes:


They make development so much easier for me that it's easy to take them for granted. I think it's a good idea to think about what would happen if you design tooling support specifically tailored for the project you're working on, and how that can accelerate the development process by taking advantage of the things you know about how the system is designed. For example, there's a "killswitch" function included in the extension, `open-nefia-reset-draw-layers`, that pops off any faulty UI layers by sending a command to the engine via a JSON message. Because input polling can get stuck like this if you accidentally break something while poking around, it's a crucial thing to have on hand to prevent from having to restart the engine and needing to get all the way back to where you were.
