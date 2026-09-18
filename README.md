# 🎬 After Effects MCP Server

![Node.js](https://img.shields.io/badge/node-%3E=14.x-brightgreen.svg)
![Build](https://img.shields.io/badge/build-passing-success)
![License](https://img.shields.io/github/license/Dakkshin/after-effects-mcp)
![Platform](https://img.shields.io/badge/platform-after%20effects-blue)

✨ A Model Context Protocol (MCP) server for Adobe After Effects that enables AI assistants and other applications to control After Effects through a standardized protocol.

<a href="https://glama.ai/mcp/servers/@Dakkshin/after-effects-mcp">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@Dakkshin/after-effects-mcp/badge" alt="mcp-after-effects MCP server" />
</a>

## Table of Contents
- [Features](#features)
  - [Core Composition Features](#core-composition-features)
  - [Layer Management](#layer-management)
  - [Animation Capabilities](#animation-capabilities)
- [Setup Instructions](#setup-instructions)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Update MCP Config](#Update-MCP-Config)
  - [Running the Server](#running-the-server)
- [Usage Guide](#usage-guide)
  - [Creating Compositions](#creating-compositions)
  - [Working with Layers](#working-with-layers)
  - [Animation](#animation)
- [Available MCP Tools](#available-mcp-tools)
- [For Developers](#for-developers)
  - [Project Structure](#project-structure)
  - [Building the Project](#building-the-project)
  - [Contributing](#contributing)
- [License](#license)

## 📦 Features

### 🎥 Core Composition Features
- **Create compositions** with custom settings (size, frame rate, duration, background color)
- **List all compositions** in a project
- **Get project information** such as frame rate, dimensions, and duration

### 🧱 Layer Management
- **Create text layers** with customizable properties (font, size, color, position)
- **Create shape layers** (rectangle, ellipse, polygon, star) with colors and strokes
- **Create solid/adjustment layers** for backgrounds and effects
- **Create camera layers** with configurable zoom and position
- **Create null objects** for animation control
- **Modify layer properties** like position, scale, rotation, opacity, timing
- **Toggle 2D/3D mode** for layers
- **Set blend modes** (normal, multiply, screen, etc.)
- **Track matte** support (alpha, luma, inverted)
- **Duplicate layers** with optional rename
- **Delete layers** from composition
- **Create/modify masks** with feather, expansion, and opacity

### 🌀 Animation Capabilities
- **Set keyframes** for layer properties (Position, Scale, Rotation, Opacity, etc.)
- **Apply easing** (easy ease / ease in / ease out) to existing keyframes
- **Apply expressions** to layer properties for dynamic animations
- **Batch set properties** across multiple layers at once
- **Trim Paths** on shape layers, including animated "draw-on"
- **Text animators** with range selectors for per-character animation

### ✨ Effects, Rendering & Project
- **Apply, remove, and reorder effects** on a layer
- **Create lights** (parallel, spot, point, ambient) and **reorder layers** in the stack
- **Set or clear a layer's parent** for rigging
- **Import footage** (image / video / audio) and optionally add it to a comp
- **Precompose** layers into a nested composition
- **Render to a video file** via the render queue, or **capture a single frame to PNG** (so the result can be inspected)
- **Inspect layer details** (effects, blend mode, parent, 3D, timing) and **save the project**

### 📁 Footage & Layer Ordering
- **Import footage** (image, audio, or video) into the project and optionally add it directly as a layer
- **Move layers** to the beginning or end of the layer stack
- **Force resampling quality** (Bicubic vs. Bilinear) when scaling a layer

### 🧰 Escape Hatch
- **Run arbitrary ExtendScript** (`evalScript`) for anything outside the fixed set of bridge commands — covers the full After Effects scripting API without needing a new named function and a rebuild for every capability. Grants the same local script/file access as running an ExtendScript file directly in After Effects, so only expose it to trusted callers.

## ⚙️ Setup Instructions

### 🛠 Prerequisites
- Adobe After Effects (2022 or later)
- Node.js (v14 or later)
- npm or yarn package manager

### 📥 Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Dakkshin/after-effects-mcp.git
   cd after-effects-mcp
   ```

2. **Install dependencies**
   ```bash
   npm install
   # or
   yarn install
   ```

3. **Build the project**
   ```bash
   npm run build
   # or
   yarn build
   ```

4. **Install the After Effects panel**
   ```bash
   npm run install-bridge
   # or
   yarn install-bridge
   ```
   This will copy the necessary scripts to your After Effects installation.

### 🔧 Update MCP Config

#### Option 1: Using .mcp.json (Recommended for Claude Code)
The repository includes a `.mcp.json` file for easy configuration. Copy or reference it in your MCP settings:

```json
{
  "mcpServers": {
    "AfterEffectsMCP": {
      "command": "node",
      "args": ["PATH/TO/after-effects-mcp/build/index.js"]
    }
  }
}
```

#### Option 2: Manual Configuration
Go to your client (e.g., Claude or Cursor) and update your config file:

```json
{
  "mcpServers": {
    "AfterEffectsMCP": {
      "command": "node",
      "args": ["C:\\Users\\Dakkshin\\after-effects-mcp\\build\\index.js"]
    }
  }
}
```

#### Optional: `AE_MCP_BRIDGE_DIR`
By default the server and the bridge panel exchange files in `~/Documents/ae-mcp-bridge`.
Set the `AE_MCP_BRIDGE_DIR` environment variable to point both sides at a custom folder —
useful when the MCP host and After Effects run in different OS contexts (for example, Node
in WSL driving After Effects on Windows) and need to rendezvous on a shared path:

```json
{
  "mcpServers": {
    "AfterEffectsMCP": {
      "command": "node",
      "args": ["PATH/TO/after-effects-mcp/build/index.js"],
      "env": { "AE_MCP_BRIDGE_DIR": "/mnt/c/Users/you/Documents/ae-mcp-bridge" }
    }
  }
}
```

The panel honors the same variable from After Effects' own environment; when it is unset,
the `~/Documents/ae-mcp-bridge` default must resolve to the same physical folder on both sides.

### ▶️ Running the Server

1. **Start the MCP server**
   ```bash
   npm start
   # or
   yarn start
   ```

2. **Open After Effects**

3. **Open the MCP Bridge Auto panel**
   - In After Effects, go to Window > mcp-bridge-auto.jsx
   - The panel will automatically check for commands every few seconds
   - Make sure the "Auto-run commands" checkbox is enabled

### ⚠️ A note on modal dialogs

The panel polls for commands using After Effects' own scheduled-task timer.
After Effects' scripting engine refuses to run *any* scheduled script while
*any* modal dialog is open anywhere in the app — including its own crash
reporter, missing-plugin warnings, or other alert boxes — and will pop up an
"Unable to execute script... Cannot run a script while a modal dialog is
waiting for response" alert of its own when that happens. This is a
restriction in After Effects' scripting engine, not something this bridge can
suppress or catch. If commands stop being picked up, check for (and close)
any open dialog in After Effects, including ones that may be hidden behind
the main window; the panel resumes polling on its own once nothing is
blocking it.

## 🚀 Usage Guide

Once you have the server running and the MCP Bridge panel open in After Effects, you can control After Effects through the MCP protocol. This allows AI assistants or custom applications to send commands to After Effects.

### 📘 Creating Compositions

You can create new compositions with custom settings:
- Name
- Width and height (in pixels)
- Frame rate
- Duration
- Background color

Example MCP tool usage (for developers):
```javascript
mcp_aftereffects_create_composition({
  name: "My Composition", 
  width: 1920, 
  height: 1080, 
  frameRate: 30,
  duration: 10
});
```

### ✍️ Working with Layers

You can create and modify different types of layers:

**Text layers:**
- Set text content, font, size, and color
- Position text anywhere in the composition
- Adjust timing and opacity

**Shape layers:**
- Create rectangles, ellipses, polygons, and stars
- Set fill and stroke colors
- Customize size and position

**Solid layers:**
- Create background colors
- Make adjustment layers for effects

### 🕹 Animation

You can animate layers with:

**Keyframes:**
- Set property values at specific times
- Create motion, scaling, rotation, and opacity changes
- Control the timing of animations

**Expressions:**
- Apply JavaScript expressions to properties
- Create dynamic, procedural animations
- Connect property values to each other

## 🛠 Available MCP Tools

| Command                     | Description                            |
|-----------------------------|----------------------------------------|
| `create-composition`        | Create a new composition               |
| `run-script`                | Run a JS script inside AE              |
| `get-results`               | Get script results                     |
| `get-help`                  | Help for available commands            |
| `bridge-status`             | Read-only liveness probe: is the panel open and responding? |
| `setLayerKeyframe`          | Add keyframe to layer property         |
| `setLayerExpression`        | Add/remove expressions from properties|
| `setLayerProperties`        | Set layer properties (position, scale, rotation, opacity, blendMode, threeDLayer, trackMatteType, enabled, etc.) |
| `batchSetLayerProperties`  | Apply properties to multiple layers   |
| `getLayerInfo`              | Get layer info (position, 3D status)  |
| `createCamera`              | Create camera layer                   |
| `createNullObject`          | Create null object for animation      |
| `duplicateLayer`            | Duplicate a layer                     |
| `deleteLayer`               | Delete a layer                        |
| `setLayerMask`              | Create/modify layer masks             |
| `evalScript`                | Run an arbitrary ExtendScript snippet and return its result |
| `move-layer`                | Reorder a layer (front/back/before/after/to-index) |
| `set-parent`                | Set or clear a layer's parent          |
| `create-null`               | Create a null object layer             |
| `set-3d-layer`              | Toggle a layer's 3D switch             |
| `set-blend-mode`            | Set a layer's blending mode            |
| `set-track-matte`           | Set/clear a track matte (optionally to a specific layer) |
| `remove-effect`             | Remove an effect from a layer          |
| `reorder-effect`            | Reorder an effect in the effect stack  |
| `create-light`              | Create a light layer                   |
| `apply-trim-paths`          | Add Trim Paths to a shape (with draw-on) |
| `add-text-animator`         | Add a text animator (per-character)    |
| `set-keyframe-ease`         | Apply easing to a property's keyframes |
| `import-footage`            | Import a file into the project         |
| `precompose`                | Precompose layers into a nested comp   |
| `render-video`              | Render a composition to a video file   |
| `save-frame`                | Render a single frame to a PNG file    |
| `save-project`              | Save the current project               |
| `get-layer-details`         | Inspect a layer (effects, blend, parent, 3D) |
| `delete-composition`        | Delete a composition from the project  |
| `remove-keyframe`           | Remove keyframe(s) by time, index, or all |
| `get-renderer-info`         | List a comp's current & available 3D renderers |
| `set-renderer`              | Set a comp's 3D renderer (Classic 3D / Cinema 4D) |
| `create-folder`             | Create a project folder (idempotent)   |
| `move-item-to-folder`       | Move a comp, footage or folder into a folder |
| `set-item-label`            | Label colour of a project item         |
| `set-layer-label`           | Label colour of a layer                |
| `set-layer-comment`         | Comment column of a layer              |
| `rename-layer`              | Rename a layer                         |
| `rename-item`               | Rename a comp, footage or folder       |
| `list-project-items`        | List items with folder path and label  |
| `build-composition`         | Build a whole scene from one JSON spec (see below) |

### 🏗 build-composition: a whole scene from one spec

Building a UI card layer by layer over the bridge takes minutes (one command every few seconds) and leaves the caller guessing which layers already exist. `build-composition` takes one JSON spec and builds every composition, layer and keyframe in a single undo group. Pass `specFile` (absolute path) for anything beyond a handful of layers; `replace: true` removes compositions with the same names first so a rebuild does not fail on duplicates.

Coordinates follow the browser convention the spec is usually generated from: origin top-left, y down, sizes in comp pixels, times in seconds, colours as `#rrggbb`. Layers are listed bottom-to-top. `origin` is the point (fractions of the layer's own box) that scale keyframes pivot around, like CSS `transform-origin`.

```json
{
  "frameRate": 60, "duration": 3, "timeOffset": 0.5, "timeScale": 1,
  "folder": "02 Bouwstenen", "partsFolder": "PRE_card_parts",
  "comp": {
    "name": "PRE_card", "width": 800, "height": 400, "comment": "why this exists",
    "layers": [
      { "type": "rect", "name": "SHP card", "x": 0, "y": 0, "w": 800, "h": 400, "roundness": 16,
        "fill": "#ffffff", "stroke": { "color": "#e6e6e6", "width": 4 },
        "shadow": { "color": "#000000", "opacity": 10, "dx": 0, "dy": 4, "blur": 12 } },
      { "type": "text", "name": "TXT title", "x": 40, "baseline": 90, "text": "Group swim",
        "font": "Inter-SemiBold", "size": 40, "color": "#000000", "tracking": 0, "label": "orange" },
      { "type": "path", "name": "SHP icon", "x": 700, "y": 40, "w": 44, "h": 44,
        "shapes": [ { "paths": [ { "vertices": [[0,0],[44,0],[44,44]], "inTangents": [[0,0],[0,0],[0,0]], "outTangents": [[0,0],[0,0],[0,0]], "closed": true } ],
                      "fill": "#333333", "fillRule": "nonzero", "stroke": { "color": "#000000", "width": 2, "cap": "round", "join": "round" } } ] },
      { "type": "comp", "name": "PRE row", "x": 40, "y": 140, "w": 720, "h": 60, "origin": [0, 0.5],
        "comp": { "name": "PRE_card__row-1", "width": 720, "height": 60, "layers": [] },
        "anim": [
          { "prop": "opacity", "ease": [0.22, 1, 0.36, 1], "keys": [[0.18, 0], [0.6, 100]] },
          { "prop": "position", "ease": [0.22, 1, 0.36, 1], "keys": [[0.18, [0, 16]], [0.6, [0, 0]]] },
          { "prop": "scale", "keys": [[0.2, [0, 1]], [0.9, [1, 1]]] }
        ],
        "effects": [ { "matchName": "ADBE Venetian Blinds", "props": { "Transition Completion": 50, "Direction": 45, "Width": 16 } } ] }
    ]
  }
}
```

Layer types: `rect` and `ellipse` (`fill`, `fillOpacity`, `stroke`, `roundness`), `path` (a list of `shapes`, each with cubic-bezier `paths` in the layer's own coordinates plus its own fill/stroke), `text` (point text anchored at the start of its first `baseline`; `leading` and `align` optional) and `comp` (a nested composition placed as a precomp, with `collapseTransformations` on). Every layer accepts `name`, `label`, `comment`, `opacity`, `shadow`, `effects` and `anim`. Position keys are offsets from the layout position; scale keys are factors (1 = 100%). `ease` is a CSS `cubic-bezier` quadruple; it becomes speed/influence handles on the keyframes. `timeScale` stretches every key time, `timeOffset` shifts them.

> ℹ️ Output format for `render-video` is determined by the chosen output module template (or the default one); on some installs the default is H.264 (.mp4). `save-frame` is handy for letting an AI assistant inspect the rendered result.

## 👨‍💻 For Developers

### 🧩 Project Structure

- `src/index.ts`: MCP server implementation
- `src/scripts/mcp-bridge-auto.jsx`: Main After Effects panel script
- `install-bridge.js`: Script to install the panel in After Effects

### 📦 Building the Project

```bash
npm run build
# or
yarn build
```

**Note:** This project uses esbuild for fast builds, replacing the previous TypeScript compiler approach that could run out of memory on larger codebases.

### 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=Dakkshin/after-effects-mcp&type=date&legend=top-left)](https://www.star-history.com/#Dakkshin/after-effects-mcp&type=date&legend=top-left)

## License

This project is licensed under the MIT License - see the LICENSE file for details.
