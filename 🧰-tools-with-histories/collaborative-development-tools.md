# Collaborative Development Tools: Sharing The Workshop Without Mailing The Whole Workshop 🤝

## The Short Version

Collaborative dev tools solve one awkward problem: "I need you in my context, but I do not want to spend forty minutes recreating my context on your machine."

The families:

- **Terminal sharing**: tmate, Zellij web/session sharing, shared tmux over SSH.
- **Editor sharing**: Visual Studio Live Share, JetBrains Code With Me.
- **Screen-control pairing**: Tuple, Zoom/Meet/Slack when nothing else is available.
- **Cloud workspaces**: Codespaces, Gitpod, dev containers.
- **AI-assisted collaboration**: agents connected to Jira, Confluence, Figma, GitHub, Blender, Unreal via MCP.

The whole story is a fight against setup drag.

## Why These Tools Exist

Pairing used to mean sitting at the same machine. Then distributed teams became normal and the shared-machine illusion had to travel through the internet.

Plain screen sharing is good for showing. It is weaker for doing. The other person can see your editor, but code navigation, search, debugging, terminal access, and local services are trapped on your side of the glass unless the tool gives them a real handle.

Collaborative dev tooling exists to move from:

```text
"Watch my screen"
```

to:

```text
"Work with my environment"
```

That difference is enormous.

## tmate: Instant Terminal Pairing

tmate is a terminal sharing tool built around tmux ideas. You start a session, it gives you a connection string, and someone else can join your terminal.

Best fit:

- Quick remote debugging
- Teaching command-line work
- Pairing on servers
- Letting someone watch a live fix without giving them broader machine access

Mental model:

```bash
tmate
# send the read-only or writable connection string
```

The important social feature is read-only access. Sometimes someone needs to observe the spell, not touch the cauldron.

## Zellij Sharing: Multiplexer As Browser-Accessible Workspace

Zellij has a web client and remote session sharing story. The built-in web server is off by default, requires authentication, and can issue tokens, including read-only tokens in newer flows.

Best fit:

- Sharing a terminal workspace in a browser
- Teaching or demos
- Remote debugging sessions
- Browser access from machines without a terminal emulator

This complements iPhone/iPad remote work nicely: SSH is still the sturdy path, but browser access can be handy when the device or environment is awkward.

Security note: if you expose a terminal workspace, you are exposing a terminal workspace. Put it behind HTTPS, auth, and boring network hygiene. Boring is beautiful here.

## Visual Studio Live Share: Share The Editor Context

Visual Studio Live Share lets people collaborate in the same project from their own editor. It can share code editing, debugging, terminals, and local servers without making everyone clone the repo and install the whole toolchain.

Best fit:

- Pair programming
- Debugging together
- Code reviews with live navigation
- Onboarding someone into a project without making them build the universe first

Current caveat: Microsoft documentation says Live Share is in maintenance mode, with existing capabilities remaining available. That makes it useful, but not the place to expect a burst of new features.

## Tuple: Screen Sharing Built For Pairing

Tuple is a paid remote pair-programming app for macOS and Windows. Its bet is that generic meeting software is not quite right for programming: the text must be sharp, audio must be crisp, remote control must feel immediate, and switching who drives should be painless.

Best fit:

- Human-to-human pairing
- Mentoring
- Debug sessions where visual context matters
- Work involving GUI tools like Figma, Blender, or Unreal where editor-only sharing is not enough

Tuple is not trying to understand your codebase. It is trying to make shared control feel less like piloting through wet cardboard.

## Cloud Workspaces: Share The Machine, Not Just The Screen

Codespaces, Gitpod, and dev-container workflows move setup into a reproducible environment. Instead of "install these twelve things and sacrifice a weekend," the project declares enough environment for a new workspace to start.

Best fit:

- Web/backend repos
- Contributor onboarding
- Teaching
- Standardized team environments
- Quick experiments

Less ideal:

- Giant Unreal projects
- Heavy Blender/GPU pipelines
- Hardware-specific workflows
- Anything where cloud cost or asset size becomes the main character

The pragmatic split:

- Cloud workspace for light-to-medium app work.
- Personal workstation over SSH for heavyweight creative/game/dev tooling.

## AI Agents As Collaborators

AI agents add a new collaboration pattern: not just another human sharing your screen, but a tool-using assistant sharing your context.

With MCP, the useful version looks like:

```text
Agent reads Jira + Confluence
Agent inspects code
Agent pulls Figma context
Agent edits branch
Agent runs checks
Human approves writes and PR
```

For Blender and Unreal:

```text
Agent connects to local editor MCP
Agent inspects scene/editor state
Agent proposes changes
Human approves tool calls
Editor changes happen on workstation
```

The remote-anywhere trick is to run the agent on the powerful machine inside tmux/Zellij, then steer from wherever you are.

## A Useful Decision Table

| Need | Tool Shape |
|---|---|
| "Watch this terminal command" | tmate read-only, Zellij read-only sharing |
| "Help me debug in terminal" | tmate writable, shared tmux over SSH |
| "Edit with me in my IDE" | Live Share / Code With Me |
| "Drive my whole screen" | Tuple |
| "Spin up the same repo quickly" | Codespaces / Gitpod / dev container |
| "Connect AI to our work tools" | MCP servers for Jira, Confluence, Figma, GitHub |
| "Automate editor work in Blender/Unreal" | Local MCP on the workstation |

## Handy Session Patterns

### Remote Debugging With Terminal Sharing

```bash
tmate
# send read-only first
# upgrade to writable only if needed
```

### Pairing On A Workstation From The Road

```bash
mosh workstation
tmux attach -t pair
```

Then use voice chat separately. The terminal does terminal things; voice does human things. Simple tools, clean boundaries.

### AI Agent In A Persistent Session

```bash
tmux new -s agent
# start Codex/Claude/etc inside it
# keep Jira/Figma/GitHub MCP available to that agent
```

When the phone disconnects, the agent session does not evaporate. This is the same old multiplexer magic, now wearing an AI hat.

## Safety Notes

- Prefer read-only sharing first.
- Share the smallest context that solves the problem.
- End sessions when done.
- Avoid sharing terminals that already hold production credentials.
- Treat shared terminals as live keyboards, not screenshots.
- For AI tools, separate read capabilities from write/delete capabilities where possible.

## Why This Is Interesting

Good collaboration tools reduce the distance between minds without pretending all work is just "a meeting." Software work has state: files, terminals, servers, breakpoints, design frames, scenes, logs, tickets, weird local scripts. The tool either carries that state across the network, or the humans have to describe it manually like medieval cartographers.

The best tools make context portable. The worst ones make you narrate your filesystem over video.

## Sources And Trails

- [tmate website](https://tmate.io/)
- [Zellij web client docs](https://zellij.dev/documentation/web-client.html)
- [Zellij features](https://zellij.dev/features/)
- [Visual Studio Live Share](https://visualstudio.microsoft.com/services/live-share/)
- [Visual Studio Live Share docs](https://learn.microsoft.com/en-us/visualstudio/liveshare/use/share-project-join-session-visual-studio)
- [Visual Studio Live Share FAQ](https://learn.microsoft.com/en-us/visualstudio/liveshare/faq)
- [Tuple website](https://tuple.app/)
- [Curiosity Cabinet: MCP Tooling](mcp-tooling.md)
- [Curiosity Cabinet: Remote Development Anywhere](remote-development-anywhere.md)
- [Curiosity Cabinet: Terminal Multiplexers](terminal-multiplexers.md)
