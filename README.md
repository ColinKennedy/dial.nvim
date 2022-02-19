# dial.nvim

**NOTICE: This plugin is work-in-progress yet. User interface is subject to change without notice.**

## FOR USERS OF THE PREVIOUS VERSION (v0.2.0)

This plugin was released v0.3.0 on 2022/xx/xx and is no longer compatible with the old interface.
If you have configured the settings for previous versions, please refer to [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) and reconfigure your settings.

## Abstract

Extended increment/decrement plugin for [Neovim](https://github.com/neovim/neovim). Written in Lua.

![demo.gif](https://github.com/monaqa/dial.nvim/wiki/fig/dial-demo.gif)

## Features

* Increment/decrement various number and other things
  * n-ary (`2 <= n <= 36`) integers
  * date
  * constant (toggle a specific string, such as a keyword or operator)
    * `true` ⇄ `false`
    * `&&` ⇄ `||`
    * `a` ⇄ `b` ⇄ ... ⇄ `z`
  * hex colors
  * semantic version
* Support `<C-a>` / `<C-x>` / `g<C-a>` / `g<C-x>` in VISUAL mode
* Flexible configuration of increment/decrement targets
  * rules that are valid only for specific FileType
  * rules that are valid only in VISUAL mode
* Support counter
* Support dot repeating (without overriding the behavior of `.`)

## Similar plugins

* [tpope/vim-speeddating](https://github.com/tpope/vim-speeddating)
* [Cycle.vim](https://github.com/zef/vim-cycle)
* [AndrewRadev/switch.vim](https://github.com/AndrewRadev/switch.vim)

## Installation

This plugin requires Neovim `>=0.5.0` (`>=0.6.1` is recommended).
You can install it by following the instructions of your favorite package manager.

## Usage

This plugin does not provide or override any default key-mappings.
To use this plugin, assign the plugin key-mapping to the key you like, as shown below:

```vim
nmap <C-a> <Plug>(dial-increment)
nmap <C-x> <Plug>(dial-decrement)
vmap <C-a> <Plug>(dial-increment)
vmap <C-x> <Plug>(dial-decrement)
vmap g<C-a> g<Plug>(dial-increment)
vmap g<C-x> g<Plug>(dial-decrement)
```

Or you can configure it with Lua as follows:

```lua
vim.api.nvim_set_keymap("n", "<C-a>", require("dial.map").inc_normal(), {noremap = true})
vim.api.nvim_set_keymap("n", "<C-x>", require("dial.map").dec_normal(), {noremap = true})
vim.api.nvim_set_keymap("v", "<C-a>", require("dial.map").inc_visual(), {noremap = true})
vim.api.nvim_set_keymap("v", "<C-x>", require("dial.map").dec_visual(), {noremap = true})
vim.api.nvim_set_keymap("v", "g<C-a>", require("dial.map").inc_gvisual(), {noremap = true})
vim.api.nvim_set_keymap("v", "g<C-x>", require("dial.map").dec_gvisual(), {noremap = true})
```

## Configuration

In this plugin, flexible increment/decrement rules can be set by using **augend** and **group**,
where **augend** represents the target of the increment/decrement operation,
and **group** represents a group of multiple augends.

```lua
local augend = require("dial.augend")
require("dial.config").augends:register_group{
  -- グループ名を指定しない場合に用いられる被加数
  default = {
    augend.integer.alias.decimal,     -- nonnegative decimal number (0, 1, 2, 3, ...)
    augend.integer.alias.hex,         -- nonnegative hex number  (0x01, 0x1a1f, etc.)
    augend.date.alias["%Y/%m/%d"],  -- date (2022/02/19)
  },

  -- `mygroup` というグループ名を使用した際に用いられる被加数
  mygroup = {
    augend.integer.alias.decimal,
    augend.constant.alias.bool,    -- boolean value (true <-> false)
    augend.date.alias["%m/%d/%Y"],  -- boolean value (true <-> false)
  }
}
```

* To define a group, use the `augends:register_group` function in the `"dial.config"` module.
  The arguments is a dictionary whose keys are the group names and whose values are the list of augends.
* Various augends are defined `"dial.augend"` by default.

To specify the group of augends, you can use **expression register** ([`:h @=`](https://neovim.io/doc/user/change.html#quote_=)) as follows:

```
"=mygroup<CR><C-a>
```

If it is tedious to specify the expression register for each increase or decrease, you can "map" it:

```vim
nmap <Leader>a "=mygroup<CR><Plug>(dial-increment)
```

Alternatively, you can set the same mapping without expression register:

```lua
vim.api.nvim_set_keymap("n", "<Leader>a", require("dial.map").inc_normal("mygroup"), {noremap = true})
```

When you don't specify any group name in the way described above, the addends in the `default` group is used instead.

### Example Configuration

```vim
lua << EOF
local augend = require("dial.augend")
require("dial.config").augends:register_group{
  default = {
    augend.integer.alias.decimal,
    augend.integer.alias.hex,
    augend.date.alias["%Y/%m/%d"],
  },
  typescript = {
    augend.integer.alias.decimal,
    augend.integer.alias.hex,
    augend.constant.new{ elements = {"let", "const"} },
  },
  visual = {
    augend.integer.alias.decimal,
    augend.integer.alias.hex,
    augend.date.alias["%Y/%m/%d"],
    augend.constant.alias.alpha,
    augend.constant.alias.Alpha,
  },
}

-- change augends in VISUAL mode
vim.api.nvim_set_keymap("v", "<C-a>", require("dial.map").inc_normal("visual"), {noremap = true})
vim.api.nvim_set_keymap("v", "<C-x>", require("dial.map").dec_normal("visual"), {noremap = true})
EOF

" enable only for specific FileType
autocmd FileType typescript lua vim.api.nvim_buf_set_keymap(0, "n", "<C-a>", require("dial.map").inc_normal("typescript"), {noremap = true})
```

## List of Augends

For simplicity, we define the variable `augend` as follows.

```lua
local augend = require("dial.augend")
```

### `integer`

`n`-based integer (`2 <= n <= 36`). You can use this rule with `augend.integer.new{ ...opts }`.

```lua
require("dial.config").augends:register_group{
  default = {
    -- uppercase hex number (0x1A1A, 0xEEFE, etc.)
    augend.integer.new{
      radix = 16,
      prefix = "0x",
      natural = true,
      case = "upper",
    },
  },
}
```

## `date`

Date and time. You can use this rule with augend alias described below.

## `constant`

Predefined sequence of strings. You can use this rule with `augend.constant.new{ ...opts }`.

```lua
require("dial.config").augends:register_group{
  default = {
    -- uppercase hex number (0x1A1A, 0xEEFE, etc.)
    augend.constant.new{
      elements = {"and", "or"},
      word = true, -- if false, "sand" is incremented into "sor", "doctor" into "doctand", etc.
      cyclic = true,  -- "or" is incremented into "and".
    },
    augend.constant.new{
      elements = {"&&", "||"},
      word = false,
      cyclic = true,
    },
  },
}
```

### `hexcolor`

RGB color code such as `#000000` and `#ffffff`.

```lua
require("dial.config").augends:register_group{
  default = {
    -- uppercase hex number (0x1A1A, 0xEEFE, etc.)
    augend.hexcolor.new{
      case = "lower",
    },
  },
}
```

### `semver`

Semantic versions. You can use this rule by augend alias described below.

It differs from a simple nonnegative integer increment/decrement in these ways:

* When the cursor is before the semver string, the patch version is incremented.
* When the minor version is incremented, the patch version is reset to zero.
* When the major version is incremented, the minor and patch versions are reset to zero.

### `user`

Custom augends.

```lua
require("dial.config").augends:register_group{
  default = {
    -- uppercase hex number (0x1A1A, 0xEEFE, etc.)
    augend.user.new{
      find = require("dial.augend.common").find_pattern("%d+"),
      add = function(text, addend, cursor)
          local n = tonumber(text)
          n = math.floor(n * (2 ^ addend))
          text = tostring(n)
          cursor = #text
          return {text = text, cursor = cursor}
      end
    },
  },
}
```

## Augend Alias

Some augend rules are defined as alias. It can be used directly without using `new` function.

```lua
require("dial.config").augends:register_group{
  default = {
    augend.integer.alias.decimal,
    augend.integer.alias.hex,
    augend.date.alias["%Y/%m/%d"],
  },
}
```

|Alias Name                                |Explanation                                      |Examples                           |
|------------------------------------------|-------------------------------------------------|-----------------------------------|
|`augend.integer.alias.decimal`            |decimal natural number                           |`0`, `1`, ..., `9`, `10`, `11`, ...|
|`augend.integer.alias.decimal_int`        |decimal integer (including negative number)      |`0`, `314`, `-1592`, ...           |
|`augend.integer.alias.hex`                |hex natural number                               |`0x00`, `0x3f3f`, ...              |
|`augend.integer.alias.octal`              |octal natural number                             |`0o00`, `0o11`, `0o24`, ...        |
|`augend.integer.alias.binary`             |binary natural number                            |`0b0101`, `0b11001111`, ...        |
|`augend.date.alias["%Y/%m/%d"]`           |Date in the format `%Y/%m/%d` (`0` padding)      |`2021/01/23`, ...                  |
|`augend.date.alias["%m/%d/%Y"]`           |Date in the format `%m/%d/%Y` (`0` padding)      |`23/01/2021`, ...                  |
|`augend.date.alias["%d/%m/%Y"]`           |Date in the format `%d/%m/%Y` (`0` padding)      |`01/23/2021`, ...                  |
|`augend.date.alias["%m/%d/%y"]`           |Date in the format `%m/%d/%y` (`0` padding)      |`01/23/21`, ...                    |
|`augend.date.alias["%d/%m/%y"]`           |Date in the format `%d/%m/%y` (`0` padding)      |`23/01/21`, ...                    |
|`augend.date.alias["%m/%d"]`              |Date in the format `%m/%d` (`0` padding)         |`01/04`, `02/28`, `12/25`, ...     |
|`augend.date.alias["%-m/%-d"]`            |Date in the format `%-m/%-d` (no paddings)       |`1/4`, `2/28`, `12/25`, ...        |
|`augend.date.alias["%Y-%m-%d"]`           |Date in the format `%Y-%m-%d` (`0` padding)      |`2021-01-04`, ...                  |
|`augend.date.alias["%Y年%-m月%-d日"]`     |Date in the format `%Y年%-m月%-d日` (no paddings)|`2021年1月4日`, ...                |
|`augend.date.alias["%Y年%-m月%-d日(%ja)"]`|Date in the format `%Y年%-m月%-d日(%ja)`         |`2021年1月4日(月)`, ...            |
|`augend.date.alias["%H:%M:%S"]`           |Time in the format `%H:%M:%S`                    |`14:30:00`, ...                    |
|`augend.date.alias["%H:%M"]`              |Time in the format `%H:%M`                       |`14:30`, ...                       |
|`augend.constant.alias.ja_weekday`        |Japanese weekday                                 |`月`, `火`, ..., `土`, `日`        |
|`augend.constant.alias.ja_weekday_full`   |Japanese full weekday                            |`月曜日`, `火曜日`, ..., `日曜日`  |
|`augend.constant.alias.bool`              |elements in boolean algebra (`true` and `false`) |`true`, `false`                    |
|`augend.constant.alias.alpha`             |Lowercase alphabet letter (word)                 |`a`, `b`, `c`, ..., `z`            |
|`augend.constant.alias.Alpha`             |Uppercase alphabet letter (word)                 |`A`, `B`, `C`, ..., `Z`            |
|`augend.semver.alias.semver`              |Semantic version                                 |`0.3.0`, `1.22.1`, `3.9.1`, ...    |


If you don't specify any settings, the following augends is set as the value of the `default` group.

* `augend.integer.alias.decimal`
* `augend.integer.alias.hex`
* `augend.date.alias["%Y/%m/%d"]`
* `augend.date.alias["%Y-%m-%d"]`
* `augend.date.alias["%m/%d"]`
* `augend.date.alias["%H:%M"]`
* `augend.constant.alias.ja_weekday_full`

## Changelog

See [HISTORY](./HISTORY.md).

## Testing

This plugin uses `PlenaryBustedDirectory` in [`plenary.nvim`](https://github.com/nvim-lua/plenary.nvim).
