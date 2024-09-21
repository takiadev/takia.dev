---
title: "Vim: boost the power of the n/N keys!"
description: Here is how to remap n/N keys to improve consistency accross vim motions and free keys on your keyboard
slug: vim-better-n-N-keys
date: 2024-09-20 05:00:00+0000
categories:
    - Neovim
---

\[EDIT\] Thanks to reddit for pointing mistakes in the first version of this article. 

Have you ever noticed how the vim keymap seems sometimes arbitrary or unbalanced? Out of all possible motions, words have three keys: `w`, `b` and `e`. Not only that's a lot of real estate on your keyboard, but one could argue that in the age of modern programming moving by word is far from optimal. With the rise of so many great plugins, new actions such as [jumping to a pair of character](https://github.com/folke/flash.nvim), [swapping two objects](https://github.com/mizlan/iswap.nvim/), [changing surroundings](https://github.com/echasnovski/mini.surround) are better candidates for our keymap. And with the introduction of [treesitter](https://github.com/nvim-treesitter/nvim-treesitter-textobjects) it is now possible to move by function, by variable, by loop, by variable name, ...by anything you can think of that is more appropriate than the old notion of a "word".

It is natural, then, to wonder if we can do better: is there a way that we can free up some space on the keyboard while retaining the power of vim motions? And how can we incorporate modern motions such as jumping from function to function or jumping straight to the next for-loop?

## The power of n/N cords

My answer to these considerations is to change the behavior of the `n` and `N` keys. After all, there are four keys to do essentially the same thing but with slight variations: 
- `n`: next search result, 
- `;`: next char-wise search result, 
- `N`: previous search result, 
- `,`: previous char-wise search result.

And like we said earlier, that's a lot of real estate on the keyboard.

> In this article we will argue to remap `n` and `N` but it works just as well if you decide to remap `;` and `,` instead. Actually, I would even find it more convenient as `;,` don't require `shift`!

The idea here is to use `n` to mean `next <something>` and `N` to mean `previous <something>`. For the search results, we can use `n/` and `N/`:

```
nnoremap n/ n
nnoremap N/ N
```

Or in lua:

```lua
vim.keymap.set('n', 'n/', 'n', { noremap = true })
vim.keymap.set('n', 'N/', 'N', { noremap = true })
```

For words we can use `nw` (next word) and `Nw` (previous word) which frees up the `b` key for something else! And also frees up the `w` key in normal mode for something else.

```
nnoremap nw w
nnoremap Nw b
```

Or in lua:

```lua
vim.keymap.set('n', 'nw', 'w', { noremap = true })
vim.keymap.set('n', 'Nw', 'b', { noremap = true })
```

> Note: Thank you [reddit](https://www.reddit.com/r/neovim/comments/1fltduc/better_mappings_for_the_n_and_p_keys/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) for your feedback! If remaping one of the main vim key worries you, you can still apply the same ideas to another key combination such as `]` and `[`. So `]w` would be next word and `[w` would be previous word, which frees-up the `w` and `b` keys. But personally, I find them too far from the main row to be nice to use.

What about end of word? I like `ne` for `next end <something>` and `Ne` for `previous end <something>`. This means that next end of word is mapped to `new`, and previous end of word is mapped to `New`:

```
nnoremap new e
nnoremap New ge
```

Or in lua:

```lua
vim.keymap.set('n', 'new', 'e', { noremap = true })
vim.keymap.set('n', 'New', 'ge', { noremap = true })
```

Now, we could keep going and remap every builtin motion, but we can do better by leveraging the power of advanced textobjects.

## Advanced textobjects

Thanks to [treesitter](https://github.com/nvim-treesitter/nvim-treesitter) and plugins such as [treesitter-textobjects](https://github.com/nvim-treesitter/nvim-treesitter-textobjects) or [mini.ai](https://github.com/echasnovski/mini.ai) we can define more useful textobjects than words and paragraphs.

For instance, using the mini.ai plugin, you can define a function textobject that will match every function in your code file as follows (this is lua code):

```lua
ai = require("mini.ai")
-- ...
custom_textobjects = {
  f = ai.gen_spec.treesitter({ 
    a = "@function.outer", 
    i = "@function.inner" })
}
```

Using this new textobject, we can:
- Select inside a function: `vif`,
- Select around a function: `vaf`,
- Select inside the next function: `vinf`,
- Select inside the previous function: `vipf`.

We can also use our new `n`/`N` keys to move to the next start of a function (`nf`), the next end of a function (`nef`), the previous start of a function (`Nf`) or the previous end of a function (`Nef`).

I have opened a [feature request](https://github.com/echasnovski/mini.nvim/issues/1236) to merge these motions inside mini.ai but for now we can define them ourselves as follows:


```lua
local MiniAi = require("mini.ai")

-- nf: to next function (around)
vim.keymap.set({"n", "x", "o"}, "nf", function() 
    return MiniAi.move_cursor("left", "a", k, 
    {search_method="next"})
end, {silent=true})

-- nef: to end of next function (around)
vim.keymap.set({"n", "x", "o"}, "nef", function() 
    return MiniAi.move_cursor("right", "a", k, 
    {search_method="cover_or_next"})
end, {silent=true})

-- nif: to next function (inside)
vim.keymap.set({"n", "x", "o"}, "nif", function() 
    return MiniAi.move_cursor("left", "i", k, 
    {search_method="next"})
end, {silent=true})

-- neif: to end of function (inside)
vim.keymap.set({"n", "x", "o"}, "neif", function() 
    return MiniAi.move_cursor("right", "i", k, 
    {search_method="cover_or_next"})
end, {silent=true})

-- pf: to prev function (around)
vim.keymap.set({"n", "x", "o"}, "Nf", function() 
    return MiniAi.move_cursor("left", "a", k, 
    {search_method="cover_or_prev"})
end, {silent=true})

-- pef: to end of prev function (around)
vim.keymap.set({"n", "x", "o"}, "Nef", function() 
    return MiniAi.move_cursor("right", "a", k, 
    {search_method="prev"})
end, {silent=true})

-- pif: to prev function (inside)
vim.keymap.set({"n", "x", "o"}, "Nif", function() 
    return MiniAi.move_cursor("left", "i", k, 
    {search_method="cover_or_prev"})
end, {silent=true})

-- peif: to end of prev function (inside)
vim.keymap.set({"n", "x", "o"}, "Neif", function() 
    return MiniAi.move_cursor("right", "i", k, 
    {search_method="prev"})
end, {silent=true})
```

## The sky is the limit

The only limit is your imagination. There are so many motions that come the in prev/next form that we can free up a lot of space on the keyboard, and reallocate it to our most used verbs or motions.

Here are some ideas, and leave a comment if you have new ideas!

- Next char: `nnoremap nc f` and Previous char: `nnoremapp Nc F`,
- Go to previous change: `nnoremap N. g;` and next change: `nnoremap n. g,`,


