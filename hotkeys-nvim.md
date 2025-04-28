# NVim Commands in Normal Mode Starting with `c` and `r`

## Commands Starting with `c`
- `cc` - Change (delete and start insert) the entire current line.
- `cw` - Change (delete and start insert) the current word from the cursor.
- `c$` - Change (delete and start insert) from the cursor to the end of the line.
- `c0` - Change (delete and start insert) from the cursor to the beginning of the line.
- `cG` - Change (delete and start insert) from the cursor to the end of the file.
- `cgg` - Change (delete and start insert) from the cursor to the beginning of the file.
- `ciw` - Change (delete and start insert) the inner word (word under the cursor).
- `ci"` - Change (delete and start insert) inside quotes (double quotes).
- `ci'` - Change (delete and start insert) inside quotes (single quotes).
- `ci(` / `ci)` - Change (delete and start insert) inside parentheses.
- `ci[` / `ci]` - Change (delete and start insert) inside square brackets.
- `ci{` / `ci}` - Change (delete and start insert) inside curly braces.
- `cit` - Change (delete and start insert) inside an HTML/XML tag.
- `cl` - Change (delete and start insert) the character under the cursor.

## Commands Starting with `r`
- `r<char>` - Replace the character under the cursor with `<char>`.
- `rr` - Replace the current line with the next line.
- `R` - Enter replace mode (overwrite text while typing).
- `rx` - Replace the character under the cursor with `x` (example).
- `rc` - Replace the character under the cursor with `c` (example).

## Command mode
- `:r!<cmd>` - Replace the current line with the output of an external command `<cmd>`.
