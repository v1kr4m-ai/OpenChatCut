# OpenChatCut: how to use it

Local-first AI video editor with a multitrack timeline. An agent (Claude Code, Codex, or the built-in agent) can read and edit real projects through MCP.

Installed at `D:\Claude\Claude_Videos\OpenChatCut` (cloned from github.com/0xsline/OpenChatCut, dependencies installed with `npm install`).

## 1. Start the editor

```powershell
cd D:\Claude\Claude_Videos\OpenChatCut
npm run dev
```

Leave that window running. Open http://localhost:5199 in Chrome or Edge (must be `localhost`, not a LAN IP).

Logs from the last start are in `D:\Claude\Claude_Videos\openchatcut_dev.log`.

Known issues:
- `package.json` requires Node 24.22.1 to 24.x. This machine runs Node 25.8.2, so `npm install` printed an engine warning. If anything breaks at runtime, install Node 24 and rerun `npm install`.
- The log shows a Remotion "not a valid Remotion project" error on port 3000 during startup. It comes from the Remotion render check, not the editor page, and the editor loads (HTTP 200).
- Config lives in `OpenChatCut\.env.local` (copied from `.env.example`). Add only the API keys you use (ElevenLabs, AssemblyAI, fal, and so on). Local timeline editing works without them.

## 2. Connect Claude Code (MCP)

The MCP endpoint always requires a bearer token. The token is generated when the editor starts and is shown only in the editor under **Settings → MCP**. It changes on every restart unless you set `OPENCHATCUT_MCP_TOKEN`.

Register it once per editor start:

```powershell
claude mcp add --transport http -H "Authorization: Bearer <token>" openchatcut http://localhost:5199/api/external-mcp/mcp
```

Check it: `claude mcp list` should show `openchatcut`. If the token changes after a restart, remove and re-add:

```powershell
claude mcp remove openchatcut
claude mcp add --transport http -H "Authorization: Bearer <new-token>" openchatcut http://localhost:5199/api/external-mcp/mcp
```

Do not put the token in any file under the repo.

## 3. Use it from Claude Code

Skill installed: `openchatcut` (global, in `~\.agents\skills\openchatcut`).

Start a session and say, for example:
- "Set up OpenChatCut" (first time)
- "List my OpenChatCut projects"
- "Open project X and cut the silences in the first clip"

The agent follows this order: `openchatcut_status` → `list_projects` → `load_skill` → `begin_edit_session` → edits → `review_edit_session` → `get_edit_session` (must say `applied`).

Default approval is `manual`: you approve the proposal inside the OpenChatCut window. Use `auto` only when you explicitly want unattended edits.

## 4. Use it directly in the browser

Open the project in http://localhost:5199. The left panel is the chat with the built-in agent (needs an API key set under **Settings → Agent model**). The middle is the preview, the bottom is the multitrack timeline. Every change is undoable.

Export from inside the editor. Renders use the Remotion/ffmpeg pipeline.
