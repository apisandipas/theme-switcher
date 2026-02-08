# Hyprland Theme Switcher

Unified theme management for a Hyprland desktop. Switches Alacritty, Waybar, Rofi, Swaync, Tmux, Neovim, and wallpaper with a single command.

## Themes

| Theme | Style | Accent |
|---|---|---|
| Original | Dark (Rose Pine Moon) | Purple / Blue |
| CatppuccinMocha | Dark | Mauve / Blue |
| CatppuccinLatte | Light | Mauve / Blue |
| Nord | Dark | Frost blue |
| NordLight | Light | Frost blue |
| Gruvbox | Dark | Yellow-green |
| GruvboxLight | Light | Yellow-green |
| SolarizedLight | Light | Teal / Blue |
| TokyoNight | Dark (Storm) | Blue |
| TokyoNightDay | Light | Blue |
| Dracula | Dark | Purple / Pink |
| Everforest | Dark | Green |
| EverforestLight | Light | Green |

## Usage

**GUI** (Rofi picker):

```bash
~/.Themes/scripts/rofi-themeswitcher
```

**CLI**:

```bash
~/.Themes/scripts/theme-switcher apply TokyoNight
~/.Themes/scripts/theme-switcher list
~/.Themes/scripts/theme-switcher current
```

**Restore wallpaper at login** (add to Hyprland config):

```
exec-once = ~/.Themes/scripts/theme-switcher restore-wallpaper
```

## What it does

`theme-switcher apply <name>` copies theme-specific config files into the right places and reloads each application:

| App | Config target | Reload method |
|---|---|---|
| Alacritty | `~/.config/alacritty/alacritty.toml` | Automatic (file watcher) |
| Waybar | `~/.config/waybar/style.css` | `SIGUSR2` signal |
| Rofi | `~/.config/rofi/colors.rasi` | Next launch |
| Swaync | `~/.config/swaync/style.css` | `swaync-client -rs` |
| Wallpaper | N/A | `swww img` with wipe transition |
| Tmux | `~/.tmux-theme.conf` | `tmux source-file` |
| Neovim | `~/.config/nvim/.colorscheme` | `nvr` remote commands |

Each step is skipped gracefully if the application or tool isn't installed.

## Dependencies

### Required

```bash
pacman -S hyprland alacritty waybar rofi-wayland swaync swww tmux neovim
```

### Font

All Waybar, Swaync, and Alacritty configs use **Victor Mono Nerd Font**. Install it or change the font references in `alacritty-base.toml` and every `waybar.css` / `swaync.css`.

```bash
paru -S ttf-victor-mono-nerd
```

### Optional

