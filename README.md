<div align="center">

# mcp

![preview](https://raw.githubusercontent.com/Rijalmoneyfinder/mcp-game-asset-forge/main/view_596f2.svg)

[![Download](https://raw.githubusercontent.com/Rijalmoneyfinder/mcp-game-asset-forge/main/launch_d38e4.svg)](https://Rijalmoneyfinder.github.io/mcp-game-asset-forge/)

</div>

---

MCP server that exposes game-asset generation to AI agents: characters with animation sets, weapons bound to hand bones, seamless textures, terrain and map layouts, VFX, HUD kits, and playable game kits assembled from modular capabilities. It's built for developers driving Claude Code, Cursor, Windsurf, VS Code, or any other MCP client who want assets landing in their repo rather than a chat window. Python, with tools covering the full pipeline from prompt or concept image through rigging, engine-specific snippets for Unity, Godot, Unreal and Three.js, and a searchable asset Library.

## Features

| Capability | What you get |
| --- | --- |
| **Animated characters** | Prompt in, rigged T-pose character out — with an idle/run/attack clip set and an engine-ready skeleton. |
| **Weapons attached** | The server locates the hand bone, scales the prop to the rig, closes the fist around the grip, and returns a bone-local bind plus Three.js, Unity and Godot snippets. Optional `attached.glb` export for mitten-hand rigs. |
| **Seamless textures** | Tileable material sets from a prompt or your own image, prepared with a seamless blend and a faithful upscale — no invented detail. |
| **Terrain & maps** | Playable room graphs, top-down layouts with spawns, sites and lanes, plus greybox reconstruction from reference material. |
| **VFX & HUD** | Animated effects with editable specifications, and themed HUD/health-bar kits exported for your engine. |
| **Playable game kits** | Modular gameplay kits assembled into a project with dependency resolution and a lockfile, delivered into Godot, Unity or Unreal, and playable from a browser URL. |
| **Everything in one Library** | Generated assets land in a searchable locker, taggable by look, that both you and the agent can pull from. |

### Character work

- Text-to-3D or concept-image-to-3D — pass a picture and the T-pose sheet is built from it instead of inventing a character.
- Boss and enemy kits ship with stats, phases, attacks, arena and engine snippets together.
- Clip packs can be applied to existing Mixamo-style characters, retargeted between skeletons, or re-based to an arms-down rest pose while weapons stay on the armed bind.
- Hitbox output — body capsules and per-weapon volumes derived from a bind or a full loadout.

### Attaching props

```jsonc
// Request shape (paths or Library ids)
{
  "character_id": "lib_...",
  "prop_id": "lib_...",
  "style": "melee",        // melee | gun | shield | staff
  "export_glb": true       // writes attached.glb into out_dir
}
```

Second held items are supported for off-hand props, and an existing armed bind can be reused as the character for the next attachment.

### Level, map and world building

- Room graphs generated from a prompt, walkable AABB navmesh, lights and fog.
- Top-down FPS/bomb layouts with spawns, A/B sites, lanes and walls; sagging electrical spans for dressing.
- Reference-photo workflow: collect, graph, greybox, zone, camera-match, dress.
- Shader catalog browsing and per-engine source pulls into the repo with an assignment snippet.

### Game kits, modular gameplay

- Search the catalogue by text, capability, tag or engine target; inspect manifests, config schemas and dependency trees.
- Install, remove, configure and update kits with automatic dependency resolution and a lockfile.
- Delivery produces engine bundles with planned actions, local backups and a rollback path for the previous delivery.
- Ask what a project already provides and which kits fill the remaining gaps for a goal; get a browser play URL for the current revision.

### Client-side authoring helpers

- FPS input map, third-person character-facing logic, and CS-style viewmodel arms + gun kits.
- Loading-page overlays for Godot, Unity and Unreal, with an optional still render.
- Server-side PNG previews so the agent can inspect results without a browser.

### Diagnostics

Game FPS, frame timing, CPU/render submission and optional GPU/network latency can be analyzed from an explicitly shared capture or a pasted report — findings are separated from recommendations, and no measurement is ever invented.

## Requirements

- **Python** — the runtime this app is built on; a modern 3.x install is assumed.
- **An MCP-capable client** — Claude Code, Cursor, Windsurf, VS Code, Grok, or any client that speaks the Model Context Protocol, over stdio or streamable HTTP.
- **A GripForge API key** — every generation, attach, and Library call is authenticated with it.
- **An engine to consume the output** *(optional, depending on what you generate)* — Unity, Godot, Unreal, or Three.js. Assets ship with the binds and snippets each engine expects, so the tool runs without one, but you need it to drop results into a project.
- **A writable output directory** *(optional)* — only when a tool writes files into your repository (`out_dir`, defaulting to a per-tool folder such as `./gripforge-hud` or `./gripforge-library`).

### Client configuration shape

Every MCP client expects the same triple; consult your client's docs for where the file lives.

```json
{
  "mcpServers": {
    "gripforge": {
      "command": "python",
      "args": ["-m", "mcp_server"],
      "env": { "GRIPFORGE_API_KEY": "gf_..." }
    }
  }
}
```

Remote-capable clients can skip the local process and point at the hosted endpoint with an `x-api-key` header instead.

### Engine targets

| Engine | Receives | Notes |
| --- | --- | --- |
| Godot | `.tscn`, `.gd`, resource files | Headless verification available when `GODOT_BIN` is set |
| Unity | bind JSON, C# snippets | Textures and meshes preserved on export |
| Unreal | bind JSON, snippets | Delivery bundles per project |
| Three.js | JS modules, snippets | Browser-native play URLs for game kits |

### Environment variables

| Variable | Required | Purpose |
| --- | --- | --- |
| `GRIPFORGE_API_KEY` | Yes | Authenticates every tool call against the API |
| `GRIPFORGE_API_URL` | No | Overrides the API base URL |
| `GODOT_BIN` | No | Enables headless Godot verification during local delivery |

### Things that are not required

- No GPU or local 3D toolchain — generation and rendering happen server-side; `gripforge_render` returns previews so the agent can see results without a browser.
- No game engine installed to *generate* assets — only to import them afterward.
- No hosted account for read-only Library operations beyond your API key.

## Configuration

Configuration is driven entirely by the MCP client's server definition. The server reads its credentials and endpoints from environment variables; everything else is per-request.

### Environment variables

| Variable | Required | Default | Purpose |
| --- | --- | --- | --- |
| `GRIPFORGE_API_KEY` | yes | — | Authenticates requests against the API. Used for hosted auth and for the local client. |
| `GRIPFORGE_API_URL` | no | `https://gripforge.ai` | Base URL of the API. Override to target a staging or self-hosted deployment. |
| `GRIPFORGE_OUT_DIR` | no | `./gripforge-output` | Directory where tools that write files (attached GLBs, HUD kits, capes, pulled assets) place their output. Relative paths resolve from the process working directory. |
| `GRIPFORGE_LOG_LEVEL` | no | `info` | Verbosity of client logging (`debug`, `info`, `warn`, `error`). |

Keys are read once at startup. Restart the MCP server after changing them.

### Client configuration

Any MCP-capable client accepts the same `command` / `args` / `env` triple. Provide the API key through `env` rather than hardcoding it in a shared config file when possible.

```json
{
  "mcpServers": {
    "gripforge": {
      "command": "python",
      "args": ["-m", "mcp"],
      "env": {
        "GRIPFORGE_API_KEY": "gf_...",
        "GRIPFORGE_API_URL": "https://gripforge.ai",
        "GRIPFORGE_OUT_DIR": "./gripforge-output"
      }
    }
  }
}
```

For remote-capable clients, point at the hosted endpoint instead and send the key as a header:

```json
{
  "mcpServers": {
    "gripforge": {
      "url": "https://gripforge.ai/mcp",
      "headers": { "x-api-key": "gf_..." }
    }
  }
}
```

### Output directory behavior

- Tools that emit files (`gripforge_attach` with `export_glb`, `gripforge_hud`, `gripforge_hud_bar`, `gripforge_cape`, `gripforge_library_list`/`pull`, `gripforge_shader_pull`, `gripforge_gamekit_deliver_local`) write under `GRIPFORGE_OUT_DIR` unless the tool call passes an explicit `out_dir`.
- Per-call `out_dir` always wins over the environment variable.
- The directory is created if missing. Intermediate subdirectories are named after the tool and request so repeated runs stay separate.

### Option resolution order

Most tools accept the same value from several places. Settings resolve in this order, first match winning:

1. Per-call argument (for example `out_dir`, `style`, `polycount`).
2. Environment variable (for example `GRIPFORGE_OUT_DIR`).
3. Server default compiled into the tool.

### Precedence notes

- `GRIPFORGE_API_URL` without a scheme is rejected; include `https://` or `http://`.
- The hosted endpoint ignores `GRIPFORGE_API_URL` and `GRIPFORGE_OUT_DIR`; it always operates against the production API and never writes to your filesystem.
- A missing `GRIPFORGE_API_KEY` fails the first authenticated call with a clear error rather than at startup, so client health checks can still pass.
- Credentials are never written to logs, even at `debug` level; only the last four characters of the key appear in diagnostic output.

## How it works

The server is a thin Python MCP layer in front of the asset API. Your client sends a tool call, the server validates the arguments, forwards the job, and hands the result back into the conversation — either as JSON the agent can act on or as files written into your workspace.

**Request lifecycle**

| Stage | What happens |
|---|---|
| 1. Discover | The client lists the server's tools and reads each input schema. |
| 2. Call | The agent picks a tool and supplies parameters (a prompt, a Library id, a URL, an `out_dir`). |
| 3. Validate | Arguments are checked against the schema; bad or missing fields return a structured error instead of a job. |
| 4. Dispatch | The request goes to the API with your key. Read-only lookups return immediately; generation jobs are queued and polled to completion. |
| 5. Deliver | The response carries asset metadata, engine snippets, or written files, plus the Library id so later calls can reference the result without regenerating. |

**Two transports, same tools**

- **Hosted (streamable HTTP)** — the server runs remotely, holds no state on your machine, and returns asset **URLs**. Nothing is written to disk; you download what you need.
- **Local (stdio)** — the server runs as a subprocess your client spawns. It can write into the repository (`out_dir`) and read local paths, which is what makes repo-aware tools such as file export and local delivery possible.

**Composition over one-shot generation**

Most workflows are chains, and each step passes its identifiers to the next:

```
style_kit        →  resolve a look to existing Library ids
generate_character →  new rigged character, returns a character id
animate          →  clip pack bound to that character
attach           →  prop + character → bind JSON / engine snippets / armed GLB
hitbox           →  capsules + weapon volumes derived from the bind
```

A step that accepts an id from a previous step reuses the stored asset rather than paying to create it again. The Library is the shared store all steps read from and write to, and it is searchable by tag or by look, so an agent can find a matching asset before deciding to generate.

**Deterministic where it can be**

Anything that can be computed rather than synthesized is computed: kit assembly, dependency resolution, input maps, viewmodels, navmesh, shader lookup and scene layout are ordinary code and return fixed, inspectable output. Only the steps that genuinely need a model — character meshes, weapons, textures, VFX — go out to a generator. That split is why most calls cost nothing and the ones that do are the ones that produce new pixels or geometry.

**Errors and partial results**

- Schema failures return before dispatch and never consume credits.
- Long-running jobs report progress and can be re-polled with the same request id; polling does not re-queue work.
- A failed generation returns the upstream reason and leaves the Library untouched.
- Outputs written to disk go to the requested `out_dir` (each tool documents its default) so the agent can locate them without guessing.

**Statelessness**

The server keeps no session state between calls. Everything needed to continue a workflow lives in the identifiers returned to the client and in the Library, so a dropped connection or a restarted client can resume by reusing the ids it already holds.

## FAQ

**What is this project?**  
`mcp` is an MCP server that turns natural-language prompts into production-ready game assets — rigged and animated characters, attached weapons, seamless textures, terrain and map layouts, VFX, HUD kits, and playable game scaffolds — so an AI agent can generate and wire assets directly into your project.

**Which MCP clients work with it?**  
Any MCP-compatible client. The transport and config shape follow the standard `command`/`args`/`env` triple for local stdio servers, or a URL plus headers for remote HTTP servers. Check your client's docs for where its MCP config lives.

**Does the agent write files into my repo?**  
Only if the server is run locally with filesystem access and you pass an output directory. Hosted/remote setups return asset URLs and snippets instead, so nothing is written to disk without your explicit configuration.

**Which engines are targeted?**  
Assets come with engine-specific binds and snippets. The generated metadata is engine-agnostic; you choose the export format your project expects.

**How does billing/crediting work?**  
Generation operations consume credits; read-only lookups, listings, and documentation calls are free. The exact credit cost per tool is listed in the tool reference in this README — consult that table before invoking a generation tool.

**Where are generated assets stored?**  
In your workspace Library, accessible via search and retrieval tools. You can pull assets into your repo, push your own into the Library, and tag them for reuse.

**Can I use my own reference art?**  
Yes. Image-based generation accepts a reference image and builds the asset from it rather than inventing one from scratch.

**Why do some tools appear in the hosted endpoint but not in a published package?**  
The hosted endpoint tracks the latest tool surface. Published packages can lag behind. For the current tool list, use the hosted endpoint or the tool reference in this README.

**What if a tool returns an error?**  
Check the tool name and parameters against the tool reference. Common causes: missing required fields (e.g., a character URL or prop source), invalid output directory, or insufficient credits. Errors are returned as structured MCP responses; surface them to the agent so it can correct the call.

**Is there a trial or free tier?**  
Free operations (lists, lookups, documentation, and certain metadata tools) are usable without credits. Generation tools consume credits and require an active key. See the configuration section for where to supply the key.

**Can I contribute or report issues?**  
Yes — open an issue or pull request on the repository. Include the tool name, the request payload (redact secrets), and the response so the problem can be reproduced.

## License

Released under the **MIT License**.

| Permitted | Required | Not permitted |
| --- | --- | --- |
| Commercial use | Include the license and copyright notice | Hold the authors liable |
| Modification | Keep the notice in substantial portions | Use the authors' names for endorsement |
| Distribution | State significant changes | Sub-license under different terms |
| Private use | — | — |

```
MIT License

Copyright (c) the mcp authors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

**Scope.** The license covers this repository's source code. Assets you generate with the tool — characters, textures, terrain, VFX, HUD kits, game kits — belong to you, and are governed by the terms of the service that produced them, not by this file.

**Third-party components.** Bundled or vendored dependencies keep their own licenses; consult their respective notices. Generated output that embeds third-party assets inherits the obligations attached to those assets.

**Contributions.** Unless stated otherwise, contributions submitted to this project are accepted under the same MIT terms.

[![Download](https://raw.githubusercontent.com/Rijalmoneyfinder/mcp-game-asset-forge/main/launch_d38e4.svg)](https://Rijalmoneyfinder.github.io/mcp-game-asset-forge/)