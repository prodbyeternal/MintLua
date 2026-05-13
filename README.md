# 🌿 MintLua API

[![Version](https://img.shields.io/badge/version-1.4.0-green.svg)](https://github.com/OutOfBoundsOffice/SourcePauseTool)
[![Platform](https://img.shields.io/badge/platform-Source%20Engine%202013-blue.svg)]()
[![Sandbox](https://img.shields.io/badge/security-sandboxed-orange.svg)]()

This is the definitive guide to the **MintLua** scripting system. Every function listed here is cross-referenced directly with the C++ implementation in the `mint` source code.

> [!IMPORTANT]
> This documentation matches the C++ bindings exactly as defined in `movement_api.cpp`, `engine_api.cpp`, `render_api.cpp`, `gui_api.cpp`, `config_api.cpp`, and `animation_api.cpp`.

---

## 📑 Table of Contents

- [🚀 Getting Started](#-getting-started)
- [📝 Script Hooks](#-script-hooks)
- [🎮 Movement API (`mint.movement`)](#-movement-api-mintmovement)
- [⌨️ Input API (`mint.input`)](#-input-api-mintinput)
- [⚙️ Engine API (`mint.engine`)](#-engine-api-mintengine)
- [🎨 Render API (`mint.render`)](#-render-api-mintrender)
- [🖼️ GUI API (`mint.gui`)](#-gui-api-mintgui)
- [✨ Animation API (`mint.anim`)](#-animation-api-mintanim)
- [💾 Config API (`mint.config`)](#-config-api-mintconfig)
- [📁 File IO API (`safe_io`)](#-file-io-api-safe_io)
- [🔒 Sandbox & Security](#-sandbox--security)
- [📚 Examples](#-examples)

---

## 🚀 Getting Started

1. **Path**: Scripts reside in `[GameDir]\lua_scripts\`.
2. **Persistence**: Script enabled/disabled states are saved in `lua_scripts\script_states.json`.
3. **Hot-Reloading**: Mint watches the file system; saving a script in your editor will instantly reload it in-game.
4. **Console**: Use `print()` to output text directly to the Source Engine console (prefixed with `[Lua]`).

---

## 📝 Script Hooks

Define these global functions in your script to respond to engine events. All are optional.

| Function | Handle(s) | Called When |
| :--- | :--- | :--- |
| `on_create_move(cmd)` | `cmd` (uintptr) | Every input tick (15ms). Modify user input. |
| `on_process_movement_pre(player, move)` | `player`, `move` | Before physics simulation starts. |
| `on_process_movement_post(player, move)` | `player`, `move` | After physics simulation completes. |
| `on_render()` | — | Every frame (between ImGui NewFrame and Render). |
| `on_menu_draw()` | — | Every frame while the `mint` menu is visible. |
| `register()` | — | Once when the script is enabled or reloaded. |
| `on_jump()` | — | Edge-triggered when the player leaves the ground. |
| `on_land()` | — | Edge-triggered when the player touches the ground. |

---

## 🎮 Movement API (`mint.movement`)

Direct interface to player physics and the `CUserCmd` / `CMoveData` structures.

### 🔍 Player State (Client-Side)
| Function | Returns | Description |
| :--- | :--- | :--- |
| `get_velocity()` | `table` | `{x, y, z}` absolute velocity from `m_vecAbsVelocity`. |
| `get_origin()` | `table` | `{x, y, z}` absolute position from `m_vecAbsOrigin`. |
| `get_eye_pos()` | `table` | `{x, y, z}` calculated as `origin + view_offset`. |
| `is_on_ground()` | `bool` | Checks `FL_ONGROUND` flag in `m_fFlags`. |
| `is_ducking()` | `bool` | Checks `FL_DUCKING` flag in `m_fFlags`. |
| `get_speed()` | `float` | Horizontal magnitude (`sqrt(x² + y²)`). |
| `get_speed_3d()` | `float` | Full magnitude (`sqrt(x² + y² + z²)`). |
| `get_z_velocity()` | `float` | Current vertical speed. |
| `get_max_speed()` | `float` | Returns `m_flMaxspeed`. |

### ⚡ Player State Manipulation
```lua
mint.movement.set_velocity(x, y, z)   -- Force absolute velocity
mint.movement.add_velocity(x, y, z)   -- Add to current velocity (impulse)
mint.movement.set_z_velocity(z)       -- Set vertical speed only
mint.movement.set_on_ground(bool)     -- Forcefully set/unset FL_ONGROUND
mint.movement.set_max_speed(v)        -- Set the engine max speed cap
```

### ⌨️ User Input (`CUserCmd`)
*Use inside `on_create_move(cmd)`*
```lua
mint.movement.set_button(cmd, name, state) -- name: "jump", "duck", "forward", "back", "left", "right", "attack", "attack2", "reload", "use"
mint.movement.get_button(cmd, name)        -- returns bool
mint.movement.set_view_angles(cmd, p, y)   -- Sets viewangles (pitch, yaw, 0)
mint.movement.get_view_angles(cmd)         -- returns {pitch, yaw}
mint.movement.get_forward_vector(p, y)     -- returns {x, y, z} direction vector

-- Movement inputs (-450 to 450)
mint.movement.set_forwardmove(cmd, v)      -- Forward/Back
mint.movement.set_sidemove(cmd, v)         -- Left/Right
mint.movement.set_upmove(cmd, v)           -- Up/Down (Swimming)

-- Raw access
mint.movement.get_buttons_raw(cmd)         -- returns int (bitfield)
mint.movement.set_buttons_raw(cmd, int)    -- sets int (bitfield)
```

### 🧬 Movement Data (`CMoveData`)
*Use inside `on_process_movement_pre/post(player, move)`*
```lua
mint.movement.move_get_velocity(move)      -- returns {x, y, z}
mint.movement.move_set_velocity(move,x,y,z)
mint.movement.move_add_velocity(move,x,y,z)
mint.movement.move_set_z_velocity(move, z)

mint.movement.move_get_origin(move)        -- returns {x, y, z}
mint.movement.move_set_origin(move,x,y,z)

mint.movement.move_get_buttons(move)       -- returns int (current buttons)
mint.movement.move_set_buttons(move, int)
mint.movement.move_get_old_buttons(move)   -- returns int (previous tick buttons)

mint.movement.move_button_down(move, name)     -- bool: held now
mint.movement.move_button_pressed(move, name)  -- bool: pressed this frame
```

---

## ⌨️ Input API (`mint.input`)

| Function | Returns | Description |
| :--- | :--- | :--- |
| `is_key_down(vk)` | `bool` | True if Windows Virtual Key is held. |
| `was_key_pressed(vk)` | `bool` | True if key was pressed since last check. |
| `is_button_down(code)` | `bool` | True if Source ButtonCode is held (Mouse, etc). |
| `get_button_name(code)`| `string` | Engine name for the button code. |
| `lookup_button(name)`  | `int` | Maps engine name (e.g. "MOUSE1") to code. |

**Constants included**: `VK_SPACE`, `VK_SHIFT`, `VK_CONTROL`, `VK_ESCAPE`, `KEY_A`..`KEY_Z`, `KEY_0`..`KEY_9`, `MOUSE_LEFT`, `MOUSE_RIGHT`, `MOUSE_WHEEL_UP`, etc.

---

## ⚙️ Engine API (`mint.engine`)

### 🛠️ CVars & Commands
```lua
local res = mint.engine.get_convar("name")  -- returns {exists, value, float_value, int_value, default_value}
mint.engine.set_convar("name", "value")     -- returns bool (success)
mint.engine.client_cmd("command")           -- executes string in console
```

### 🌍 World Info
```lua
mint.engine.get_local_player()   -- returns int (entity index)
mint.engine.get_map_name()       -- returns string (e.g. "d1_trainstation_01")
mint.engine.get_time()           -- returns float (real wall-clock time)
mint.engine.get_host_time()      -- returns float (game time)
mint.engine.get_frametime()      -- returns float (last frame delta)
mint.engine.get_tick_interval()  -- returns float (usually 0.015)
mint.engine.get_host_tick()      -- returns int (current tick number)
```

### 🎯 Ray Tracing
```lua
-- Hits world + entities (skips local player)
local tr = mint.engine.trace_ray(x1,y1,z1, x2,y2,z2)
-- returns {hit, fraction, hit_pos={x,y,z}, normal={x,y,z}, entity_index}

-- Hits world ONLY (brushes/displacements)
local tr = mint.engine.trace_line(x1,y1,z1, x2,y2,z2)
```

---

## 🎨 Render API (`mint.render`)

### 🖼️ Layers & Drawing
```lua
mint.render.set_layer("name")   -- "background" (behind), "foreground" (front), "window" (current tab)
local s = mint.render.get_screen_size() -- returns {width, height}

mint.render.draw_line(x1, y1, x2, y2, r, g, b, a, thickness)
mint.render.draw_rect_filled(x, y, w, h, r, g, b, a)
mint.render.draw_circle_filled(x, y, radius, r, g, b, a)
mint.render.draw_text(x, y, "text", r, g, b, a, size)
```

### ✨ Advanced Graphics
```lua
mint.render.draw_rect_gradient(x,y,w,h, c1, c2, c3, c4) -- 4 corner RGBA colors
mint.render.draw_polyline(points, r,g,b,a, closed, thickness) -- points: {x1,y1, x2,y2...}
mint.render.draw_text_rainbow_wave(x, y, text, alpha, size, amp, freq, t, phase, hue_spd, hue_char, s, v)
```

---

## 🖼️ GUI API (`mint.gui`)

All value-based widgets return a table: `{ [1] = changed (bool), [2] = new_value }`.

### 🎛️ Standard Widgets
```lua
mint.gui.checkbox("Label", current_bool)
mint.gui.slider_float("Label", current_val, min, max, "%.2f")
mint.gui.button("Label", width, height) -- returns bool directly
mint.gui.keybind("Label", current_vk)
mint.gui.color_edit("Label", {r, g, b, a})
```

### 🏗️ Layout
```lua
mint.gui.register_tab("Tab Name", function() ... end) -- Adds a custom tab to the sidebar
mint.gui.same_line()
mint.gui.separator()
mint.gui.indent(px)
```

---

## ✨ Animation API (`mint.anim`)

| Function | Returns | Description |
| :--- | :--- | :--- |
| `lerp(a, b, t)` | `float` | Standard linear interpolation. |
| `damp(curr, target, rate, dt)` | `float` | Frame-independent exponential smoothing. |
| `pulse(time, freq)` | `float` | Sinusoidal pulse [0, 1]. |
| `ease_in_out_cubic(t)` | `float` | Smooth acceleration and deceleration. |

**Available Easings**: `linear`, `quad`, `cubic`, `quart`, `expo`, `sine`, `back`, `elastic`, `bounce`.

---

## 💾 Config API (`mint.config`)

Data is saved as JSON in `lua_scripts\config\[script_name].json`.

```lua
mint.config.save("key", value)              -- Save bool, number, or string
local val = mint.config.load("key", default)

mint.config.save_table("name", table)       -- Save a full Lua table (string keys only)
local tbl = mint.config.load_table("name", default_table)
```

---

## 📁 File IO API (`safe_io`)

Restricted, read-only access to files within the `lua_scripts` directory.

```lua
safe_io.read_script("filename.txt")  -- returns string content
safe_io.script_exists("filename.txt") -- returns bool
```

---

## 🔒 Sandbox & Security

MintLua runs in a restricted sandbox. The following globals are **REMOVED**:
- `os`, `io`, `package`, `debug`
- `dofile`, `loadfile`, `load`, `require`
- `rawget`, `rawset`, `rawequal`, `collectgarbage`

**Available Libraries**: `base`, `string`, `table`, `math`.

---

<p align="center">
  <i>Documentation Generated for <b>Mint</b> v1.4.0 — All parameters verified against C++ source.</i>
</p>
