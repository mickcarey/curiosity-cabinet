# Command-Line Accelerators: Tiny Tools That Make The Shell Feel Telepathic ⚡

## The Short Version

Some tools do not replace your workflow. They remove the gravel from it.

This is the little kit:

- `rg`: find text fast.
- `fd`: find files without summoning ancient `find` syntax.
- `fzf`: choose things fuzzily.
- `zoxide`: jump to directories by memory, not full paths.
- `jq`: interrogate JSON without crying into braces.
- `bat`: `cat`, but with colour, line numbers, and manners.
- `just`: project commands without Makefile cosplay.
- `mise`: tool versions, env vars, and tasks in one tidy station.
- `direnv`: load project environment when you enter; unload when you leave.

None of these is individually revolutionary. Together they make a terminal feel like it has been quietly paying attention.

## `rg`: ripgrep, Search With Its Boots On 🔎

`ripgrep`, by **Andrew Gallant** (`BurntSushi`), recursively searches directories for regex patterns while respecting `.gitignore` by default.

That default matters. Old recursive search often wastes time rummaging through `node_modules`, build outputs, vendor blobs, and binary files. `rg` usually starts from the assumption you actually mean "search the project," not "search every fossil layer under this directory."

Useful commands:

```bash
rg "ModelContextProtocol"
rg -n "TODO|FIXME"
rg -tmd "tmux"
rg -uuu "secret-looking-thing"
rg --files | rg "mcp"
```

Use it when:

- You need to understand a codebase or notes repo quickly.
- You want every caller of a function before changing it.
- You are searching over SSH and want the answer before your coffee cools.

## `fd`: Find Files Without The Incantation

`fd`, by **David Peter** (`sharkdp`) and maintainers, is a friendlier file finder. It is fast, parallel, colourful, regex-friendly, and ignores hidden/gitignored files by default.

Useful commands:

```bash
fd tmux
fd -e md mcp
fd -t f "notes"
fd -H ".env"
fd -x wc -l
```

Why it is nice:

- `fd thing` is easier than `find . -iname '*thing*'`.
- Defaults match common developer intent.
- It composes beautifully with `fzf`.

## `fzf`: The Fuzzy Finder That Turns Lists Into Interfaces 🌸

`fzf`, by **Junegunn Choi**, is a command-line fuzzy finder. It reads a list, lets you interactively filter it, then outputs your selection.

That sounds tiny. It is not tiny. It is a pocket UI toolkit for the terminal.

Useful commands:

```bash
vim "$(fzf)"
git checkout "$(git branch --format='%(refname:short)' | fzf)"
rg --files | fzf
history | fzf
```

Nice shell bindings:

- `Ctrl+t`: pick files into the command line.
- `Ctrl+r`: fuzzy-search shell history.
- `Alt+c`: fuzzy-change directory, depending on shell integration.

Pair it with:

```bash
rg --files | fzf --preview 'sed -n "1,120p" {}'
```

That one turns your terminal into a tiny searchable file browser. No ceremony. Just a list with a flashlight.

## `zoxide`: `cd` With A Memory 🧭

`zoxide`, by **ajeetdsouza**, is a smarter `cd`. It learns where you go, then lets you jump back by partial names.

Useful commands:

```bash
z cabinet
z unreal project
zi
```

The real win is remote work. On a phone SSH session, typing long paths is thumb punishment. `zoxide` turns:

```bash
cd ~/dev/personal/curiosity-cabinet/🧰-tools-with-histories
```

into:

```bash
z curiosity tools
```

That is not laziness. That is ergonomic mercy.

## `jq`: JSON Archaeology Tools

`jq`, by **Stephen Dolan** originally, slices and transforms JSON from the command line.

Useful commands:

```bash
jq .
jq '.items[].name'
jq -r '.url'
curl -s https://api.github.com/repos/ghostty-org/ghostty | jq -r '.stargazers_count'
```

Use it for:

- API responses
- MCP tool outputs
- package metadata
- logs
- anything that looks like braces had a meeting

## `bat`: `cat` With Better Lighting

`bat`, also by **sharkdp**, prints files with syntax highlighting, line numbers, and Git change markers.

Useful commands:

```bash
bat README.md
bat -n file.md
bat --plain file.md
```

It pairs beautifully with `fzf` previews:

```bash
rg --files | fzf --preview 'bat --style=numbers --color=always {}'
```

## `just`: Save The Project Spells

`just`, by **Casey Rodarmor**, is a command runner. It stores project recipes in a `justfile`.

It is not a build system. That is the charm. It is a tidy menu of commands people actually run.

Example:

```make
test:
    pnpm test

dev:
    pnpm dev

notes:
    rg --files | rg '\.md$'
```

Then:

```bash
just test
just dev
just notes
```

Use it when README command lists keep going stale. A `justfile` is documentation that can be executed, which is the best kind of documentation when commands are involved.

## `mise`: Everything In Its Place 🍳

`mise`, created by **Jeff Dickey**, manages dev tools, environment variables, and tasks. Think Node/Python/Terraform versions, per-project env, and common commands in one place.

Useful commands:

```bash
mise use node@24
mise install
mise ls --current
mise run test
mise exec -- node --version
```

Why it complements remote dev:

- New machine setup gets less mysterious.
- Project versions travel with the repo.
- CI and local tasks can speak the same language.
- SSHing into a workstation feels less like entering a room where someone moved all the spoons.

## `direnv`: Project Environments That Clean Up After Themselves

`direnv` loads environment variables when you enter a directory and unloads them when you leave.

Useful flow:

```bash
echo 'export PROJECT_NAME=curiosity-cabinet' > .envrc
direnv allow
cd ..
cd curiosity-cabinet
echo "$PROJECT_NAME"
```

The best part is the unload. Global shell profiles become junk drawers very quickly. `direnv` keeps project-specific secrets and switches near the project instead of smeared across your entire machine.

## The Tiny Superstack

For a fast remote shell:

```bash
rg + fd + fzf + zoxide + tmux
```

For project setup:

```bash
mise + just + direnv
```

For API/MCP spelunking:

```bash
curl + jq + bat
```

For iPhone SSH back to a workstation:

```bash
mosh workstation
tmux attach -t project
z project
rg "thing I need"
```

## Why This Is Interesting

The command line stays powerful because little tools keep making it sharper. No giant platform migration. No productivity cathedral. Just ten small utilities quietly removing friction from search, movement, setup, and repetition.

The trick is not to install every shiny binary. The trick is to notice where your hands keep doing boring work, then give that boring work a tiny tool.

## Sources And Trails

- [ripgrep GitHub](https://github.com/BurntSushi/ripgrep)
- [ripgrep guide](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md)
- [fd GitHub](https://github.com/sharkdp/fd)
- [fzf GitHub](https://github.com/junegunn/fzf)
- [zoxide GitHub](https://github.com/ajeetdsouza/zoxide)
- [jq website](https://jqlang.org/)
- [jq GitHub](https://github.com/jqlang/jq)
- [bat GitHub](https://github.com/sharkdp/bat)
- [just GitHub](https://github.com/casey/just)
- [mise website](https://mise.en.dev/)
- [mise about](https://mise.en.dev/about)
- [direnv website](https://direnv.net/)
- [Curiosity Cabinet: Remote Development Anywhere](remote-development-anywhere.md)