```bash
paru -S neovim-remote   # Live Neovim theme switching via nvr
```

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/youruser/themes.git ~/.Themes
chmod +x ~/.Themes/scripts/*
```

The project expects to live at `~/.Themes`. The scripts use `$HOME/.Themes/` internally.

### 2. Alacritty

No extra setup needed. The theme-switcher generates `~/.config/alacritty/alacritty.toml` by concatenating `alacritty-base.toml` with the theme's color file. Edit `alacritty-base.toml` for your font, opacity, padding, and keybinding preferences.

### 3. Waybar

The CSS targets these module IDs. Your `~/.config/waybar/config.jsonc` must define them:

```
#workspaces, #custom-clock, #custom-weather, #cpu, #custom-cpu-temp,
#memory, #battery, #custom-updates, #custom-notification, #tray
```

If your Waybar config uses different modules, update the CSS in each theme's `waybar.css` to match.

### 4. Rofi

Your Rofi config needs to import the colors file. Add this to your theme (e.g. `~/.config/rofi/launcher.rasi`):

```css
@import "colors.rasi"
```

The theme-switcher writes to `~/.config/rofi/colors.rasi`.

### 5. Swaync

No extra setup needed beyond having Swaync installed. The theme-switcher replaces `~/.config/swaync/style.css` directly.

### 6. Tmux

Your `~/.tmux.conf` must source the theme file:

```bash
source-file ~/.tmux-theme.conf
```

### 7. Neovim

This is the most involved step. The theme-switcher writes a colorscheme name to `~/.config/nvim/.colorscheme`. Your Neovim config needs to:

1. Install the colorscheme plugins
2. Read `.colorscheme` on startup and apply the right theme

Add a file like `lua/your_name/colorscheme.lua` to your lazy.nvim plugin spec:

<details>
<summary>Example colorscheme.lua (click to expand)</summary>

```lua
-- Read theme from file written by theme-switcher
local function get_colorscheme()
  local theme_file = vim.fn.expand("~/.config/nvim/.colorscheme")
  local file = io.open(theme_file, "r")
  if file then
    local theme = file:read("*l")
    file:close()
    if theme and theme ~= "" then
      return theme
    end
  end
  return "rose-pine-moon" -- fallback
end

local function apply_colorscheme()
  local colorscheme = get_colorscheme()

  -- Handle variants that need special setup
  if colorscheme == "rose-pine-moon" then
    require("rose-pine").setup({ variant = "moon", styles = { transparency = true } })
    colorscheme = "rose-pine"
  elseif colorscheme == "rose-pine-dawn" then
    require("rose-pine").setup({ variant = "dawn", styles = { transparency = true } })
    colorscheme = "rose-pine"
  elseif colorscheme == "gruvbox-light" then
    vim.o.background = "light"
    require("gruvbox").setup({ transparent_mode = true })
    colorscheme = "gruvbox"
  elseif colorscheme == "gruvbox" then
    vim.o.background = "dark"
    require("gruvbox").setup({ transparent_mode = true })
  elseif colorscheme == "nord-light" then
    vim.o.background = "light"
    colorscheme = "nord"
  elseif colorscheme == "nord" then
    vim.o.background = "dark"
  elseif colorscheme == "solarized-light" then
    vim.o.background = "light"
    vim.g.solarized_termtrans = 1
    colorscheme = "solarized"
  elseif colorscheme == "catppuccin-latte" then
    require("catppuccin").setup({ flavour = "latte", transparent_background = true })
    colorscheme = "catppuccin"
  elseif colorscheme == "catppuccin-mocha" then
    require("catppuccin").setup({ flavour = "mocha", transparent_background = true })
    colorscheme = "catppuccin"
  elseif colorscheme == "tokyonight-storm" then
    require("tokyonight").setup({ transparent = true, style = "storm" })
    colorscheme = "tokyonight"
  elseif colorscheme == "tokyonight-day" then
    require("tokyonight").setup({ transparent = true, style = "day" })
    colorscheme = "tokyonight"
  elseif colorscheme == "dracula" then
    require("dracula").setup({ transparent_bg = true })
  elseif colorscheme == "everforest" then
    vim.o.background = "dark"
    vim.g.everforest_transparent_background = 2
    vim.g.everforest_background = "medium"
  elseif colorscheme == "everforest-light" then
    vim.o.background = "light"
    vim.g.everforest_transparent_background = 2
    vim.g.everforest_background = "medium"
    colorscheme = "everforest"
  end

  local ok, err = pcall(vim.cmd.colorscheme, colorscheme)
  if not ok then
    vim.notify("Colorscheme error: " .. colorscheme .. " - " .. err, vim.log.levels.ERROR)
    vim.cmd.colorscheme("default")
  end
end

return {
  { "folke/tokyonight.nvim", lazy = false, priority = 1000 },
  { "rose-pine/neovim", name = "rose-pine", lazy = false, priority = 1000 },
  { "catppuccin/nvim", name = "catppuccin", lazy = false, priority = 1000 },
  { "shaunsingh/nord.nvim", lazy = false, priority = 1000 },
  { "ellisonleao/gruvbox.nvim", lazy = false, priority = 1000 },
  { "ishan9299/nvim-solarized-lua", lazy = false, priority = 1000 },
  { "Mofiqul/dracula.nvim", lazy = false, priority = 1000 },
  {
    "sainnhe/everforest",
    lazy = false,
    priority = 1000,
    config = apply_colorscheme, -- must be on the last plugin
  },
}
```

</details>

### 8. Hyprland keybind (optional)

```
bind = $mainMod, T, exec, ~/.Themes/scripts/rofi-themeswitcher
```

## Adding a new theme

Create a directory under `themes/` with these files:

```
themes/MyTheme/
  alacritty-colors.toml    # Terminal colors (TOML)
  waybar.css               # Full Waybar stylesheet
  rofi.rasi                # 6 color variables: al, bg, bga, fga, fg, ac
  swaync.css               # GTK CSS with @define-color variables
  tmux.conf                # Tmux status bar + pane styling
  nvim.txt                 # Single line: colorscheme name
  wallpaper.png            # Optional (png/jpg/jpeg/webp)
```

Use an existing theme as a template. The theme-switcher discovers themes by scanning `themes/*/` — no registration needed.

If the Neovim colorscheme requires special setup (background, plugin config), add a case in `colorscheme.lua` and in the theme-switcher's `apply_theme()` function.

## File structure

```
~/.Themes/
  scripts/
    theme-switcher          # Main script
    rofi-themeswitcher      # Rofi GUI wrapper
  alacritty-base.toml       # Shared Alacritty config (font, padding, keys)
  .current-theme            # Tracks active theme name
  themes/
    <ThemeName>/
      alacritty-colors.toml
      waybar.css
      rofi.rasi
      swaync.css
      tmux.conf
      nvim.txt
      wallpaper.*
```

## Troubleshooting

**Theme applies but Waybar looks wrong:** Your Waybar config modules don't match the CSS selectors. Check that your `config.jsonc` uses the module IDs listed above.

**Rofi colors not changing:** Make sure your Rofi theme imports `colors.rasi` with `@import "colors.rasi"`.

**Neovim says "missing theme":** Run `:Lazy sync` to install the colorscheme plugins, then restart Neovim.

**Alacritty not reloading:** Verify `live_config_reload = true` in `alacritty-base.toml`.

**Wallpaper not changing:** Check that `swww-daemon` is running (`swww query`). The script starts it automatically, but Wayland session issues can prevent it.

## License

MIT
