# Getting Started with SemiBlock Three.js

This guide walks you through opening the editor, building your first live 3D scene, using the three toolbox categories, loading a GLTF model, and saving or exporting your work. Everything described here matches the implementation in `newblock-server/blockly-threeJS`.

## 1. Opening the Editor

The Three.js editor is embedded inside the larger NewBlock / SemiBlock web application (Laravel + Vite frontend). It is served as a pre-built webpack library from:

```
public/blockly-threeJS/build-production/
```

Typical integration points expose a button or menu item labeled "Three.js", "3D Blocks", or similar that loads the Blockly canvas, output pane, and `#threeJSDiv` preview.

Once loaded you will see:

- Top toolbar with **Clear**, **Load** (demo project), and **Save** buttons.
- Left/center: the Blockly workspace (`#blocklyDiv`).
- Right side: 400 px output pane (generated code + console) + 600 px live 3D canvas (`#threeJSDiv`).

The global `quantr` object (the webpack library export) provides the control API used by the host page.

## 2. Your First Scene (Step by Step)

1. **Clear the workspace** (top button) so you start fresh.
2. From the **Scene** category (pink), drag:
   - `scene_create` → emits `const scene = new THREE.Scene()`
   - `camera_create` → set reasonable values (e.g. FOV 70, Aspect 1.5, Near 0.01, Far 10)
   - `set_camera_position` → X:0 Y:0 Z:5
   - `light_create` → choose `AmbientLight()` (or Point/Directional)
   - `set_background_color` → pick a dark gray
3. Still in Scene:
   - `create_renderer` (width 800, height 600) — this is important: it clears `#threeJSDiv` and appends the WebGL canvas.
   - `create_geometry` (choose BoxGeometry) or use the specific `cylinder_create` / `ball_create`
   - `create_mesh_normal_material`
   - `create_mesh` (type the variable names you used, e.g. `geometry` and `material`)
   - `add_mesh_to_scene` (type `mesh`)
4. Add animation:
   - `create_clock`
   - `function_animate` (drop a `rotate_mesh` X:0.01 Y:0.01 Z:0 and a `render_scene_camera` inside its body)
   - `set_animation_loop`
5. Watch the **3D pane** update live as soon as you snap blocks together. The generated code appears in the top half of the output pane; any `console.log` or errors go to the bottom half.

If the preview goes blank, re-emit a `create_renderer` block — it resets the container.

## 3. Using the Javascript Category (Purple)

This category gives you the usual programming building blocks so you can add logic, variables, and loops around your 3D objects:

- `js_code` — paste any raw JavaScript (including Three.js calls). Use the multiline field.
- `declare_const`, `declare_string_variable`, `declare_number_variable`
- `variable_set` / `variable_get`
- `loop_for`, `if_else`
- `string_concat`, `math_add`
- `debug_log` — sends values to the `#output` console pane (very useful while learning)

These blocks generate ordinary JavaScript and can be mixed freely with Scene blocks.

## 4. Loading GLTF Models

The **GLTF Loader** category (blue) provides four blocks that wrap `THREE.GLTFLoader`:

- `gltf_loader_load` — simple fire-and-forget load. The generated code automatically centers the model and adds a couple of lights.
- `gltf_loader_on_load` — the recommended block for most work. It gives you a statement body that runs after the model is loaded and centered. Inside that body you can create a mixer, fetch clips, set up actions, etc.
- `gltf_loader_on_progress` and `gltf_loader_on_error` — containers for the corresponding callbacks (note: current generator snippets for progress/error have a couple of undefined-variable issues in the emitted code; prefer the `on_load` pattern for now).

Example flow for an animated model:

```
gltf_loader_on_load (url = "...RobotExpressive.glb")
    create_mixer (object = "model")
    get_animation_clip (index = 6)
    create_action_from_clip
    set_action_properties (clamp checked, play checked)
```

The `function_animate` block (in Scene) already contains the `if (mixer) mixer.update(dt)` call, so once you have a mixer the animation will play.

## 5. Saving, Loading, and Exporting

- **Local save**: Click the **Save** button or call `quantr.saveWS()`. Data goes to `localStorage` under the key `mainWorkspace` using Blockly's standard serialization format.
- **Load from localStorage**: `quantr.loadWS()` (the host page usually calls this on startup).
- **Programmatic load** (used by the platform): the host can call `quantr.loadWS(base64Data)` where the data is a base64-encoded JSON workspace dump (see the `load()` helper in `src/index.html` for an example).
- **Get the generated code**: `quantr.getCode()` returns the string that `threejsGenerator.workspaceToCode(...)` would produce.
- **Get the raw workspace**: `quantr.getWS()` returns the serialized JSON.
- **Clear**: `quantr.clearWorkspace()` or the top button.
- **Refresh / re-render**: `quantr.refreshScreen()` (forces a re-run of the current code).

The parent Laravel application can also POST the serialized workspace to `/project/*` endpoints for permanent storage (the same pattern used by the flowchart tool).

## 6. Tips & Common Patterns

- Always include a `create_renderer` block if you want to see anything — it is what actually creates and mounts the `<canvas>`.
- The animation system relies on a `create_clock` block somewhere in the workspace (it sets the global `clock` used by `function_animate`).
- For GLTF models with multiple animations, use `get_animation_clip` with different indices and multiple `create_action_from_clip` + `set_action_properties` blocks.
- Use `debug_log` liberally while you are learning; its output appears in the bottom pane without polluting the browser console.
- The `number_input` and `color_picker` blocks are value blocks — wire them into other blocks that accept numeric or color inputs.
- If you need behavior that isn't covered by the existing blocks, drop in a `js_code` block. The generator just emits the text you typed.
- The editor is intentionally a live scratchpad. For a production page you would normally export the generated code (or the workspace JSON) and run it in a normal Three.js application with proper module imports and no `eval`.

## 7. Exploring the Source

All of the behavior above is implemented in the `blockly-threeJS` folder:

- `src/index.js` — Blockly injection, quantr theme, CustomCategory with SVG icons, change listeners, `runCode()`, exports.
- `src/toolbox.js` — exact category structure and block ordering.
- `src/blocks/threejsBlocks.js` — every block definition (JSON + `createBlockDefinitionsFromJsonArray`).
- `src/generators/threejs.js` — `threejsGenerator`, `forBlock` implementations, `myStatementsToCode`, custom `workspaceToCode`.
- `src/index.html` + `index.css` — the standalone host page layout and the demo `load()` project.
- `webpack.config.js` — builds the library into the Laravel public folder.

Open any of those files to see the precise messages, colours, generator strings, and runtime wiring.

You now have enough to create, animate, and load models in the SemiBlock Three.js editor. Happy block-building!