# Theme Switcher

Unified theme management for Hyprland desktop. Switches themes across multiple applications with a single command.

## Supported Applications

| App       | Live Reload | Notes            |
| --------- | ----------- | ---------------- |
| Alacritty | ✓           | Instant          |
| Waybar    | ✓           | Via SIGUSR2      |
| Rofi      | ✓           | Next launch      |
| Tmux      | ✓           | Via source-file  |
| Neovim    | ✗           | Restart required |
| Wallpaper | ✓           | Via swww         |

## Installation

### Dependencies

```bash
# Required
pacman -S rofi waybar swww tmux alacritty

# Optional (for neovim live reload)
paru -S neovim-remote
```

### Setup

1. Clone/copy to `~/.Themes`

2. Make scripts executable:

```bash
chmod +x ~/.Themes/scripts/*
```

3. Create initial configs:

```bash
# Alacritty
mkdir -p ~/.config/alacritty
cat ~/.Themes/alacritty-base.toml ~/.Themes/themes/Original/alacritty-colors.toml > ~/.config/alacritty/alacritty.toml

# Rofi (ensure launcher.rasi imports colors.rasi)
cp ~/.Themes/themes/Original/rofi.rasi ~/.config/rofi/colors.rasi
```

4. Add to Hyprland config:

```bash
bind = $mainMod, T, exec, ~/.Themes/scripts/rofi-themeswitcher
bind = $mainMod, D, exec, rofi -show drun -theme launcher
```

5. Neovim setup - add to lazy.nvim plugins:

```lua
require("bryan.colorscheme")
```

## Usage

### GUI

```bash
~/.Themes/scripts/rofi-themeswitcher
```

### CLI

```bash
~/.Themes/scripts/theme-switcher apply <theme-name>
~/.Themes/scripts/theme-switcher list
~/.Themes/scripts/theme-switcher current
```

## Available Themes

| Theme           | Style                 |
| --------------- | --------------------- |
| Original        | Dark (Rosé Pine Moon) |
| CatppuccinMocha | Dark                  |
| CatppuccinLatte | Light                 |
| Nord            | Dark                  |
| NordLight       | Light                 |
| Gruvbox         | Dark                  |
| GruvboxLight    | Light                 |
| SolarizedLight  | Light                 |

## Adding New Themes

1. Create theme directory:

```bash
mkdir ~/.Themes/themes/MyTheme
```

2. Add required files:

```
~/.Themes/themes/MyTheme/
├── alacritty-colors.toml    # Alacritty colors
├── waybar.css               # Waybar stylesheet
├── rofi.rasi                # Rofi colors
├── tmux.conf                # Tmux styling
├── nvim.txt                 # Neovim colorscheme name
└── wallpaper.{png,jpg,webp} # Optional wallpaper
```

3. Use existing themes as templates.

## File Structure

```
~/.Themes/
├── scripts/
│   ├── theme-switcher       # Main switching script
│   └── rofi-themeswitcher   # Rofi GUI wrapper
├── alacritty-base.toml      # Alacritty base config (font, padding, etc.)
├── .current-theme           # Tracks active theme
└── themes/
    └── <ThemeName>/
        ├── alacritty-colors.toml
        ├── waybar.css
        ├── rofi.rasi
        ├── tmux.conf
        ├── nvim.txt
        └── wallpaper.*
```

## Troubleshooting

**Rofi not changing theme:**
Ensure your rofi command uses `-theme launcher` (not a hardcoded theme).

**Alacritty not reloading:**
Check `live_config_reload = true` in alacritty-base.toml.

**Neovim theme not loading:**
Run `:Lazy sync` to install colorscheme plugins, then restart.
