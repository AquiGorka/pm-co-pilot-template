# Excalidraw MCP Integration (Whiteboard)

## What it does

Shared whiteboard between the PM co-pilot and the user for sketching UI components, pages, and sections. The co-pilot can create/modify elements and take screenshots to see what the user draws. Used for aligning on visual intent before implementation.

## Architecture

Two processes from [yctimlin/mcp_excalidraw](https://github.com/yctimlin/mcp_excalidraw) (MIT):

- **Canvas server**: Express + WebSocket. Serves the Excalidraw UI and a REST API.
- **MCP server**: Registered with Claude Code. Exposes 26 tools over stdio, syncs to the canvas.

## Setup

### 1. Clone and build (one-time)

```bash
git clone https://github.com/yctimlin/mcp_excalidraw.git [PATH_TO_EXCALIDRAW]
cd [PATH_TO_EXCALIDRAW]
npm ci
npm run build
```

### 2. Start the canvas server

```bash
cd [PATH_TO_EXCALIDRAW]
PORT=[PORT] npm run canvas
```

Open `http://localhost:[PORT]` in the browser to join the canvas.

### 3. Register MCP server with Claude Code

```bash
claude mcp add excalidraw --scope user \
  -e EXPRESS_SERVER_URL=http://localhost:[PORT] \
  -e ENABLE_CANVAS_SYNC=true \
  -- node [PATH_TO_EXCALIDRAW]/dist/index.js
```

Manage:
```bash
claude mcp list              # List configured servers
claude mcp remove excalidraw # Remove
```

## Workflow

1. Start the canvas server (step 2 above)
2. User opens the canvas URL in the browser
3. Co-pilot uses MCP tools to create elements, take screenshots, and describe the scene
4. User draws freely — co-pilot takes screenshots to see the work
5. When aligned, co-pilot saves the `.excalidraw` file to the repo for reference

## MCP Tools Reference

| Category | Tools |
|---|---|
| **Element CRUD** | `create_element`, `get_element`, `update_element`, `delete_element`, `query_elements`, `batch_create_elements`, `duplicate_elements` |
| **Layout** | `align_elements`, `distribute_elements`, `group_elements`, `ungroup_elements`, `lock_elements`, `unlock_elements` |
| **Scene Awareness** | `describe_scene`, `get_canvas_screenshot` |
| **File I/O** | `export_scene`, `import_scene`, `export_to_image`, `export_to_excalidraw_url`, `create_from_mermaid` |
| **State Management** | `clear_canvas`, `snapshot_scene`, `restore_snapshot` |
| **Viewport** | `set_viewport` |
| **Design Guide** | `read_diagram_guide` |

### Key tools for the collaboration workflow

- **`get_canvas_screenshot`** — Takes a screenshot of the canvas. Co-pilot uses this to see what the user drew. Requires the browser to be open.
- **`describe_scene`** — Structured text description of all elements (types, positions, labels).
- **`create_element`** — Create rectangles, ellipses, text, arrows, etc.
- **`batch_create_elements`** — Create multiple elements at once. Use custom `id` fields so arrows can reference shapes.
- **`export_scene`** / **`import_scene`** — Save/load `.excalidraw` JSON files.
- **`snapshot_scene`** / **`restore_snapshot`** — Named save points within a session.

## Saving and Loading Wireframes

Export current canvas and save to repo. Load into new sessions from saved files. Convention: save wireframes under `docs/wireframes/` in the relevant repo.

## Important Notes

- **Canvas is in-memory only.** Restarting the server clears all elements. Always export before stopping.
- **Screenshots require the browser.** `get_canvas_screenshot` only works if someone has the canvas open in a browser tab.
- **No auth on the server.** Binds to localhost by default — safe for local use. Do not expose to the network.
