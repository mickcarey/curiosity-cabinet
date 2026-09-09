# Terminal Multiplexers: Tiny Terminal Hotels For Long-Running Thoughts 🧰

## The Short Version

A terminal multiplexer lets one terminal contain many terminal sessions. More importantly, it lets those sessions keep living after your laptop sleeps, your train tunnel eats the network, or your iPhone decides the mobile signal is now interpretive dance.

The classics:

- **GNU Screen**: the old mountain hut. Not glamorous, still useful when the weather turns.
- **tmux**: the standard workshop bench. Scriptable, sturdy, everywhere sane.
- **Zellij**: the modern studio. Friendly UI, layouts, plugins, collaboration, batteries included.
- **Superlogical**: Mitchell Hashimoto's new multiplexer-shaped mystery box, announced as the first product from his post-Ghostty company.

## Why Multiplexers Exist 🧠

Before graphical desktops made windows feel obvious, a terminal was often a physical thing: keyboard, monitor, serial cable, one session. You had a shell prompt, maybe 80 columns by 24 rows, and that was your little cockpit.

Then Unix people did what Unix people do: they asked, "What if one thing pretended to be many things?"

A multiplexer sits between your real terminal and your programs. Your shell, editor, logs, tests, REPL, and database console talk to the multiplexer as if it were a terminal. The multiplexer then paints the combined result back onto your actual screen.

It is a stage manager:

- It gives each program a fake terminal to perform in.
- It remembers the set.
- It lets you switch scenes.
- It keeps the actors performing even if the audience briefly leaves the theatre.

That last bit is the magic. If you start a deployment, migration, render, build, or long test run over SSH and your connection drops, a plain shell may take the process with it. Inside tmux, Screen, or Zellij, the remote session keeps breathing.

## The Ancient Ancestors 🦴

### Time-Sharing: One Big Computer, Many Humans

The idea of one machine serving many interactive users goes back to time-sharing systems. The whole culture of terminals came from sharing expensive central computers through cheaper remote terminals.

The original problem was not "my MacBook has too many tabs." It was "this computer costs as much as a building, please let more than one person poke it."

### Hardware Terminals: One Window To Rule Them All

By the 1970s and 1980s, terminals like the DEC VT series helped define the language of terminal control: cursor movement, clearing the screen, alternate screens, character sets, line drawing. Modern terminal apps still speak descendants of these old escape-sequence dialects.

Multiplexers inherited a tricky job: they are not just splitting pixels. They are pretending to be a terminal emulator while also translating back to the real terminal emulator. It is terminals all the way down, like a stack of tiny glass panes.

## GNU Screen: The Old Spellbook 📜

GNU Screen appeared in 1987 and did the core trick beautifully: multiple virtual terminals inside one physical terminal, detachable sessions, scrollback, copying between windows, and processes that keep running even when the visible window changes or the whole session detaches.

Screen mattered because it solved the survival problem. A modem hangup or flaky SSH session no longer had to vaporize the workbench.

Its vibe:

- Prefix key: usually `Ctrl+a`
- Windows, not modern panes-first workflow
- Very portable
- Often present on dusty servers where installing new packages is frowned upon by change control

Screen is not the multiplexer most people would design today. It is the one you are grateful for when a locked-down machine says, "No tmux for you."

## tmux: The Unix Workbench 🪚

tmux arrived in 2007, created by **Nicholas Marriott**, and became the practical default for a huge chunk of terminal-heavy engineering work. Its own docs describe the central promise: run multiple terminal programs inside one terminal, detach them, and reattach later from another terminal.

tmux cleaned up many of Screen's rough edges:

- A clear client-server model
- Panes and windows as first-class daily tools
- Scriptable commands
- Good session naming
- A serious plugin ecosystem
- Easy SSH survival

tmux feels like a workshop where every drawer can be relabelled. This is wonderful until you inherit someone's `.tmux.conf` and discover it contains twelve years of keyboard archaeology.

### Why tmux Became The Default

tmux hit the sweet spot:

- It is small enough to trust.
- It is powerful enough to live in all day.
- It is mature enough that the exciting part is usually your work, not the multiplexer.
- It has enough community gravity that most terminal tools are tested inside it.

