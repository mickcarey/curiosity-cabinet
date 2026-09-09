# MCP Tooling: Giving AI Agents Hands Without Giving Them The Keys To The Volcano 🌋

## The Short Version

**MCP** stands for **Model Context Protocol**. It is an open standard for connecting AI tools to the places where useful work actually lives: code repositories, issue trackers, docs, design files, databases, browsers, 3D editors, game engines, and local scripts.

The easiest mental model: MCP is like a plugboard for AI tools.

- The **AI app** is the host: Codex, Claude, Cursor, VS Code, ChatGPT, etc.
- The **MCP client** is the connector inside that app.
- The **MCP server** exposes capabilities: tools, resources, prompts.
- The **human** remains the adult in the room, approving risky actions.

## Why MCP Exists 🧠

LLMs are clever, but by default they are trapped behind glass. They can reason about Jira tickets if you paste them in. They can reason about Figma designs if you screenshot them. They can suggest Blender Python if you describe the scene.

That is useful, but clumsy.

MCP says: instead of every AI app inventing a one-off integration for every tool, give everyone a common protocol. One clean doorway per tool, many agents can walk through it.

It is inspired by the same ecosystem logic as the Language Server Protocol: editors got smarter when language tooling had a standard way to talk. AI agents get more useful when work tools have a standard way to talk.

## The Three MCP Primitives 🧰

| Primitive | What It Means | Example |
|---|---|---|
| **Tools** | Actions the model can call | Create Jira issue, list selected Unreal actors, render Blender scene |
| **Resources** | Readable context | Confluence page, design variables, repo files, schema docs |
| **Prompts** | Reusable workflows | "Turn meeting notes into Jira tickets" |

Tools are the spicy one. They can change things. Good clients show tool calls clearly and ask before dangerous operations. Badly scoped tools are how an innocent "tidy this up" becomes "why is half the project gone?"

## The Origin Story

Anthropic introduced MCP in November 2024 as an open standard for connecting AI assistants to external systems. The announcement credits **David Soria Parra** and **Justin Spahr-Summers** as creators.

The timing made sense. Coding agents were getting useful enough to act, but acting without context is how you get confident nonsense. MCP attacked the integration problem: let agents fetch the right context and call the right tool instead of relying on paste archaeology.

## Jira And Confluence: Atlassian Rovo MCP 🗂️

Atlassian's official remote MCP server connects AI tools to Jira, Confluence, Jira Service Management, Bitbucket, Compass, Loom, and broader Atlassian platform data.

What it is good for:

- Search Confluence without leaving your agent.
- Summarize a spec and create Jira work items.
- Pull ticket acceptance criteria into a coding session.
- Update issue fields after implementation.
- Turn meeting notes into tracked work.

Example prompts:

- "Find the Confluence page for the inventory sync design and summarize the risky assumptions."
- "Create Jira subtasks for the implementation plan in this markdown file."
- "Compare this branch diff with the linked Jira acceptance criteria."
- "Draft a Confluence release note from these commits and linked issues."

The important bit: the server respects existing Atlassian access controls. MCP does not magically grant permission; it gives the agent a protocol-shaped way to use permissions you already have.

## Figma MCP: Design Context Without Screenshot Soup 🎨

Figma's MCP server brings design context into coding tools. Instead of feeding an agent a flattened screenshot and hoping it guesses spacing, tokens, layers, and component intent, the agent can ask for structured Figma context.

Useful flows:

- Generate UI code from selected frames.
- Pull design variables and component data into the IDE.
- Use Code Connect so generated code maps to real production components.
- Write native Figma content back to the canvas when supported.

Example prompts:

- "Use the selected Figma frame and implement this with our existing button/card components."
- "Extract the design variables used in this flow and compare them to our theme tokens."
- "Create a first pass component, but reuse mapped Code Connect components where available."
- "Write a rough Figma frame for this empty state so design can edit it."

This is one of the cleanest examples of MCP's value: design-to-code gets much better when the AI can see design structure, not just pixels.

## Blender MCP: Natural Language Meets `bpy` 🧊

Blender's official MCP Lab server is a lightweight bridge from AI tools into Blender's Python API. It is explicitly powerful and explicitly risky: generated code runs inside Blender without guard rails, so the Blender project recommends using a VM or a system without sensitive data.

Useful flows:

- Inspect complex scenes.
- Generate or modify objects.
- Explain material/node setups.
- Automate repetitive asset cleanup.
- Render, inspect, revise.

Example prompts:

- "List the objects in this scene and identify anything with unapplied scale."
- "Create three material variants for this prop: worn metal, clean plastic, painted ceramic."
- "Add labels to the major parts of this mechanical model for a tutorial render."
- "Render a clay preview from camera 2 and save it to `/tmp/preview.png`."

The correct vibe is "assistant with a power drill," not "assistant with root on your life." Keep sensitive files elsewhere.

## Unreal MCP: AI Inside The Editor Loop 🎮

Epic's Unreal MCP documentation for Unreal Engine 5.8 describes an experimental plugin that embeds an MCP server inside the Unreal Editor. It exposes editor functionality as tools: spawning actors, configuring lighting, creating material instances, inspecting Slate widgets, and running automation tests.

For UE 5.8.2-style work, that means an AI agent can potentially help with editor automation while the actual engine stays open on the workstation.

Useful flows:

- Ask what actors are selected.
- Spawn prototype scene objects.
- Configure lights or materials.
- Run automation tests.
- Generate or refresh an MCP client config from the editor.

Example prompts:

