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
As a prerequisite install `phook`. Either from [drinchev's repo](https://github.com/drinchev/phook), or from [mine](https://github.com/kontza/phook).
The actual setup operation of changing the terminal background requires a chain of three operations.

### The First Link In The Chain: SSH Config
Use SSH's `SetEnv` to store metadata about the theme to use, add `LocalCommand` to run the next link's script, and finally permit local commands with `PermitLocalCommand`. As an example:
```
Host somehost
    PermitLocalCommand yes
    LocalCommand phook-prep %n
    # Theme names with whitespace in them require quoting.
    SetEnv TERMINAL_THEME="Red Sands"
```

### The Middle Link: `phook-prep`
Add the script below as `phook-prep` somewhere in your `PATH`. The script is called by SSH as `LocalCommand`. It first gets the parent of the parent process; the first parent is your shell running `phook-prep`, and _its_ parent is the SSH process. That process is then hooked on to with `phook`. Also, the script checks if SSH is being run in `BatchMode`. `BatchMode` is enabled, for example, when you're using Fish shell and tab completing a command like like: `scp local.file somehost:/var/lib/tar<TAB>`.

```sh
#!/bin/sh
SSH_PID=$(ps -o ppid= $PPID)
SSH_COMMAND_LINE=$(ps -eo args= $SSH_PID)
case $SSH_COMMAND_LINE in
*"rsync"*)
  # Bypass setting colors on rsync over SSH
  ;;
*"ssh -W"*)
  # Bypass setting colors on ProxyJump stage with SSH
  ;;
*"BatchMode yes"*)
  # BatchMode; e.g. tab completion in an 'scp' command completion on remote server
  ;;
*)
  phook -p $SSH_PID -e "ssc $1" -a 'ssc reset'
  ;;
esac
```

### The Final Link: `ssc`
Add the script below as `ssc` somewhere in your `PATH`. Adapt the `host_to_theme` dict to your own environment. The dict contains regular expressions for host name matching as keys, and Ghostty theme names as values. The script iterates over dict keys, and when it matches the host you're connecting to, it will print ANSI OSC codes to set the colors to use according to the ones in the specified Ghostty theme. As you can see from the `open` statement, this script assumes the OS being macOS, and that you've installed Ghostty into `/Applications`.

```python
#!/usr/bin/env python3
import configparser
import os
import re
import subprocess
import sys

patterns = [
    {
        "pat": re.compile("^palette *="),
        "fmt": "4;{0}",
    },
    {"pat": re.compile("^foreground *="), "fmt": "10;{0}"},
    {"pat": re.compile("^background *="), "fmt": "11;{0}"},
    {"pat": re.compile("^cursor-color *="), "fmt": "12;{0}"},
]
show_debug = False


def get_theme_name(host_name: str) -> str:
    res = subprocess.run(["ssh", "-G", host_name], capture_output=True)
    for line in res.stdout.decode().split("\n"):
        if "setenv terminal_theme" in line.lower():
            theme_name = line.split("=")[-1].replace('"', "").replace("'", "")
            if show_debug:
                print(f"Found theme: {theme_name}")
            return theme_name
    return None


def set_scheme(host_name: str):
    if host_name == "reset":
        reset_scheme()
        return
    theme_name = get_theme_name(host_name)
    if theme_name is not None:
        res_dir = os.environ["GHOSTTY_RESOURCES_DIR"]
        if res_dir is None:
            match sys.platform:
                case "darwin":
                    res_dir = "/Applications/Ghostty.app/Contents/Resources/ghostty"
                case "linux":
                    res_dir = "/usr/share/ghostty"
        with open(
            f"{res_dir}/themes/{theme_name}",
        ) as config_file:
            for line in config_file.readlines():
                line = line.strip()
                if line is None or line.startswith("#"):
                    continue
                for p in patterns:
                    mo = p["pat"].match(line)
                    if mo is not None:
                        format = p["fmt"].replace("\\", "").format("0")
                        if mo.group().startswith("palette"):
                            color = line[mo.span()[1] :].replace("=", ";").strip()
                        else:
                            color = f'#{line.split("=")[1].strip()}'
                        if show_debug:
                            print(
                                "'{0}': Would use '\\033]{1}\\007'".format(
                                    line, p["fmt"].format(color)
                                )
                            )
                        print("\033]{0}\007".format(p["fmt"].format(color)), end="")


def reset_scheme():
    # Reset a palette color (OSC 104) or the foreground (OSC 110), background (OSC 111), or cursor (OSC 112) color.
    for c in range(0, 15):
        print(f"\033]104;{c}\007", end="")
    print("\033]110\007", end="")
    print("\033]111\007", end="")
    print("\033]112\007", end="")


if __name__ == "__main__":
    try:
        set_scheme(sys.argv[1])
    except IndexError:
        print("Gimme a hostname to work on!", file=sys.stderr)

# vim: set ft=python
```
