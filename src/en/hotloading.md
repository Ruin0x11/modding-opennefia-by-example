# Hotloading

Hotloading in OpenNefia is a "75% feature." That means that it won't work in all cases, perhaps at most 75% of the time, due to inherit things like state not being preserved. However, the 75% of the time it works translates into saved development time and the ability to poke around at the code and try out the changes you make. Without hotloading, every change to the engine or mods you'd want to make would have been accompanied by a program restart or a compilation cycle.

I believe that one of the most important things that can be said about the engine is this: *hotloading tries to make up for the lack of type support.* If I revisit some code after a few months and have no idea how it works, what the correct input parameters are, and so on, my first instinct is to fire up the in-game REPL and try it out on some example input. The REPL is pretty forgiving of errors, and the engine is intentionally designed to not manipulate global state unless absolutely necessary, so most of the time it's easy feel safe about going ham on some unfamiliar code.

But sometimes things get borked anyway.

## How hotloading works

This feature takes advantage of the fact that nearly all of the important datum in the engine are kept in Lua tables. As a simple example, when a Lua file containing a API is loaded, it should return a table as its final value. This table will be returned as the result of a call to `require()` by other files that use the API. 

When the file is hotloaded, the engine will take the original copy of the table that was created pre-hotloading and overwrite its contents with that of the newly edited file:

Because we're using the original table reference, this means that all the dependent Lua files will "see" the change, as long as they index into the API table to call things:

Of note is that this will not work if the API users instead keep a local reference to things like functions in the table. In this case you'd have to hotload the file using the API as well for the change to be reflected:
