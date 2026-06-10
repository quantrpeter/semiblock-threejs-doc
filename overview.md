# SemiBlock Three.js

SemiBlock Three.js is a visual, block-based editor for building interactive 3D scenes with [Three.js](https://threejs.org/). It brings the accessibility of Blockly-style programming to WebGL development, letting you assemble scenes, cameras, lights, geometries, materials, meshes, animations, and GLTF models by snapping blocks together instead of writing raw JavaScript.

The editor generates executable Three.js code on the fly and immediately evaluates it, giving you a live 3D preview as you build.

## Key Features

- **Live 3D Preview**: Every change to the workspace triggers code generation and `eval()` into the `#threeJSDiv` preview pane. See results instantly.
- **Three Focused Toolbox** (custom "quantr" theme):
  - **Javascript** (`#cdb4db`): General programming — `js_code` (multiline), `declare_const` / string / number variables, `variable_set` / `get`, `loop_for`, `if_else`, `string_concat`, `math_add`, `debug_log`.
  - **Scene** (`#ffafcc`): Core Three.js primitives — `scene_create`, `light_create` (Ambient/Point/Directional), `set_light_position`, `camera_create` (FOV/Aspect/Near/Far), `set_camera_position`, `set_background_color`, `create_renderer`, `render_scene_camera`, `set_animation_loop`, `create_animation_loop`, `function_animate`, geometry creators (`cylinder_create`, `ball_create`, `triangle_create`, `create_geometry`), material blocks (`create_material`, `create_mesh_normal_material`), `create_mesh`, `add_mesh_to_scene`, `set_mesh_position`, `rotate_mesh`, value helpers (`number_input`, `color_picker`), and animation support (`create_clock`, `create_mixer`, `get_animation_clip`, `create_action_from_clip`, `set_action_properties`).
  - **GLTF Loader** (`#b4cddb`): Async model loading — `gltf_loader_load`, `gltf_loader_on_load` (with body), `gltf_loader_on_progress`, `gltf_loader_on_error`.
- **Custom Category Icons**: Toolbox categories use colored SVG icons (`scene.svg`, `javascript.svg`, `gltfLoader.svg`) that adapt when selected.
- **Code Visibility & Export**: The generated JavaScript is always shown in the output pane. The host exposes `quantr.getCode()`, `quantr.getWS()`, `quantr.saveWS()`, `quantr.loadWS()`, `quantr.clearWorkspace()`, and `quantr.refreshScreen()`.
- **Persistence**: Workspaces are saved to `localStorage` (key `mainWorkspace`) via Blockly serialization. The parent platform can also inject serialized projects (base64) and call `loadWS`.
- **Integration Ready**: Built with webpack as a library (`quantr`) and served from `public/blockly-threeJS/build-production`. The sample host page loads Three.js r128 + GLTFLoader from CDN and provides the `#threeJSDiv`, `#generatedCode`, and `#output` containers.

## User Interface Layout

```
[ Clear ] [ Load (demo) ] [ Save ]
+-----------------------------------------------+
| Blockly Workspace (50%)   | Output Pane (400px) |  #threeJSDiv (600px)
|                           |   - Generated Code  |  (live WebGL canvas)
|   (drag & snap blocks)    |   - Console Output  |
+-----------------------------------------------+
```

- **Blockly canvas** uses the zelos renderer and 20 px grid snapping.
- **Output pane** shows pretty-printed generated code (top) and runtime `console.log` / errors (bottom).
- **3D pane** receives the renderer DOM element when you use the `create_renderer` block.

## How Code Generation Works

Blocks are defined as JSON arrays in `blocks/threejsBlocks.js` and registered with `Blockly.common.defineBlocks`. A custom generator (`threejsGenerator` in `generators/threejs.js`) implements `forBlock` handlers that emit raw `THREE.*` statements. A custom `workspaceToCode` + `generateCodeForBlock` walks top-level blocks and their `next` chains (plus `myStatementsToCode` for statement inputs such as animate bodies or GLTF callbacks).

On every workspace change the host calls `runCode()`:

1. `threejsGenerator.workspaceToCode(ws)` produces the JS string.
2. The string is placed into `#generatedCode`.
3. A `try { eval(code) } catch(e) { ... }` executes it, targeting the live `#threeJSDiv`.

Special conventions:
- `create_renderer` clears `#threeJSDiv` and appends the new `WebGLRenderer` canvas.
- `function_animate` defines a global `animate(time)` that the `set_animation_loop` block wires to `renderer.setAnimationLoop(animate)`.
- `create_clock` / `create_mixer` set `globalThis.clock` and `globalThis.mixer` so the animate helper can call `mixer.update(dt)`.
- GLTF loaders create lights and center the model automatically in the generated code.

## Place in the SemiBlock Ecosystem

SemiBlock Three.js sits alongside the other visual tools in the platform:

- JVM Blockly editor (for generating Java/Kotlin/Scala code)
- Flowchart designer (SVG canvas + Monaco JSON view)
- IoT dashboard / workflows
- Micropython, student cloud, and other documentation sets

It lowers the barrier for students and educators to explore 3D graphics, animations, and model loading while still exposing the generated Three.js source for inspection, learning, or copy-paste into other projects.

## Limitations & Notes

- Execution uses `eval()` of generated code (convenient for live preview but not suitable for untrusted user content in production).
- The runtime expects a global `THREE` (and `THREE.GLTFLoader`) to be present; the sample host loads them from CDN.
- Some GLTF progress/error generator snippets reference variables (`url`, `loader`) that are not in scope in the emitted code — these blocks are best used via the `on_load` container for now.
- The editor is primarily a prototyping / teaching surface; full applications are usually built by exporting the generated code or embedding the library.

For a hands-on introduction, see the [Getting Started](getting-started.md) guide. For the complete list of blocks, explore the toolbox categories directly in the editor or the source in `blockly-threeJS/src/`.
