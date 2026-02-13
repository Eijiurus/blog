---
title: '[Log] My Linux Post-Installation Setup.'
published: 2026-01-01
draft: false
lang: en
description: Transform a fresh Linux installation into a powerful development workstation. From shell customization (Zsh/Bash) to essential dev tools, discover how to configure the environment for maximum productivity and efficiency.
abbrlink: linux-setup
---

## TODOs:
- [x] yazi
- [x] Kitty
- [x] Zsh
- [ ] tmux
- [ ] lazygit
- [ ] Nvim
- [ ] Terminal tools(fzf, bat, fdfind, zoxide)

# Linux Toolchain

My daily-driver setup on KDE Plasma / Ubuntu. Everything lives under `~/.config/`.

---

## Terminal: Kitty

```
~/.config/kitty/
├── kitty.conf             # main config (font, window, cursor, tabs, highlights)
├── keyboard.conf          # all key bindings (included from kitty.conf)
├── current-theme.conf     # active color theme (symlink / include)
└── install.sh             # one-click font installer (Maple Mono + Symbols Nerd Font)
```

### Prerequisite

A bundled `install.sh` downloads and installs the two required fonts:

1. **Maple Mono NF CN** -- main font with CJK support
2. **Symbols Nerd Font Mono** -- icon glyphs

```sh
cd ~/.config/kitty && chmod +x install.sh && ./install.sh
```

After installing, press `Ctrl+Shift+F5` to reload.


### Key Bindings

All bindings live in `keyboard.conf`. Standard `Ctrl+Shift` combos throughout.

**Tabs**:

| Key | Action |
|-----|--------|
| `ctrl+shift+t` | New tab (inherits cwd) |
| `ctrl+shift+w` | Close tab |
| `ctrl+shift+←/→` | Switch tabs |
| `ctrl+shift+./,` | Move tab forward / backward |
| `ctrl+shift+alt+t` | Rename tab |
| `ctrl+1-9` | Jump to tab by number |

**Windows**:

| Key | Action |
|-----|--------|
| `ctrl+shift+enter` | New window (inherits cwd) |
| `ctrl+shift+q` | Close window |
| `ctrl+shift+[/]` | Previous / next window |
| `ctrl+shift+f/b` | Move window forward / backward |
| `ctrl+shift+r` | Resize window |
| `ctrl+shift+l` | Cycle layouts |

**Scroll**:

| Key | Action |
|-----|--------|
| `ctrl+shift+↑/↓` | Scroll line by line |
| `ctrl+shift+pgup/pgdn` | Scroll page |
| `ctrl+shift+home/end` | Top / bottom |
| `ctrl+shift+h` | Open scrollback in pager |

**Font size**:

| Key | Action |
|-----|--------|
| `ctrl+shift+=/-` | Increase / decrease |
| `ctrl+shift+0` | Reset |

**Clipboard**:

| Key | Action |
|-----|--------|
| `ctrl+shift+c/v` | Copy / paste |
| `ctrl+shift+s` | Paste from selection |

**Hints & Misc**:

| Key | Action |
|-----|--------|
| `ctrl+shift+e` | Open URLs with hints |
| `ctrl+shift+p>f` | Pick file path |
| `ctrl+shift+p>l` | Pick line |
| `ctrl+shift+p>w` | Pick word |
| `ctrl+shift+f5` | Reload config |
| `ctrl+shift+f2` | Edit config |
| `ctrl+shift+f1` | Open kitty docs |

`Ctrl+Shift+M` toggles regex markers that color-code log levels inline:

| Marker | Pattern | Color |
|--------|---------|-------|
| 1 | `INFO` | Blue (`#74c7ec`) |
| 2 | `WARNING` | Yellow (`#F9E2AF`) |
| 3 | `ERROR` / `FATAL` | Red (`#F38BA8`) |

## Terminal Shell: Zsh

```
~/.config/zsh/
├── .zshrc             # main interactive shell config
├── .zimrc             # Zim module list
├── .zprofile          # login shell (PATH for pipx)
├── fzf.zsh            # fzf key-bindings + completion (sourced from .zshrc)
└── .zim/              # Zim home (plugins, init.zsh, zimfw.zsh)
    └── modules/       # auto-installed Zim modules
```

All zsh config lives under `ZDOTDIR=~/.config/zsh` (set in `/etc/zsh/zshenv` or `~/.zshenv`).

