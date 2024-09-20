---
title: Better motions in vim
description: How to leverage powerful textobjects and navigate between them to define better motions.
slug: nvim-better-motions
date: 2024-09-20 01:00:00+0000
categories:
    - Neovim
---

In vim (and neovim), textobjects and motions are naturally linked, for instance in normal mode `w` move to the next start of a word, while in operator pending mode `w` denotes the word object, allowing `viw` to select inside the word, and `vaw` to select around the word.

Other builting objects include `p` (a paragraph), `{` (a matching pair of braces) or `"` (a matching pair of quotes).

You can get more information on the builtin textobjects with:

```
:help text-objects 
```

## Powerful textobjects 

More powerful objects can be defined with the [mini.ai](https://github.com/echasnovski/mini.ai) plugin. This plugin allows using regexes, custom object definitions or even the syntax tree of your programming language to define smart objects. For instance, the following line uses the syntax tree to define a function definition object:

```lua
custom_textobjects = {
  f = ai.gen_spec.treesitter({ 
    a = "@function.outer", 
    i = "@function.outer" })
}
```

This code block defines a new textobject `f` that selects a function definition: `vaf` to select around it, and `vif` to select inside it.

## Powerful motions

As of today, mini.ai does not provide a configuration option to define powerful motions to jump to the next/previous text object like `w` in normal mode would. But we can define these motions ourselves.

Mini.ai does expose a lua function to move the cursor the the next/last start/end of a textobject:

```lua
-- "left" or "right"
local direction = "left" 

-- "a" for around and "i" for inside
local ai = "a" 

-- the name of your textobject,
-- here "f" for function as defined above
local k = "f" 

-- "next", "cover_or_next", "prev", "cover_or_prev"
local method = "next" 

MiniAi.move_cursor(direction, ai, k, {search_method=method}
```

We can leverage this function to define a motion grammar:

Let's say for instance that we repurpose the key `t` as our main motion operator:

- `t`: go to next start
- `T`: go to next end
- `tl`: go to prev start
- `Tl`: go to prev end
- `ti`: go to next start (inside)
- `til`: go to prev start (inside)

We can define the following mappins to move to the next/prev start/end of a function:

```lua
-- tf: to next function (around)
vim.keymap.set({"n", "x", "o"}, "tf", function() 
    return MiniAi.move_cursor("left", "a", k, 
    {search_method="next"})
end, {silent=true})

-- Tf: to end of next function (around)
vim.keymap.set({"n", "x", "o"}, "Tf", function() 
    return MiniAi.move_cursor("right", "a", k, 
    {search_method="cover_or_next"})
end, {silent=true})

-- tif: to next function (inside)
vim.keymap.set({"n", "x", "o"}, "tif", function() 
    return MiniAi.move_cursor("left", "i", k, 
    {search_method="next"})
end, {silent=true})

-- Tif: to end of function (inside)
vim.keymap.set({"n", "x", "o"}, "Tif", function() 
    return MiniAi.move_cursor("right", "i", k, 
    {search_method="cover_or_next"})
end, {silent=true})

-- tlf: to last function (around)
vim.keymap.set({"n", "x", "o"}, "tlf", function() 
    return MiniAi.move_cursor("left", "a", k, 
    {search_method="cover_or_prev"})
end, {silent=true})

-- Tlf: to end of last function (around)
vim.keymap.set({"n", "x", "o"}, "Tlf", function() 
    return MiniAi.move_cursor("right", "a", k, 
    {search_method="prev"})
end, {silent=true})

-- tilf: to last function (inside)
vim.keymap.set({"n", "x", "o"}, "tilf", function() 
    return MiniAi.move_cursor("left", "i", k, 
    {search_method="cover_or_prev"})
end, {silent=true})

-- Tilf: to end of last function (inside)
vim.keymap.set({"n", "x", "o"}, "Tilf", function() 
    return MiniAi.move_cursor("right", "i", k, 
    {search_method="prev"})
end, {silent=true})
```


I have opened a [feature request](https://github.com/echasnovski/mini.nvim/issues/1236) to merge these motions inside mini.ai.