For remote work, tmux is less a preference and more a seatbelt. You may never need it. Then one day your connection drops during a database migration and suddenly you develop religion.

## Zellij: The Friendly Mosaic 🧩

Zellij is a newer Rust terminal workspace led by **Aram Drevekenin**. Its name comes from Moroccan mosaic tilework, which is a pretty good metaphor: many small panes arranged into a useful pattern.

Zellij keeps the multiplexer idea but changes the onboarding:

- A visible status bar explains modes and keys.
- Layouts are a first-class feature.
- Floating and stacked panes are built in.
- Plugins use WebAssembly.
- Session management and resurrection are friendlier.
- It has collaboration features and a web client story.

The important philosophical difference: tmux often assumes you will become a cave cartographer of keybindings. Zellij tries to put labels on the doors.

That makes it especially nice for:

- Project dashboards
- Repeatable layouts
- Pairing and shared sessions
- People who want power without a week of dotfile archaeology

## Superlogical: Mitchell Hashimoto Enters The Chat 👻

Mitchell Hashimoto, co-founder of HashiCorp and creator of Ghostty, announced **Superlogical** in 2026. The first product is planned to be a terminal multiplexer, built on **libghostty**, the reusable terminal engine extracted from Ghostty.

This is interesting because Mitchell has spent years obsessing over terminals from the emulator side. Most multiplexers begin from the Unix-session-management side. Superlogical appears to be asking: what happens if you design the multiplexer with modern terminal rendering, UX, and application-building ambitions from the start?

Important uncertainty: as of the announcement, the public details are intentionally thin. The shape is clear enough to track; the final product is not yet something to write workflows around.

Rabbit hole worth watching: Ghostty made "the terminal emulator" feel newly fashionable. Superlogical may try to make "the multiplexer" feel like an application platform rather than a pane splitter with a trench coat.

## The Big Design Tension ⚖️

Every multiplexer fights the same three-way tension:

- **Persistence**: keep sessions alive when the outer connection dies.
- **Composition**: arrange many programs without needing many terminal windows.
- **Fidelity**: pass through modern terminal features without mangling colours, images, keyboard protocols, or clipboard behaviour.

That third one is sneakier than it looks. A multiplexer is an extra terminal emulator in the middle. New terminal features have to survive the journey through it. This is why graphics protocols, true colour, special keyboard handling, and clipboard integration can behave differently inside and outside tmux.

The multiplexer is both umbrella and ceiling.

## When To Use Which 🧭

Use **tmux** when:

- You SSH into servers constantly.
- You want the standard answer with huge community knowledge.
- You like scriptable control.
- You want `Ctrl+b` muscle memory everywhere.

Use **Zellij** when:

- You want discoverable controls.
- You like saved layouts.
- You want nicer default session UX.
- You are building a polished local or remote workspace.

Use **Screen** when:

- It is the only thing installed.
- You need detach/reattach and nothing fancy.
- The server looks old enough to have opinions about dot-matrix printers.

Watch **Superlogical** when:

- You like Ghostty.
- You care about terminal UX as a product surface.
- You want to see whether the multiplexer becomes the next serious frontier for agentic/dev workflows.

## A Remote Dev Pattern That Actually Holds 📱

For iPhone or iPad SSH back to a powerful computer:

1. Connect with SSH or Mosh from a mobile terminal.
2. Attach to tmux or Zellij on the powerful machine.
3. Keep editor, tests, logs, and agents running there.
4. Let the phone be a tiny steering wheel, not the engine.

The phone should not compile Unreal, render Blender scenes, or index a monorepo. The phone should send keystrokes to the machine that can.

## Cheatsheet: tmux With `Ctrl+b` 🔥

In tmux, press `Ctrl+b`, release, then press the command key.

