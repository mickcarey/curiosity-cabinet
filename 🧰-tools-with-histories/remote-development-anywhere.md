# Remote Development Anywhere: iPhone As Periscope, Big Computer As Engine 📱

## The Big Idea

Remote development is not "make the tiny device do everything." That way lies hot glass, sad battery, and builds that finish sometime next geological period.

The better pattern:

- **Powerful computer**: code, build, render, index, run agents, host tmux/Zellij.
- **iPhone/iPad/laptop**: connect, steer, inspect, approve, nudge.
- **Network glue**: SSH, Mosh, Tailscale, or a remote IDE tunnel.
- **Session glue**: tmux or Zellij so the work survives disconnects.

Your phone becomes a ship's wheel. The actual engine room stays somewhere with fans.

## The Lineage: From Telnet To Pocket Workstations 🧬

### Telnet And rlogin: Trusting The Network Like A Foolhardy Wizard

Early remote terminals were wonderfully useful and wildly trusting. Protocols like Telnet and rlogin made it possible to use another machine interactively, but the security assumptions belonged to a smaller, friendlier internet.

### SSH: The Lock On The Door

SSH arrived in the 1990s, created originally by **Tatu Ylonen**, then became a staple through OpenSSH after OpenBSD developers rescued a free-enough codebase in 1999 and pushed it forward with their usual "security is not a garnish" intensity.

SSH changed remote work because it wrapped remote shells, file transfer, tunnels, and command execution in encryption. It became the default doorway into servers.

### Mosh: SSH For Trains, Cafes, And Phone Networks

Mosh, written by **Keith Winstein** with Anders Kaseorg, Quentin Smith, Richard Tibbetts, Keegan McAllister, and John Hood, solves a different pain: flaky interactive connections.

Mosh logs in over SSH, then uses a UDP-based protocol that keeps the session usable while networks roam. Change Wi-Fi to cellular, close the lid, move through patchy coverage, and Mosh is much less dramatic than plain SSH.

It also predicts local typing, so a high-latency link feels less like shouting commands across a canyon.

### Tailscale: Make Your Machines Feel Adjacent

Tailscale, started in 2019 by **Avery Pennarun, David Carney, and David Crawshaw**, builds a mesh network on WireGuard. Instead of exposing SSH to the whole internet or wrestling port forwards, your devices join a private tailnet.

For remote dev, the magic trick is boring names:

```bash
ssh workstation
tmux attach -t project
```

No public IP dance. No hotel router negotiation ceremony. Just your phone finding your big machine through the mesh.

## The iPhone SSH Stack 🧳

### Blink Shell

Blink Shell is an iOS/iPadOS terminal focused on SSH and Mosh. It supports mobile-friendly terminal work, keys, SFTP, external keyboards, and Mosh sessions that survive the usual phone-network nonsense.

Best fit:

- You want a terminal-first iPhone/iPad workflow.
- You use Mosh.
- You care about keyboard feel.
- You want your mobile device to feel like a terminal, not a web form with delusions.

### Termius

Termius is the polished cross-platform SSH client path: saved hosts, snippets, SFTP, sync, vault/team features, and a more GUI-managed style.

Best fit:

- You manage many hosts.
- You want synced connection profiles.
- You like snippets and host groups.
- You want paid convenience instead of hand-rolled config.

### Working Copy

Working Copy is the serious Git client for iOS. It can clone, edit, commit, push, search repositories, use Shortcuts automation, and integrate with iOS file workflows.

Best fit:

- You need Git on-device.
- You want to review or patch Markdown/code locally.
- You want commits from the couch without opening a full remote IDE.

## Handy Tool Cabinet: Paid, Open, And Worth Knowing 🧰

| Tool | Kind | Creator / Origin | Good Use |
|---|---|---|---|
| `tmux` | Open source | Nicholas Marriott, 2007 | Long-lived SSH workspaces, logs, builds, agents |
| Zellij | Open source | Aram Drevekenin and community | Discoverable terminal workspaces, layouts, collaboration |
| GNU Screen | Open source | GNU project lineage, 1987 | Legacy boxes where it is the only multiplexer available |
| Mosh | Open source | Keith Winstein and collaborators | Mobile SSH over flaky Wi-Fi/cellular |
| OpenSSH | Open source | OpenBSD project, derived from Tatu Ylonen's SSH | The default secure remote doorway |
| Tailscale | Commercial + free tiers, open client pieces | Avery Pennarun, David Carney, David Crawshaw | Private mesh access to your workstation without public SSH |
| Blink Shell | Paid app, open-source codebase | Blink Shell Project | iPhone/iPad terminal with SSH, Mosh, keys, SFTP |
| Termius | Commercial + free tier | Termius | Saved hosts, snippets, synced SSH profiles, team vault workflows |
| Working Copy | Commercial iOS app | Working Copy | Git review, edits, commits, Shortcuts automation on iOS |
| VS Code Remote SSH | Free extension ecosystem | Microsoft | Rich remote IDE against your own machine |
| GitHub Codespaces | Commercial + quotas | GitHub | Disposable/cloud dev environments when self-hosting is overkill |

Example projects:

- **Website hotfix from phone**: Blink -> Mosh -> tmux -> edit Markdown/code -> run check -> push.
- **Game-dev triage**: iPhone SSH -> workstation tmux -> Unreal log pane + AI agent pane + Jira context.
- **Design handoff**: laptop/iPad browser -> remote IDE -> Figma MCP -> implementation branch.
- **3D automation**: phone as approver -> workstation Blender/Unreal session -> MCP-driven scene edits and render previews.

## The Practical Anywhere Setup 🛠️

### 1. Prepare The Big Computer

Install:

- `openssh-server`
- `tmux` or `zellij`
- `mosh` if you want mobile-friendly roaming
- Tailscale if you want private-device networking
- Your real tools: editor, compilers, Docker, Blender, Unreal, AI agent CLI, database clients

Keep the heavy stuff here. The whole point is that your phone should not be pretending to be a workstation. It should be a remote control with excellent posture.

### 2. Put Work Inside A Named Session

```bash
tmux new -s game-dev
```

Inside:

- Pane 1: editor or agent
- Pane 2: tests/builds
- Pane 3: logs
- Pane 4: scratch shell

Later:

```bash
tmux attach -t game-dev
```

Zellij version:

```bash
zellij --session game-dev
zellij attach game-dev
```

### 3. Connect From iPhone

Plain SSH:

```bash
ssh user@workstation
```

Mosh:

```bash
mosh user@workstation
```

Tailscale SSH:

```bash
ssh user@workstation
# or
tailscale ssh user@workstation
```

The holy little loop:

```bash
mosh workstation
tmux attach -t project
```

That is the pocket workstation spell. Very small incantation, very large computer.

## Remote IDE Options 🖥️

### VS Code Remote SSH

VS Code Remote SSH opens a folder on a remote machine and runs the VS Code server-side pieces there. The local UI still feels like VS Code, but the repo, extensions, terminals, and debugging can live on the remote host.

Best fit:

- You want rich code navigation.
- You use extensions heavily.
- You are on a laptop or iPad browser workflow more than raw terminal.

### GitHub Codespaces / Gitpod / Cloud Dev Boxes

These move the powerful computer into the cloud. Useful when you do not want to maintain the box yourself, less ideal when you need local GPU, giant asset projects, Blender, Unreal, or private hardware.

Paid convenience buys:

- Fast onboarding
- Shareable environments
- Fewer "works on my machine" rituals

Self-hosted muscle buys:

- GPU access
- Huge local storage
- No cloud meter ticking during long compiles
- Your weird tools installed exactly once

## Blender And Unreal From The Road 🎨

For Blender and Unreal, the phone should mostly supervise:

- Kick off renders or automation.
- Check build/test status.
- Ask an AI agent to inspect assets or create editor changes via MCP.
- Review generated screenshots or logs.
- Approve commits and PRs.

Do not try to *be* the viewport on a phone unless the task is tiny. A phone is excellent for command, review, and triage. It is not a comfortable place to model bevels for three hours.

## A Good Daily Layout 🧭

Use tmux or Zellij on the big machine:

| Pane | Purpose |
|---|---|
| Editor/agent | Codex, Claude Code, Vim, Neovim, or shell-driven work |
| Build/test | `pnpm test`, `cargo test`, Unreal automation, render scripts |
| Logs | app logs, engine logs, server logs |
| Ops | Git, Jira notes, deploy commands |

Add a remote IDE only when the terminal stops being enough. The lazy version is SSH + multiplexer. The deluxe version is remote IDE + multiplexer + mesh network.

## Tiny Security Notes 🔐

- Use SSH keys, not passwords, where possible.
- Keep SSH off the public internet if Tailscale can cover it.
- Use separate low-privilege accounts for remote mobile access when practical.
- Be careful with AI agents connected to powerful tools. "Can edit Jira" and "can delete Jira" are not the same permission emotionally, even if the OAuth scope is feeling spicy.
- Treat Blender/Unreal MCP as local automation power tools. Local-only is a feature, not a nuisance.

## Why This Is Interesting

The history loops beautifully.

We started with dumb terminals connected to big machines. Then personal computers put the power under the desk. Then laptops carried it around. Now phones are good enough to be terminals again, while the real horsepower lives in a workstation, cloud box, or GPU tower somewhere else.

Remote development is not a new idea. It is the old terminal dream returning with better encryption, better networks, better screens, and a keyboard you can fold into a cafe table.

## Sources And Trails

- [OpenSSH project history](https://www.openssh.org/history.html)
- [Mosh: the mobile shell](https://mosh.org/)
- [Tailscale SSH docs](https://tailscale.com/docs/features/tailscale-ssh)
- [Tailscale: how it works](https://tailscale.com/blog/how-tailscale-works)
- [Tailscale: what is Tailscale?](https://tailscale.com/docs/concepts/what-is-tailscale)
- [VS Code Remote Development using SSH](https://code.visualstudio.com/docs/remote/ssh)
- [Blink Shell docs](https://docs.blink.sh/)
- [Blink Shell website](https://blink.sh/)
- [Termius website](https://termius.com/)
- [Working Copy website](https://workingcopy.app/)
- [Curiosity Cabinet: Terminal Multiplexers](terminal-multiplexers.md)
