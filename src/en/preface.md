# Preface

Since rewriting a game's entire engine is a lot of work, there were at least a few things I wanted to accomplish to make it worth the time:

1. Add a mod system to the game which would allow adding new game data and features without having to edit and recompile the base source code as before.
2. Redesign the engine so it's easier to work with, eliminating almost all global state, and provide a unified interface for mods to extend what the base engine already provides.
3. Have fun.

The third is pretty important since I end up getting sucked into this project in fits of passion lasting a few months at a time. This is a project that isn't betting on anything like a mass migration from the original's playerbase. It just exists. There are no real feature guarantees beyond what's provided by the vanilla engine. All I hope is that I can eventually add all the things from the different versions of the game that I found most interesting to me, and perhaps some brand-new things that are now easier to implement.

I don't come from a background of game design. I just like building things. The thing about Elona is that it's already a decently-designed game (well, if you don't mind grinding occasionally). The main issue is that when I wanted to do more, the state of the codebase was a limiting factor. Not just the fact that it was written in a bastardized dialect of BASIC with no local variables, but the fact that this choice of language caused a number of severe limitations in the higher level design of the engine.

- No official support for any other operating systems besides Windows.
- You can only have one map available to manipulate at a time, meaning shopkeepers have to overwrite the global list of items temporarily to show their stock to the player.
- No ability to do things like script custom dialogs or have create custom item behaviors without editing the source code. Many variants do have a custom item system, but for some reason nobody thought to add a scripting layer for creating your own custom item behaviors.
- User interfaces designed around copious usage of `goto`.
- Many, many functions tied to global state that didn't have to be.

Most of these things are qualities specific to the ease of developing the game, as opposed to just playing it. But after a long time of figuring those issues could be worked around and trying to hack it anyway, I finally got burnt out and instead tried to see what rewriting the whole engine *might* look like if it were finished at some point.


- Unified interface for mods to add new data definitions.
- Support for code hotloading.
- In-game command line where you can execute any code the engine or mods provide.
- Support for localization in languages besides English.
- Support for any operating system that's supported by LÖVE.

I admit that half of these things are just niceties for people working on the codebase or developing mods, and have no relevance whatsoever to somebody who just downloads the game and plays it. But those are the tradeoffs I chose to make. Seeing the path the engine has taken by now, I personally find it hard to imagine writing it in a different way. The biggest downside for me is the lack of types.