| Task | Command |
|---|---|
| Start tmux | `tmux` |
| Start named session | `tmux new -s work` |
| List sessions | `tmux ls` |
| Attach last session | `tmux attach` |
| Attach named session | `tmux attach -t work` |
| Detach | `Ctrl+b d` |
| New window | `Ctrl+b c` |
| Next window | `Ctrl+b n` |
| Previous window | `Ctrl+b p` |
| Pick window | `Ctrl+b w` |
| Rename window | `Ctrl+b ,` |
| Split horizontally | `Ctrl+b "` |
| Split vertically | `Ctrl+b %` |
| Move between panes | `Ctrl+b` then arrow key |
| Resize pane | `Ctrl+b` then hold `Ctrl` + arrow key |
| Zoom pane | `Ctrl+b z` |
| Kill pane | `Ctrl+b x` |
| Enter copy mode | `Ctrl+b [` |
| Paste buffer | `Ctrl+b ]` |
| Show clock | `Ctrl+b t` |
| Command prompt | `Ctrl+b :` |

Tiny daily loop:

```bash
tmux new -s project
# do work, split panes, run tests
# detach before leaving
# later...
tmux attach -t project
```

## Cheatsheet: Zellij Remapped To Unlock With `Ctrl+b` 🧩

Zellij's default key story is modal. If you remapped the unlock key to `Ctrl+b`, treat it as the "wake up the command layer" key.

Exact keys can vary by config, but the useful mental model is:

| Task | Typical Zellij Flow |
|---|---|
| Start Zellij | `zellij` |
| Start named session | `zellij --session work` |
| List sessions | `zellij list-sessions` |
| Attach session | `zellij attach work` |
| Detach | `Ctrl+b`, then detach command from session mode/status help |
| New pane | `Ctrl+b`, then pane/new-pane action |
| New tab | `Ctrl+b`, then tab/new-tab action |
| Move focus | `Ctrl+b`, then arrows or configured movement keys |
| Resize | `Ctrl+b`, then resize mode/actions |
| Rename tab | `Ctrl+b`, then tab rename action |
| Open session manager | `Ctrl+b`, then session manager action |
| Lock back down | mode exits automatically or use your lock binding |

The best Zellij cheat sheet is the one on screen: after unlocking, the status bar tells you what mode you are in and what keys are alive. It is the multiplexer equivalent of labelling the drawers.

## Handy Config Ideas 🛠️

### tmux: Minimal Remote-First Defaults

```tmux
set -g mouse on
set -g history-limit 50000
setw -g mode-keys vi
set -g base-index 1
setw -g pane-base-index 1
```

Why these:

- Mouse support helps from iPad/iPhone terminals when touching, selecting, or resizing.
- More history means log spelunking without immediate regret.
- Vi copy mode fits terminal-brain if you already use Vim.
- Starting at 1 makes windows match human counting.

### Zellij: Layouts Are The Good Bit

A Zellij layout can say: open editor here, logs there, shell below, task runner on the side. This turns "rebuild my workspace" from morning ritual into one command.

That is the real upgrade over "just split panes": your workspace becomes a recipe.

## Why This Matters 🌍

Multiplexers are boring in exactly the way good infrastructure is boring. They do not write your code. They preserve the shape of your attention.

The deeper story is that terminals keep surviving every generation of UI because they are compact, scriptable, remote-friendly, and weirdly humane once your fingers learn the dance. A multiplexer turns that terminal from a single hallway into a little workshop with rooms.

For remote-anywhere work, especially from an iPhone SSH session, that workshop should live on the powerful computer. Your phone is just the periscope.

## Sources And Trails

- [GNU Screen project page](https://www.gnu.org/software/screen/)
- [tmux README](https://github.com/tmux/tmux/blob/master/README)
- [tmux Getting Started wiki](https://github.com/tmux/tmux/wiki/Getting-Started)
- [Zellij website](https://zellij.dev/)
- [Zellij FAQ](https://zellij.dev/faq/)
- [Zellij beta announcement](https://zellij.dev/news/beta/)
- [Mitchell Hashimoto: Superlogical](https://mitchellh.com/writing/superlogical)
- [Mitchell Hashimoto: Ghostty notes](https://mitchellh.com/ghostty)
- [Terminfo.dev: Terminal Multiplexers](https://terminfo.dev/multiplexers)
- [LinuxCommand.org: Terminal Multiplexers](https://linuxcommand.org/lc3_adv_termmux.php)
