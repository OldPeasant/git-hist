# git-hist

An interactive, two-pane Git history browser for the terminal.

The left pane shows the commit graph (`git log --graph --oneline`). The right pane
shows `git show` for the selected commit: message, author, date and the full diff.
It updates as you move through the history.

```
 git la                              │ git show a251cac
* 4b06179 commit 13                  │commit a251caccf7c2bc9d5ae41fafa34f947745a3a566
* c36ef76 commit 4                   │Merge: e30b3b8 d9fae44
*   a251cac Merge branch 'feature'   │Author: …
|\                                   │
| * d9fae44 (feature) feature work   │    Merge branch 'feature'
* | e30b3b8 main work                │
|/                                   │
* 4a4bda7 commit 3                   │
```

It's a single Python 3 script that uses only the standard library (`curses`).

## Usage

```
git hist                  # history of all refs      (based on `git la`)
git hist <args...>        # history of given revs    (based on `git ll <args...>`)
```

Examples:

```
git hist
git hist main
git hist -50 main feature
git hist v1.0..HEAD -- src/
```

`la` and `ll` are Git aliases. If they're defined in your Git config, they are used,
including any changes you made to them. Otherwise these defaults apply:

```
git config --global alias.la "log --decorate --oneline --graph --all"
git config --global alias.ll "log --decorate --oneline --graph"
```

## Keys

| Key                 | Action                                                            |
|---------------------|-------------------------------------------------------------------|
| `←` / `→`           | Focus the history / details pane (`Tab` toggles)                  |
| `↑` / `↓` (`k`/`j`) | History: select previous / next commit. Details: scroll           |
| `PgUp` / `PgDn`     | Page up / down (`Space` also pages down)                          |
| `Home` / `End`      | First / last                                                      |
| `Enter`             | Show the details full-screen (`Enter`, `Esc`, `q` or `←` go back) |
| `/`                 | Search the focused pane (see below)                               |
| `n` / `N`           | Next / previous match                                             |
| `y` / `Y`           | Copy the full / short hash of the selected commit                 |
| `<` / `>`           | Shrink / grow the history pane                                    |
| `q` / `Esc`         | Quit                                                              |

Graph-only lines such as `|\` or `|/` are skipped when moving the selection.

**Search:**

- `/` searches the focused pane. Searching the history jumps to the next matching
  commit line (hash, refs or subject). Searching the details scrolls the match to
  the top.
- The pattern is a regular expression. If it isn't a valid one, it's searched as
  plain text.
- Search ignores upper/lower case unless the pattern contains a capital letter.
- Search wraps around at the end.
- Matches are highlighted in both panes.
- Pressing `Enter` on an empty prompt repeats the last search.

## Installation

When you run `git hist`, Git looks for an executable named `git-hist` on your
`PATH`. Installing means putting the script, unchanged and without a `.py`
extension, into a folder that is on your `PATH`.

### Linux

Requires Python 3, which most distributions include.

`~/.local/bin` is a good place. Most distributions, including Fedora and Ubuntu,
already add it to your `PATH` in the default `~/.bashrc` or `~/.profile`.

```
mkdir -p ~/.local/bin
cp git-hist ~/.local/bin/
chmod +x ~/.local/bin/git-hist
```

Or, to keep using a cloned copy of this repository so `git pull` updates it:

```
ln -s "$PWD/git-hist" ~/.local/bin/git-hist
```

Check it with `command -v git-hist`. If nothing is printed, add
`export PATH="$HOME/.local/bin:$PATH"` to your `~/.bashrc` and open a new
terminal.

Copying a hash uses `wl-copy` (Wayland), or `xclip` / `xsel` (X11).

### macOS

1. **Python 3:** `python3` is available once the Xcode Command Line Tools are
   installed (`xcode-select --install`). You likely have them already if you use
   the `git` that comes with macOS. Homebrew's `python3` works as well.
2. **Install the script.** zsh is the default shell, and `~/.local/bin` isn't on
   the `PATH` by default, so add it:

   ```
   mkdir -p ~/.local/bin
   cp git-hist ~/.local/bin/
   chmod +x ~/.local/bin/git-hist
   echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
   ```

   Open a new terminal, and `git hist` is available.

Copying a hash uses `pbcopy`.

### Windows

Requires [Git for Windows](https://gitforwindows.org/).

1. **Install Python 3** from [python.org](https://www.python.org/downloads/).
2. **Install curses support.** Windows Python doesn't include curses:

   ```
   py -m pip install windows-curses
   ```

3. **Put `git-hist` in a folder on your user `PATH`**, for example
   `%USERPROFILE%\bin`. Add that folder under *Settings → System → About →
   Advanced system settings → Environment Variables → user `Path`*, then open a
   new terminal. Git Bash already includes `~/bin`, but cmd and PowerShell need
   the Windows `PATH` entry.
4. **Run `git hist`** in a repository.

   Git for Windows reads the script's first line, `#!/usr/bin/env python3`, and
   looks for a `python3` command. Depending on how Python was installed, it may
   only exist as `python` or `py`, or `python3` may be the Microsoft Store
   placeholder. If `git hist` doesn't start, define an alias that names the
   interpreter directly:

   ```
   git config --global alias.hist "!py C:/Users/<you>/bin/git-hist"
   ```

   Git runs aliases from the repository's top folder. With the alias, a path
   argument like `git hist -- src/foo.c` therefore resolves from the top folder,
   not from the subfolder you're in.

Use [Windows Terminal](https://aka.ms/terminal) for proper colors and keys.
Copying a hash uses `clip.exe`, which comes with Windows.

## Clipboard

`y` / `Y` try, in this order:

1. `wl-copy` (if `WAYLAND_DISPLAY` is set)
2. `xclip` / `xsel` (if `DISPLAY` is set)
3. `pbcopy` (macOS)
4. `clip.exe` (Windows, and Linux under WSL)

If none of these works, the script falls back to OSC 52, an escape sequence that
asks the terminal to set the clipboard. This also works over SSH in most modern
terminals. Inside tmux, enable it with `set -g set-clipboard on`.

## Help

```
git-hist --help
```

`git hist --help` doesn't work: Git handles `--help` itself and looks for a man
page.

## License

MIT, see [LICENSE](LICENSE).