- "What actors are selected, and what components do they have?"
- "Create a simple greybox room with a door, two lights, and a player start."
- "Make material instances for these assets using the project's standard parameters."
- "Run the relevant automation tests and summarize failures."

Big caution: Epic labels Unreal MCP experimental, local by default, and not safe to expose remotely. That is exactly right. Use SSH/Tailscale to reach the workstation; do not publish the editor's MCP endpoint to the open internet like a cursed piñata.

## GitHub, Docker, And The MCP Tool Shed 🧱

GitHub has an official MCP server for repository context and actions. Docker has an MCP Catalog and Toolkit for finding and running MCP servers in a more packaged, isolated way.

This matters because local MCP can get messy:

- Every server wants dependencies.
- Every dependency wants a version.
- Every token wants storage.
- Every client wants config.

Docker's pitch is: package MCP servers as containers, manage profiles, and reduce the "my agent has twelve weird little background daemons" problem.

Example useful server set:

- GitHub: issues, PRs, code scanning, repository context
- Atlassian: Jira and Confluence
- Figma: design context
- Playwright/browser: inspect and test web UI
- Filesystem/database: local project context, with tight permissions
- Blender/Unreal: local creative/editor automation

## A Practical MCP Stack For Your Workflow 🧭

Given the tools you use, the neat stack is:

| Area | MCP Direction |
|---|---|
| Jira | Search issues, create/update tickets, pull acceptance criteria |
| Confluence | Read specs, summarize docs, draft pages |
| Figma | Pull design context, map components, write canvas drafts |
| Blender | Inspect/create scene elements, render previews, explain setups |
| Unreal 5.8.2 | Drive editor tools locally, run automation, inspect selected actors |
| GitHub/Git | Connect work to branches, PRs, commits, review context |
| Remote dev | Run the agent on the powerful machine inside tmux/Zellij |

The nice architecture:

```text
iPhone/iPad
  -> SSH/Mosh/Tailscale
    -> powerful workstation
      -> tmux or Zellij
        -> AI agent
          -> MCP servers for Jira, Confluence, Figma, Blender, Unreal, GitHub
```

The phone stays light. The workstation holds credentials, engines, repos, GPU tools, and persistent sessions.

## MCP Safety Rules That Are Actually Useful 🔐

- Prefer official servers for critical SaaS tools.
- Give the narrowest permissions that still make the workflow useful.
- Separate read-only research servers from write-capable action servers when possible.
- Keep Blender and Unreal MCP loopback/local unless you deeply know what you are doing.
- Read tool calls before approving writes.
- Treat "delete," "publish," "merge," "send," and "charge money" as red-button verbs.
- Use project-specific MCP profiles so a game-dev agent does not automatically have production finance tools for no reason.

## Fun Use Cases Worth Trying

### Spec To Work Plan

1. Agent reads Confluence spec.
2. Agent finds related Jira epic.
3. Agent drafts implementation tasks.
4. You approve the shape.
5. Agent creates Jira subtasks.

### Design To First Pass

1. Agent reads selected Figma frame.
2. Agent maps design components to code components.
3. Agent implements the view.
4. Agent runs Playwright.
5. Agent posts a short summary back to Jira.

### Unreal Greybox Sprint

1. Agent connects to local Unreal MCP.
2. You describe a prototype room.
3. Agent spawns actors/materials/lights.
4. Agent runs automation checks.
5. You review in-editor on the workstation, even if you are steering from a phone.

### Blender Asset Cleanup

1. Agent inspects scene objects.
2. Agent finds naming, scale, material, or collection mess.
3. You approve a cleanup plan.
4. Agent applies repeatable changes.
5. Agent renders a preview.

## Why MCP Is Interesting

MCP is not exciting because it lets AI "do everything." That is also the danger.

It is exciting because it makes context portable. Your agent can stop being a brilliant amnesiac in a blank room and start being a weirdly fast assistant standing in the actual workshop, looking at the actual tickets, designs, files, scenes, and editor state.

The future is not one mega-agent with every permission in the kingdom. The future is smaller, sharper tool access: the right agent, in the right project, with the right context, doing one useful thing at a time.

## Sources And Trails

- [Anthropic: Introducing the Model Context Protocol](https://www.anthropic.com/news/model-context-protocol)
- [Anthropic docs: Model Context Protocol](https://docs.anthropic.com/en/docs/mcp)
- [MCP specification](https://modelcontextprotocol.io/specification/2024-11-05/index)
- [MCP tools specification](https://modelcontextprotocol.io/specification/draft/server/tools)
- [MCP Registry overview](https://modelcontextprotocol.io/registry/about)
- [Atlassian: Remote MCP Server announcement](https://www.atlassian.com/blog/announcements/remote-mcp-server)
- [Atlassian Rovo MCP Server docs](https://atlassian.github.io/atlassian-mcp-server/)
- [Figma: Introducing the Figma MCP server](https://www.figma.com/blog/introducing-figma-mcp-server/)
- [Figma MCP developer docs](https://developers.figma.com/docs/figma-mcp-server/)
- [Blender Lab: MCP Server](https://www.blender.org/lab/mcp-server/)
- [Unreal Engine 5.8: Unreal MCP](https://dev.epicgames.com/documentation/unreal-engine/unreal-mcp-in-unreal-editor)
- [GitHub changelog: GitHub MCP server public preview](https://github.blog/changelog/2025-04-04-github-mcp-server-public-preview/)
- [Docker MCP Catalog and Toolkit](https://docs.docker.com/ai/mcp-catalog-and-toolkit/)
- [Curiosity Cabinet: Remote Development Anywhere](remote-development-anywhere.md)
