# Introduction

This book will try to gather various examples of ways to mod OpenNefia in an
easy way for people with little to no programming experience, with sections on
each feature. With this guide you should be able to create your own customized
version of Elona.

This guide will try to keep the explanations to a minimum and focus on providing
various examples of how the engine can be added to. For a more comprehensive
introduction to the engine and a guide on how to set up the source code, you can
check out the official wiki.

## Following Along

You can follow along with the examples in this guide by creating a new mod named
`example` and adding some of the code in each chapter into it. Simply run this
command on the command line to create a new empty mod:

```sh
opennefia mod create example
```

Then, open up `init.lua` and add some code.

```lua
local Log = require("api.Log")

Log.info("Greetings from %s.", _MOD_ID)
```

## Conventions in this guide

### Interactive code examples

When you see code that looks like this:

```lua
> Chara.create("example.my_putit", Map.current())
<map object `base.chara:example.my_putit`>
```

That means this is an invitation to try out the code interactively in the
in-game command prompt, also known as the REPL (meaning
"read-evaluate-print-loop"). You can open the command prompt in-game with the
grave key (`` ` ``) and paste code into it with `Ctrl-V`.

The lines in the code block starting with `>` indicate a command to run, and the
lines that don't start with `>` incidate the result of running the command. So
in the above example, we called `Chara.create()` and the command prompt printed
out the return value of the function call, which in this case is the created
character.

### Mod organization

The convention in this guide is to organize the files in your mod separately and
import them all when it is first loaded. For example, if you see an example
referring to the file `mod/example/data/chara.lua`, then you should also
`require` the file somewhere in your mod, or it won't get loaded.

```lua
require("mod.example.data.chara") -- loads the file `mod/example/data/chara.lua`
```

A common thing to do is to have a file like `mod/example/data/init.lua`, where
you require all the data your mod defines.

<span class="filename">Filename: mod/example/data/init.lua</span>

```lua
require("mod.example.data.chara")
require("mod.example.data.item")
require("mod.example.data.feat")
-- ...
```

Then, require `mod.example.data.init` in your `init.lua`, which is run when the
engine first starts:

<span class="filename">Filename: mod/example/init.lua</span>

```lua
local Log = require("api.Log")

require("mod.example.data.init")

Log.info("Greetings from %s.", _MOD_ID)
```

Note that you don't have to do this for files under the `locale` folder, as
those get loaded automatically by the engine on startup.

> **Tip**: If you see statements like `data:add { ... }` in the example code,
> remember that you can insert a default template with all the required fields
> inserted for you using the editor extension instead of copying it from this
> guide. This is useful in case you end up forgetting which fields are required
> on each definition type.
