# Introduction

This book will try to gather various examples of ways to mod OpenNefia, with
sections on each feature. With this guide you should be able to create your own
customized version of Elona.

For more details about setting up the OpenNefia repository and some more
fundamental information about the engine, you can read the official wiki.

## Following Along

You can follow along by creating a new mod named `example` and adding some of
the code in each chapter. Simply run this command to create a new empty mod:

```sh
opennefia mod new example
```

### Conventions for organizing mods

The convention in this guide is to organize files separately and require them
all. For example, if you see a doc referring to the file
`mod/example/data/chara.lua`, then you should also `require` the file somewhere
in your mod, or it won't get loaded.

```lua
require("mod.example.data.chara")
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

Then, require `mod.example.data.init` in your `init.lua`:

<span class="filename">Filename: mod/example/init.lua</span>

```lua
local Log = require("api.Log")

require("mod.example.data.init")

Log.info("Greetings from %s.", _MOD_ID)
```

Also, if you see statements like `data:add { ... }` in the docs, remember that
you can insert a default template using the editor extension instead of copying
it from this guide, if you end up forgetting which fields are required.
