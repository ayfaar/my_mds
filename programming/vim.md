# VIM

## Key bindings

- `w` to move word by word
- `b` to move backwards word by word
- `W` to move word by WORD
- `B` to move backwards WORD by WORD
- `e` to jump to the end of a word
- `ge` to jup to the end of the previous word
- `E` is like `e` but operates on WORDS
- `gE` is like `ge` but operates on WORDS
- `f{character}` (find) to move to the next occurrence of a character in a line.
- `F{character}` to find the previous occurrence of a character
- `t{character}` to move the cursor just before the next occurrence of a character (think of `t{character}` of moving your cursor until that character).
- `T{character}` to do the same as `t{character}` but backwards
- `0`: Moves to the first character of a line
- `^`: Moves to the first non-blank character of a line
- `$`: Moves to the end of a line
- `g_`: Moves to the non-blank character at the end of a line
- `}` jumps entire paragraphs downwards
- `{` similarly but upwards
- `CTRL-D` lets you move down half a page by scrolling the page
- `CTRL-U` lets you move up half a page also by scrolling
- `/{pattern}` to search forward
- `?{pattern}` to search backwards
- `n` to go to the next match
- `N` to go to the previous match

```
{count}{command}
```

- `gd` to **g**o to **d** definition of whatever is under your cursor.
- `gf` to **g**o to a **f**ile in an import.
- `gg` to go to the top of the file.
- `{line}gg` to go to a specific line.
- `G` to go to the end of the file.
- `%` jump to matching `({[]})`.

> Mini-refresher: The d command
>
> - d{motion} - delete text covered by motion
>
>   - d2w => deletes two words
>   - dt; => delete until ;
>   - d/hello => delete until hello
>
> - dd - delete line
> - D - delete from cursor until the end of the line

- `cc` changes a complete line
- `C` changes from the cursor until the end on the line

```
{operator}{a|i}{text-object}
```

- `y` (yank): Copy in Vim jargon
- `p` (put): Paste in Vim jargon
- `g~` (switch case): Changes letters from lowercase to uppercase and back. Alternatively, use `gu` to make something lowercase and `gU` to make something uppercase
- `>` (shift right): Adds indentation
- `<` (shift left): Removes indentation
- `=` (format code): Formats code
- `x` is equivalent to `dl` and deletes the character under the cursor
- `X` is equivalent to `dh` and deletes the character before the cursor
- `s` is equivalent to `ch`, deletes the character under the cursor and puts you into Insert mode
- `r` allows you to replace one single character for another. Very handy to fix typos.
- `~` to switch case for a single character