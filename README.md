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
- Color setting scripts from https://github.com/mbadolato/iTerm2-Color-Schemes.git do not work.
- But if I split the palette setting statement into parts, it works.
    ```sh
    #!/bin/sh
    # Original Red Sands
    printf "\033]4;0;#000000;1;#ff3f00;2;#00bb00;3;#e7b000;4;#0072ff;5;#bb00bb;6;#00bbbb;7;#bbbbbb;8;#555555;9;#bb0000;10;#00bb00;11;#e7b000;12;#0072ae;13;#ff55ff;14;#55ffff;15;#ffffff\007"
    printf "\033]10;#d7c9a7;#7a251e;#ffffff\007"
    printf "\033]17;#a4a390\007"
    printf "\033]19;#000000\007"
    printf "\033]5;0;#dfbd22\007"
    ```
    ```sh
    #!/bin/sh
    # Modified Red Sands
    printf "\033]4;0;#000000\007"
    printf "\033]4;1;#ff3f00\007"
    printf "\033]4;2;#00bb00\007"
    printf "\033]4;3;#e7b000\007"
    printf "\033]4;4;#0072ff\007"
    printf "\033]4;5;#bb00bb\007"
    printf "\033]4;6;#00bbbb\007"
    printf "\033]4;7;#bbbbbb\007"
    printf "\033]4;8;#555555\007"
    printf "\033]4;9;#bb0000\007"
    printf "\033]4;10;#00bb00\007"
    printf "\033]4;11;#e7b000\007"
    printf "\033]4;12;#0072ae\007"
    printf "\033]4;13;#ff55ff\007"
    printf "\033]4;14;#55ffff\007"
    printf "\033]4;15;#ffffff\007"
    printf "\033]10;#d7c9a7\007"
    printf "\033]11;#7a251e\007"
    ```
