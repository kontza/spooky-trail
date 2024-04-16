# Introduction
A random collection of my configuration changes to Ghostty on macOS.


## Theme
Ghostty supports Catppuccin Mocha OOTB. Nice. Just set your config:

```
theme = catppuccin-mocha
```

Also, make sure you don't have any `foreground=...`, and `background=...` settings active as they will override the theme configuration.

## Fonts
This was a bit tricky. Lately I've been using Hack font in Wezterm. The way I had installed the font made it so that Ghostty didn't find the font.

This was easy to fix: use Homebrew casks.

1. Install font:
    ```sh
    ❯ brew tap homebrew/cask-fonts
    ❯ brew install --cask font-hack
    ```
2. Set _font-family_ in configuration:
    ```
    font-family = "Hack"
    ```

## Change Pane Background Per SSH Host
Thoughts:
- As Ghostty does not have scripting supported, yet, my Wezterm approach (events etc) will not work.
- Probably https://github.com/drinchev/phook is the way to go. This is not new, I used this before starting to use Wezterm's events.

