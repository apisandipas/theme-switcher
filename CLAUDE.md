# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Unified theme switcher for a Hyprland desktop environment. Manages consistent theming across Alacritty, Waybar, Rofi, Swaync, Tmux, Neovim, and wallpaper (via swww).

## Key Commands

```bash
# Apply a theme
~/.Themes/scripts/theme-switcher apply <ThemeName>

# List available themes
~/.Themes/scripts/theme-switcher list

# Show current theme
~/.Themes/scripts/theme-switcher current

# Restore wallpaper at login
~/.Themes/scripts/theme-switcher restore-wallpaper

# Launch GUI picker
~/.Themes/scripts/rofi-themeswitcher
```

## Architecture

### Theme Application Flow

`theme-switcher apply <name>` does the following in order:

1. **Alacritty** — Concatenates `alacritty-base.toml` + `themes/<name>/alacritty-colors.toml` → `~/.config/alacritty/alacritty.toml` (live reloads automatically)
2. **Waybar** — Copies `waybar.css` → `~/.config/waybar/style.css`, sends `SIGUSR2` to reload
3. **Rofi** — Copies `rofi.rasi` → `~/.config/rofi/colors.rasi` (applies on next launch)
4. **Swaync** — Copies `swaync.css` → `~/.config/swaync/style.css`, reloads via `swaync-client -rs`
5. **Wallpaper** — Uses `swww img` with wipe transition (auto-starts daemon if needed)
6. **Tmux** — Copies `tmux.conf` → `~/.tmux-theme.conf`, hot-reloads via `tmux source-file`
7. **Neovim** — Writes colorscheme name to `~/.config/nvim/.colorscheme`, sends commands to running instances via `nvr` (neovim-remote). Has special-case handling for rose-pine variants, solarized-light, gruvbox-light, nord-light, everforest-light, and catppuccin flavours.
8. **State** — Writes theme name to `.current-theme`

### Alacritty Template Pattern

The Alacritty config is generated, not edited directly. `alacritty-base.toml` contains shared settings (font, window, keybindings). Theme color files are appended to it. Any changes to base Alacritty settings must go in `alacritty-base.toml`, not the generated output at `~/.config/alacritty/alacritty.toml`.

**Important**: TOML does not support `\x` hex escapes. Use `\u001b` instead of `\x1b` for the ESC character.

### Theme Directory Structure

Each theme in `themes/<ThemeName>/` contains:

| File | Target App | Format |
|---|---|---|
| `alacritty-colors.toml` | Alacritty | TOML color sections |
| `waybar.css` | Waybar | Full CSS stylesheet |
| `rofi.rasi` | Rofi | 6 color variables (al, bg, bga, fga, fg, ac) |
| `swaync.css` | Swaync | GTK CSS with color variables |
| `tmux.conf` | Tmux | Status bar + pane styling |
| `nvim.txt` | Neovim | Single line: colorscheme name |
| `wallpaper.*` | swww | Optional (png/jpg/jpeg/webp) |

### Adding a New Theme

Create a new directory under `themes/` with all the files above. The theme-switcher discovers themes by scanning `themes/*/` directories — no registration needed.

## Current Themes

Original (Rosé Pine Moon), CatppuccinMocha, CatppuccinLatte, Nord, NordLight, Gruvbox, GruvboxLight, SolarizedLight, TokyoNight, TokyoNightDay, Dracula, Everforest, EverforestLight
