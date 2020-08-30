# Books

You can add new books that get randomly generated in dungeons and the like. This
feature was ported from the old `books.txt` format.

Book items have the item ID `elona.book`. The text and title of the book item is
controlled by the `params.book_id` property, which is an ID of an `elona.book`
data entry.

```lua
local book = Item.create("elona.book", Map.current())

data["elona.book"]:ensure(book.params.book_id)

Gui.mes(book.params.book_id) -- "elona.my_diary"
```

Localized text for books lives under the `_.elona.book` namespace. First, add a
new entry for your book there.

<span class="filename">Filename: mod/example/locale/en/book.lua</span>

```lua
local book = {
   my_book = {
      title = "My Book",
      text = [[
Here is the text of my book.

...

Go long!
]]
   }
}

return {
   _ = {
      elona = {
         book = {
            example = book
         }
      }
   }
}
```

Next, add an entry for your book in `data` of type `elona.book`.

<span class="filename">Filename: mod/example/data/book.lua</span>

```lua
data:add {
   _type = "elona.book",
   _id = "my_book",
   
   -- If false, this book will not be randomly generated in dungeons and such.
   is_randomly_generated = true
}
```

Now your book can be randomly generated in dungeons, or you can create it
yourself.

```lua
> book = Item.create("elona.book", Chara.player())
nil
> book
<map object `base.item:elona.book`>
> book.params.book_id = "example.my_book"
nil
```

To get the text of a book, use `data["elona.book"]:localize(_id)`:

```lua
local localized = data["elona.book"]:localize("example.my_book")

Gui.mes(localized.title) -- "My Book"
Gui.mes(localized.text) -- "Here is the text of my book. <...>"
```
