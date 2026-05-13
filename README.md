# 🌿 MintLua API

[![Version](https://img.shields.io/badge/version-1.3.0-green.svg)](https://github.com/OutOfBoundsOffice/SourcePauseTool)
[![Platform](https://img.shields.io/badge/platform-Windows-blue.svg)]()
[![Sandbox](https://img.shields.io/badge/security-sandboxed-orange.svg)]()

**MintLua** is a high-performance, sandboxed Lua scripting engine for `mint`. It allows developers to extend game functionality with custom movement mechanics, advanced visuals, and interactive GUI controls—all without touching C++.

> [!IMPORTANT]
> This documentation is generated from the C++ source code and is 100% accurate to the implementation.

---

## 📑 Table of Contents

- [🚀 Getting Started](#-getting-started)
- [📝 Script Structure](#-script-structure)
- [🔗 Movement Hooks](#-movement-hooks)
- [🎮 Movement API](#-movement-api-mintmovement)
- [⌨️ Input API](#-input-api-mintinput)
- [⚙️ Engine API](#-engine-api-mintengine)
- [🎨 Render API](#-render-api-mintrender)
- [🖼️ GUI API](#-gui-api-mintgui)
- [✨ Animation API](#-animation-api-mintanim)
- [💾 Config API](#-config-api-mintconfig)
- [📁 File IO API](#-file-io-api-safe_io)
- [🔒 Sandbox](#-sandbox)
- [📚 Examples](#-examples)
- [🛠️ Troubleshooting](#-troubleshooting)

---

## 🚀 Getting Started

1. **Install Scripts**: Place your `.lua` files in the `/lua_scripts/` folder within your game directory.
2. **Auto-Load**: Scripts are detected and loaded automatically when the game starts.
3. **Manage**: Open the **Scripts** tab in the `mint` menu to enable, disable, edit, or reload scripts in real-time.
4. **Debug**: View errors in the game console (prefixed with `[Lua]`) or directly within the Script Manager interface.

---

## 📝 Script Structure

Extend the engine by defining these optional global hooks:

```lua
-- Called every tick before the engine processes input (CUserCmd available)
function on_create_move(cmd) end

-- Called every tick BEFORE physics simulation (CMoveData available)
function on_process_movement_pre(player, move) end

-- Called every tick AFTER physics simulation
function on_process_movement_post(player, move) end

-- Called every ImGui frame for rendering (use mint.render here)
function on_render() end

-- Called every frame when the mint menu is open
function on_menu_draw() end

-- Called once when the script is enabled (use mint.gui.register_tab here)
function register() end

-- Edge-triggered ground events
function on_jump() end
function on_land() end
```

> [!NOTE]
> `cmd`, `player`, and `move` are opaque integer handles passed back into the respective `mint.movement.*` API calls.

---

## 🔗 Movement Hooks

| Hook | When called | Handle |
|---|---|---|
| `on_create_move(cmd)` | Every input tick | `cmd` → CUserCmd |
| `on_process_movement_pre(player, move)` | Before engine physics | `move` → CMoveData |
| `on_process_movement_post(player, move)` | After engine physics | `move` → CMoveData |
| `on_render()` | Every ImGui frame | — |
| `on_menu_draw()` | While mint menu is open | — |
| `register()` | Once on enable | — |
| `on_jump()` | Edge: just left ground | — |
| `on_land()` | Edge: just touched ground | — |

---

## 🎮 Movement API (`mint.movement`)

### 🔍 Player State (Read)

```lua
mint.movement.get_velocity()      -- {x, y, z}  AbsVelocity
mint.movement.get_origin()        -- {x, y, z}  AbsOrigin
mint.movement.get_eye_pos()       -- {x, y, z}  EyePosition
mint.movement.is_on_ground()      -- bool        FL_ONGROUND flag
mint.movement.is_ducking()        -- bool        FL_DUCKING flag
mint.movement.get_speed()         -- float       sqrt(vx²+vy²)
mint.movement.get_speed_3d()      -- float       sqrt(vx²+vy²+vz²)
mint.movement.get_z_velocity()    -- float       vz only
mint.movement.get_max_speed()     -- float       m_flMaxspeed
```

### ⚡ Player State (Write)

```lua
mint.movement.set_velocity(x, y, z)   -- overwrite AbsVelocity
mint.movement.add_velocity(x, y, z)   -- impulse (adds to AbsVelocity)
mint.movement.set_z_velocity(z)       -- vertical only
mint.movement.set_on_ground(bool)     -- force FL_ONGROUND flag
mint.movement.set_max_speed(v)        -- overwrite m_flMaxspeed
```

### 🐰 Bhop Control

```lua
mint.movement.set_bhop(bool)   -- enable/disable Lua-controlled autojump
mint.movement.get_bhop()       -- bool
```

### ⌨️ CUserCmd — *Use inside `on_create_move(cmd)`*

```lua
-- View angles
local a = mint.movement.get_view_angles(cmd)   -- {pitch, yaw}
mint.movement.set_view_angles(cmd, pitch, yaw)

-- Forward direction vector from angles
local fwd = mint.movement.get_forward_vector(pitch, yaw)  -- {x, y, z}

-- Move inputs
local f = mint.movement.get_forwardmove(cmd)
mint.movement.set_forwardmove(cmd, 450)
local s = mint.movement.get_sidemove(cmd)
mint.movement.set_sidemove(cmd, -450)
local u = mint.movement.get_upmove(cmd)
mint.movement.set_upmove(cmd, 0)

-- Buttons (string names)
-- Valid names: "jump", "duck", "forward", "back", "left", "right",
--              "attack", "attack2", "reload", "use"
mint.movement.set_button(cmd, "jump", true)
local held = mint.movement.get_button(cmd, "jump")   -- bool

-- Force a single attack click reliably (via GetButtonBits)
mint.movement.force_attack()

-- Raw button bitfield
local bits = mint.movement.get_buttons_raw(cmd)   -- int
mint.movement.set_buttons_raw(cmd, bits)
```

### 🧬 CMoveData — *Use inside `on_process_movement_pre/post`*

```lua
-- Velocity
local v = mint.movement.move_get_velocity(move)        -- {x, y, z}
mint.movement.move_set_velocity(move, vx, vy, vz)
mint.movement.move_add_velocity(move, ax, ay, az)
mint.movement.move_set_z_velocity(move, vz)

-- Origin
local p = mint.movement.move_get_origin(move)          -- {x, y, z}
mint.movement.move_set_origin(move, x, y, z)

-- Move inputs
local f = mint.movement.move_get_forwardmove(move)
mint.movement.move_set_forwardmove(move, v)
local s = mint.movement.move_get_sidemove(move)
mint.movement.move_set_sidemove(move, v)
local u = mint.movement.move_get_upmove(move)
mint.movement.move_set_upmove(move, v)

-- Max speed
local ms = mint.movement.move_get_max_speed(move)
mint.movement.move_set_max_speed(move, v)
local cms = mint.movement.move_get_client_max_speed(move)
mint.movement.move_set_client_max_speed(move, v)

-- Buttons (raw int)
local b = mint.movement.move_get_buttons(move)         -- int
mint.movement.move_set_buttons(move, b)
local ob = mint.movement.move_get_old_buttons(move)    -- int

-- Button helpers
-- move_button_down valid names: "jump","duck","forward","back","left","right",
--                               "moveleft","moveright","attack","attack2","use"
mint.movement.move_button_down(move, "jump")           -- bool: held this tick
-- move_button_pressed valid names: "jump","duck","forward","back",
--                                  "attack","attack2","use"
mint.movement.move_button_pressed(move, "jump")        -- bool: pressed this tick (not last)

-- View angles
local a = mint.movement.move_get_view_angles(move)     -- {pitch, yaw, roll}
mint.movement.move_set_view_angles(move, pitch, yaw, roll)
```

---

## ⌨️ Input API (`mint.input`)

```lua
mint.input.is_key_down(vk)      -- bool: key currently held (uses GetAsyncKeyState)
mint.input.was_key_pressed(vk)  -- bool: key pressed since last call

-- Source Engine ButtonCode support
mint.input.is_button_down(code) -- bool: button code is held (mouse, etc.)
mint.input.get_button_name(code)-- string: get engine name for code
mint.input.lookup_button(name)  -- int: find code by engine name
```

### ⌨️ Key & Button Constants

| Constant Type | Examples |
|---|---|
| **Windows VK** | `VK_SPACE`, `VK_SHIFT`, `VK_CONTROL`, `VK_MENU` (ALT), `VK_TAB`, `VK_ESCAPE`, `VK_F1`..`VK_F12`, `VK_LEFT`..`VK_DOWN` |
| **Alpha Keys** | `KEY_A` through `KEY_Z` |
| **Numeric Keys** | `KEY_0` through `KEY_9` |
| **Mouse/Engine** | `MOUSE_LEFT`, `MOUSE_RIGHT`, `MOUSE_MIDDLE`, `MOUSE_4`, `MOUSE_5`, `MOUSE_WHEEL_UP`, `MOUSE_WHEEL_DOWN` |

---

## ⚙️ Engine API (`mint.engine`)

### 🛠️ ConVar manipulation

```lua
local cvar = mint.engine.get_convar("sv_cheats")
-- Returns: {exists=bool, value=string, float_value=float, int_value=int, default_value=string}

local ok = mint.engine.set_convar("sv_cheats", "1")  -- bool
```

### 🖥️ Engine Commands

```lua
mint.engine.client_cmd("noclip")
```

### 📊 Info Queries

```lua
local idx = mint.engine.get_local_player()   -- int: local player entity index
local dir = mint.engine.get_game_dir()       -- string: game directory path
local t   = mint.engine.get_time()           -- double: real wall-clock time (seconds)
local ht  = mint.engine.get_host_time()      -- double: game host time
local ft  = mint.engine.get_frametime()      -- double: last frame delta (seconds)
local ti  = mint.engine.get_tick_interval()  -- float: always 0.015
local htk = mint.engine.get_host_tick()      -- int: host tick counter
local map = mint.engine.get_map_name()       -- string: e.g. "d1_canals_01" (empty if no map loaded)
local fc  = mint.engine.get_frame_count()   -- int: ImGui frame counter (increments every render frame)

-- Get specific entity info by index
local ent = mint.engine.get_entity_info(index)
-- Returns: {valid=bool, index=int, is_dormant=bool, class_name=string, origin={x,y,z}}

-- Iterate all client entities
mint.engine.for_each_entity(function(ent) ... end)
```

### 📍 Spatial Queries (Ray Tracing)

```lua
-- Entity-aware trace: hits players, NPCs, and props.
local tr = mint.engine.trace_ray(x1,y1,z1, x2,y2,z2)
-- tr.hit, tr.fraction, tr.hit_pos, tr.normal, tr.entity_index

-- World-only trace: passes through all entities.
local tr = mint.engine.trace_line(x1,y1,z1, x2,y2,z2)
```

---

## 🎨 Render API (`mint.render`)

### 🖼️ Layers & Screen
```lua
mint.render.set_layer("background")   -- behind game world
mint.render.set_layer("foreground")   -- above everything
mint.render.set_layer("window")       -- current ImGui window
local s = mint.render.get_screen_size()  -- {width, height}
```

### 📐 2D Primitives
```lua
mint.render.draw_line(x1,y1, x2,y2, r,g,b,a, thickness)
mint.render.draw_rect(x,y,w,h, r,g,b,a, thickness)
mint.render.draw_rect_filled(x,y,w,h, r,g,b,a)
mint.render.draw_rect_rounded(x,y,w,h, r,g,b,a, rounding, thickness)
mint.render.draw_rect_rounded_filled(x,y,w,h, r,g,b,a, rounding)
mint.render.draw_rect_gradient(x,y,w,h, r1,g1,b1,a1, r2,g2,b2,a2, r3,g3,b3,a3, r4,g4,b4,a4)
mint.render.draw_circle(x,y, radius, r,g,b,a, thickness)
mint.render.draw_circle_filled(x,y, radius, r,g,b,a)
mint.render.draw_triangle(x1,y1, x2,y2, x3,y3, r,g,b,a, thickness)
mint.render.draw_triangle_filled(x1,y1, x2,y2, x3,y3, r,g,b,a)
mint.render.draw_quad(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a, thickness)
mint.render.draw_quad_filled(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a)
mint.render.draw_arc_filled(cx,cy, radius, a_start, a_end, r,g,b,a, segments)
mint.render.draw_polyline(points, r,g,b,a, closed, thickness)
mint.render.draw_polygon_filled(points, r,g,b,a)
```

### 📝 Text Rendering
```lua
mint.render.draw_text(x,y, text, r,g,b,a, size)
local m = mint.render.text_size(text, size)
mint.render.draw_text_rotated(cx,cy, text, r,g,b,a, size, angle_rad)
mint.render.draw_text_wave(x,y, text, r,g,b,a, size, amplitude, frequency, time, phase_per_char)
mint.render.draw_text_rainbow_wave(x,y, text, alpha, size, amp, freq, t, phase, h_speed, h_char, s, v)
```

### 🌍 3D / World Space
```lua
local s = mint.render.world_to_screen(x, y, z)  -- {valid, x, y}
mint.render.draw_line_3d(x1,y1,z1, x2,y2,z2, r,g,b,a, thickness)
mint.render.draw_image(tex_id, x,y,w,h, r,g,b,a)
```

---

## 🖼️ GUI API (`mint.gui`)

### 🎛️ Widgets
```lua
mint.gui.checkbox("Label", bool)
mint.gui.slider_float("Label", value, min, max, format)
mint.gui.slider_int("Label", value, min, max, format)
mint.gui.button("Label", w, h)
mint.gui.input_text("Label", string)
mint.gui.combo("Label", index, items_table)
mint.gui.color_edit("Label", {r,g,b,a})
mint.gui.keybind("Label", vk)
```

### 🏗️ Layout & Groups
```lua
mint.gui.separator()
mint.gui.same_line(offset, spacing)
mint.gui.new_line()
mint.gui.spacing()
mint.gui.indent(w)
mint.gui.unindent(w)
mint.gui.text("text")
mint.gui.text_colored(r, g, b, a, "text")
mint.gui.progress_bar(fraction, w, h, overlay)
mint.gui.begin_child("id", w, h, border, flags)
mint.gui.end_child()
mint.gui.begin_group()
mint.gui.end_group()
```

---

## ✨ Animation API (`mint.anim`)

### 📉 Interpolation
```lua
mint.anim.lerp(a, b, t)
mint.anim.clamp(v, lo, hi)
mint.anim.smoothstep(a, b, v)
mint.anim.damp(current, target, rate, dt)
```

### 🌊 Easing Functions
`ease_linear`, `ease_in_quad`, `ease_out_quad`, `ease_in_out_quad`, `ease_in_cubic`, `ease_out_cubic`, `ease_in_out_cubic`, `ease_in_sine`, `ease_out_sine`, `ease_out_elastic`, `ease_out_bounce`, etc.

---

## 💾 Config API (`mint.config`)

```lua
mint.config.save(key, value)
local v = mint.config.load(key, default)
mint.config.save_table(section, table)
local t = mint.config.load_table(section, default_table)
```

---

## 🔒 Sandbox

The following globals are **REMOVED** for security:
`os`, `io`, `package`, `debug`, `dofile`, `loadfile`, `load`, `require`, `getfenv`, `setfenv`, `rawget`, `rawset`, `rawequal`, `collectgarbage`

---

## 📚 Examples

### 🐰 Simple Auto-Bhop
```lua
function on_create_move(cmd)
    if mint.movement.is_on_ground() then
        mint.movement.set_button(cmd, "jump", true)
    end
end
```

---

## 🛠️ Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Script shows error | Syntax or API | Check console for `[Lua]` prefix |
| `save_table` crash | Integer keys | Use string keys only |
| `on_jump` not firing | Method name | Ensure lowercase `on_jump` |

---

<p align="center">
  <i>Developed with ❤️ by the mint team.</i>
</p>