**Framework**: [Zim](https://zimfw.sh/). Modules are declared in `.zimrc` and auto-installed on shell start.

**Modules (`.zimrc`)**

```
environment         # sane defaults
zsh-completions     # extra completions
completion          # tab completion engine
git                 # git aliases (G-prefixed: Gws, Gco, Gp, etc.)
input               # keybindings, bracketed paste
termtitle           # dynamic terminal title
utility             # colored ls, grep aliases
git-info            # git status in prompt
duration-info       # command duration tracking
prompt-pwd          # shortened cwd
magicmace           # minimal prompt theme
zsh-autosuggestions # fish-style suggestions
zsh-history-substring-search # up/down history filtering
fast-syntax-highlighting     # real-time syntax coloring
zfm                 # fuzzy bookmark manager (fzf-based)
fzf-tab             # fzf-powered tab completion
```


### Module Configuration

```zsh
ZSH_AUTOSUGGEST_MANUAL_REBIND=1           # skip auto-rebind on precmd (faster)
ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets) # highlight commands + bracket matching
```

**Aliases**

```sh
alias cat='batcat'
alias fd='fdfind'
```

### Plugins

**fzf-tab**

Replaces zsh's default completion menu with [fzf](https://github.com/junegunn/fzf). Press `Tab` as usual -- fzf takes over automatically.

| Key | Action |
|-----|--------|
| `Tab` | Trigger fzf completion |
| `Ctrl+Space` | Toggle multi-select |
| `F1` / `F2` | Switch completion groups |
| `/` | Continuous completion (deep paths) |
| `Enter` | Accept selection |

```bash
disable-fzf-tab    # temporarily disable, fall back to default
enable-fzf-tab     # re-enable
toggle-fzf-tab     # toggle on/off
```

**fzf shell keybindings**

Sourced from `fzf.zsh`, which bundles fzf's native key-bindings and completion scripts. These work independently of fzf-tab.

| Key | Action |
|-----|--------|
| `Ctrl+R` | Fuzzy search command history |
| `Ctrl+T` | Fuzzy search files/dirs, insert into command line |
| `Alt+C` | Fuzzy search dirs and cd into selection |
| `<Tab>` | Trigger path completion (e.g. `vim <Tab>`, `kill -9 <Tab>`) |

**zfm (Zsh Fuzzy Marks)**

A bookmark manager for files and directories, built on fzf.

| Key | Action |
|-----|--------|
| `Ctrl+P` | Fuzzy select a bookmarked directory and cd into it |
| `Ctrl+O` | Fuzzy select bookmarks and insert path(s) into command line |

```bash
zfm add ~/Projects ~/.config       # bookmark directories or files
zfm list                            # list all bookmarks
zfm list --files                    # file bookmarks only
zfm list --dirs                     # directory bookmarks only
zfm edit                            # edit bookmarks in $EDITOR
f <pattern>                         # jump to matching bookmarked directory
```

Bookmarks are stored in `~/.zfm.txt`.

---

## File Manager: Yazi

```
~/.config/yazi/
├── yazi.toml          # main config (layout, sorting, openers, preview)
├── keymap.toml        # key bindings (vim-style navigation)
├── theme.toml         # colors and borders
├── init.lua           # plugin initialization
├── package.toml       # plugin dependencies (ya pack)
├── starship.toml      # starship prompt config for header
├── bookmark            # saved bookmarks (yamb)
└── plugins/           # installed plugins
```

**Launch**: Shell wrapper `y` (defined in `.zshrc`) launches yazi and `cd`s to its CWD on exit. From Neovim, press `R` to open yazi in a full-screen floating window via `yazi.nvim`.

### Key Bindings

**Navigation**:

| Key | Action |
|-----|--------|
| `h` / `l` | Parent dir / enter dir or open file (smart-enter) |
| `j` / `k` | Next / previous file |
| `H` / `L` | History back / forward |
| `J` / `K` | Seek preview down / up 5 lines |
| `gg` / `G` | Top / bottom of list |
| `<C-u>` / `<C-d>` | Half-page up / down |
| `gh` / `gc` / `gd` | Go to `~` / `~/.config` / `~/Downloads` |
| `z` / `Z` | Jump via fzf / zoxide |

**File operations**:

| Key | Action |
|-----|--------|
| `<Space>` | Toggle selection |
| `<C-a>` | Select all |
| `v` / `V` | Visual mode / unset mode |
| `y` / `x` | Yank (copy) / yank (cut) |
| `p` / `P` | Paste / paste (overwrite) |
| `d` / `D` | Trash / permanent delete |
| `a` | Create file (append `/` for directory) |
| `r` | Rename |
| `.` | Toggle hidden files |

**Search & filter**:

| Key | Action |
|-----|--------|
| `f` | Filter files |
| `F` | Jump to a file / directory via fzf |
| `/` / `?` | Find next / previous |
| `s` / `S` | Search by name (fd) / by content (rg) |
| `,` + key | Sort (`,a` alpha, `,s` size, `,m` mtime, etc.) |

**Copy paths**:

| Key | Action |
|-----|--------|
| `cc` / `cd` / `cf` / `cn` | Copy path / dirname / filename / name without ext |

**Tabs**:

| Key | Action |
|-----|--------|
| `t` | New tab with CWD |
| `1`–`9` | Switch to tab N |
| `[` / `]` | Previous / next tab |
| `{` / `}` | Swap tab with previous / next |
| `<C-c>` | Close current tab |

**Bookmarks (yamb)**:

| Key | Action |
|-----|--------|
| `ba` | Add bookmark |
| `bg` / `bG` | Jump by key / by fzf |
| `bd` / `bD` | Delete by key / by fzf |
| `br` / `bR` | Rename by key / by fzf |
| `bA` | Delete all bookmarks |

**Other**:

| Key | Action |
|-----|--------|
| `;` / `:` | Shell command / shell command (blocking) |
| `<C-g>` | Launch lazygit |
| `<Tab>` | Spot hovered file |
| `ms` / `mp` / `mm` | Line mode: size / permissions / mtime |
| `~` | Open help |
| `q` / `Q` | Quit (update CWD) / quit (keep CWD) |

### Plugins

| Plugin | Effect |
| :--- | :--- |
| [git.yazi](https://github.com/yazi-rs/plugins) | Show git status indicators (modified, staged, untracked) alongside files |
| [smart-enter.yazi](https://github.com/yazi-rs/plugins) | `l`/`Enter` enters directories or opens files -- dual-action smart key |
| [yamb.yazi](https://github.com/h-hg/yamb) | Bookmark manager -- save, jump, delete, rename bookmarks with `b` prefix keys |
| [yaziline.yazi](https://github.com/llanosrocas/yaziline) | Custom statusline with curvy separators and select/yank icons |
| [full-border.yazi](https://github.com/yazi-rs/plugins) | Draw borders around all panes for cleaner visual separation |
| [starship.yazi](https://github.com/Rolv-Apneseth/starship) | Starship prompt in the header bar (aws, gcloud, lua modules disabled) |

**Neovim Integration (yazi.nvim)**

Yazi is opened from Neovim via [yazi.nvim](https://github.com/mikavilpas/yazi.nvim). Press `R` to launch a full-screen floating window with no border. Opening a directory with `nvim .` automatically starts Yazi.

| Key (inside floating window) | Action |
|-----|--------|
| `<C-v>` | Open in vertical split |
| `<C-x>` | Open in horizontal split |
| `<C-t>` | Open in new tab |
| `<C-f>` | Grep in directory |
| `<C-r>` | Replace in directory |
| `<Tab>` | Cycle open buffers |
| `<C-y>` | Copy relative path |
| `<C-q>` | Send to quickfix list |
| `<F1>` | Show help |

---

## Editor: Neovim

```
~/.config/nvim/
```

**Structure**:

```
init.lua                 # entry point
lua/config/
  defaults.lua           # vim.opt settings
  keymaps.lua            # custom bindings
  plugins.lua            # lazy.nvim bootstrap
lua/plugins/             # plugin specs (auto-imported)
  core.lua               # telescope, treesitter, which-key
  lsp.lua                # mason, lspconfig
  ui.lua                 # colorscheme, notifications
  editor.lua             # illuminate, autopairs, colorizer, ufo, move
  mini.lua               # mini.ai, mini.icons, mini.surround
  telescope.lua          # telescope config + fzf
  fzf.lua                # fzf-lua grep
  statusline.lua         # lualine
  terminal.lua           # toggleterm
  git.lua                # gitsigns, lazygit
  tabline.lua            # bufferline
  tex.lua                # vimtex (zathura viewer)
  typst.lua              # typst support
  ...
```

**Plugin manager**: [lazy.nvim](https://github.com/folke/lazy.nvim) (~44 plugins)

**Colorscheme**: nvim-deus

**LSP servers** (via Mason): clangd, lua_ls, pylsp, ruff, texlab, tinymist, ts_ls, marksman

### Key Settings

| Setting | Value |
|---------|-------|
| Leader | Space |
| Tabs | Hard tabs, width 4 |
| Line numbers | Hybrid (absolute + relative) |
| Clipboard | System (unnamedplus) |
| Folding | Indent-based, all open by default |
| Splits | Right / below |
| Search | Smart case, live preview |

### Key Bindings

| Key | Action |
|-----|--------|
| `S` | Save |
| `Q` | Save + quit |
| `jk` | Exit insert mode |
| `;` | Command mode |
| `H` / `L` | Line start / end |
| `J` / `K` | 5 lines down / up |
| `<C-p>` | Find files (telescope) |
| `<C-f>` | Global grep (fzf-lua) |
| `<C-g>` | Lazygit |
| `<C-\>` | Toggle floating terminal |
| `<leader>ff` | Find files |
| `<leader>fg` | Live grep |
| `<leader>fb` | Buffers |
| `<leader>e` | File tree |
| `<leader>d` | Diagnostics |
| `gd` | Go to definition |
| `gD` | Go to definition (split) |
| `<Tab>` / `<S-Tab>` | Next / prev buffer |
| `<leader>x` | Close buffer |
| `s{motion}` | Substitute with yanked text |
| `sa` / `sd` / `sr` | Surround add / delete / replace |
| `gcc` | Toggle comment |
| `H` (git) | Preview hunk |

---


## Multiplexer: tmux

```
~/.config/tmux/tmux.conf
```

**Prefix**: `Ctrl-a` (rebound from default Ctrl-b)

| Setting | Value |
|---------|-------|
| Mouse | On |
| Base index | 1 |
| Escape time | 0 (for nvim) |
| True color | Enabled (`terminal-overrides`) |
| Theme | Catppuccin Mocha |

**Pane splitting**: `|` horizontal, `-` vertical

**Pane navigation**: `prefix + h/j/k/l` (vim-style)

**Config reload**: `prefix + r`

**Plugins** (TPM):

```
tmux-plugins/tpm              # plugin manager
tmux-plugins/tmux-sensible    # sensible defaults
christoomey/vim-tmux-navigator # Ctrl-h/j/k/l across tmux + nvim
catppuccin/tmux               # theme (mocha)
```

The vim-tmux-navigator plugin makes `Ctrl-h/j/k/l` seamlessly switch between tmux panes and neovim splits.

---

## Git TUI: Lazygit

```
~/.config/lazygit/config.yml   # empty (defaults)
```

Running with all defaults. Accessed primarily through nvim via `<C-g>` (lazygit.nvim plugin).

---

## Other Tools

| Tool | Command | Replaces | Core Function | Basic Usage Example |
| :--- | :--- | :--- | :--- | :--- |
| zoxide | `z` | `cd` | Smart Directory Jumper: Navigates to directories based on frequency and recency without full paths. | `z pro`<br>(Jumps to `/home/user/projects`) |
| ripgrep | `rg` | `grep` | Fast Text Search: Recursively searches files for text/regex patterns. Respects `.gitignore` by default. | `rg "main" src/`<br>(Finds "main" inside files in `src/`) |
| bat | `bat` / `batcat`* | `cat` | Enhanced File Viewer: Displays file contents with syntax highlighting, line numbers, and Git integration. | `bat config.json`<br>(Pretty-prints the JSON file) |
| fd | `fd` / `fdfind`* | `find` | Fast File Finder: Locates files/directories on the filesystem with simple syntax. Respects `.gitignore`. | `fd logo`<br>(Finds files named "logo" recursively) |

:::tip
On Ubuntu/Debian, `bat` is installed as `batcat` and `fd` is installed as `fdfind` due to naming conflicts.
:::

---
## Summary

| Tool | Config Location | Theme/Style |
|------|----------------|-------------|
| Kitty | `~/.config/kitty/` | Catppuccin Mocha |
| Zsh | `~/.config/zsh/` | Zim + magicmace prompt |
| KDE Plasma | `~/.config/kde*` | Apple-Ventura-Dark + Gruvbox icons |
| Clash-Verge | `~/.config/clash-verge/` | Rule-based proxy |
| Chrome | `~/.config/google-chrome/` | Default |
| Neovim | `~/.config/nvim/` | nvim-deus, lazy.nvim |
| Yazi | `~/.config/yazi/` | Starship header, yaziline statusline |
| tmux | `~/.config/tmux/` | Catppuccin Mocha |
| Lazygit | `~/.config/lazygit/` | Defaults |
