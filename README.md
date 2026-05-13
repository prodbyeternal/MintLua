# 🌿 MintLua API Reference

[![Version](https://img.shields.io/badge/version-1.3.0-green.svg)](https://github.com/OutOfBoundsOffice/SourcePauseTool)
[![Platform](https://img.shields.io/badge/platform-Source%20Engine%202013-blue.svg)]()
[![Sandbox](https://img.shields.io/badge/security-sandboxed-orange.svg)]()

MintLua is a sandboxed Lua scripting system for **Mint** that allows you to create custom movement functions, visuals, and GUI controls. This documentation is synchronized 100% with the C++ implementation.

---

## 📋 Table of Contents

- [🚀 Getting Started](#-getting-started)
- [📝 Script Structure](#-script-structure)
- [📝 Script Hooks](#-script-hooks)
- [🎮 Movement API (`mint.movement`)](#-movement-api-mintmovement)
- [⌨️ Input API (`mint.input`)](#-input-api-mintinput)
- [⚙️ Engine API (`mint.engine`)](#-engine-api-mintengine)
- [🎨 Render API (`mint.render`)](#-render-api-mintrender)
- [🖼️ GUI API (`mint.gui`)](#-gui-api-mintgui)
- [✨ Animation API (`mint.anim`)](#-animation-api-mintanim)
- [💾 Config API (`mint.config`)](#-config-api-mintconfig)
- [📁 File IO API (`safe_io`)](#-file-io-api-safe_io)
- [🔒 Sandbox](#-sandbox)
- [📚 Examples](#-examples)
- [🛠️ Troubleshooting](#-troubleshooting)

---

## 🚀 Getting Started

1. **Path**: Place `.lua` files in the `lua_scripts` folder inside your game directory.
2. **Management**: Use the **Scripts** tab in the Mint menu to enable/disable, edit, and reload scripts.
3. **Hot-Reloading**: Scripts reload instantly upon saving.
4. **Console**: Use `print()` to output to the game console (prefixed with `[Lua]`).

---

## 📝 Script Structure

Define any of these global functions — all are optional:

```lua
-- Called every tick (15ms) to modify input
function on_create_move(cmd) end

-- Called every tick BEFORE physics simulation
function on_process_movement_pre(player, move) end

-- Called every tick AFTER physics simulation
function on_process_movement_post(player, move) end

-- Called every ImGui frame for rendering
function on_render() end

-- Called every frame when the mint menu is open
function on_menu_draw() end

-- Called once when the script is enabled/reloaded
function register() end

-- Edge-triggered events
function on_jump() end
function on_land() end
```

> [!NOTE]
> `cmd`, `player`, and `move` are opaque integer handles passed back into the `mint.movement.*` API calls.

---

## 📝 Script Hooks

| Hook | Handle(s) | Description |
| :--- | :--- | :--- |
| `on_create_move(cmd)` | `cmd` (uintptr) | Every input tick. Use `mint.movement` with `cmd`. |
| `on_process_movement_pre` | `player`, `move` | Before physics. `move` is a `CMoveData` handle. |
| `on_process_movement_post` | `player`, `move` | After physics. |
| `on_render()` | — | Every frame. Use `mint.render`. |
| `on_menu_draw()` | — | Every frame while Mint menu is open. |
| `register()` | — | Called once on enable. Use for `mint.gui.register_tab`. |
| `on_jump()` | — | Edge-triggered when leaving the ground. |
| `on_land()` | — | Edge-triggered when touching the ground. |

---

## 🎮 Movement API (`mint.movement`)

### 🔍 Player State (Read)
| Function | Returns | Description |
| :--- | :--- | :--- |
| `get_velocity()` | `table` | `{x, y, z}` absolute velocity. |
| `get_origin()` | `table` | `{x, y, z}` absolute position. |
| `get_eye_pos()` | `table` | `{x, y, z}` (Origin + View Offset). |
| `is_on_ground()` | `bool` | True if `FL_ONGROUND` is set. |
| `is_ducking()` | `bool` | True if `FL_DUCKING` is set. |
| `get_speed()` | `float` | Horizontal magnitude (`sqrt(vx² + vy²)`). |
| `get_speed_3d()` | `float` | Full magnitude (`sqrt(vx² + vy² + vz²)`). |
| `get_z_velocity()` | `float` | Vertical speed (`vz`) only. |
| `get_max_speed()` | `float` | Current `m_flMaxspeed`. |
| `get_bhop()` | `bool` | Current state of Lua-controlled autojump. |

### ⚡ Player State (Write)
| Function | Args | Description |
| :--- | :--- | :--- |
| `set_velocity(x,y,z)` | `3 floats` | Overwrite `AbsVelocity`. |
| `add_velocity(x,y,z)` | `3 floats` | Impulse (adds to `AbsVelocity`). |
| `set_z_velocity(z)` | `float` | Overwrite vertical speed. |
| `set_on_ground(bool)` | `bool` | Forcefully set/unset `FL_ONGROUND`. |
| `set_max_speed(v)` | `float` | Overwrite `m_flMaxspeed`. |
| `set_bhop(active)` | `bool` | Enable/disable Lua-controlled autojump. |
| `force_attack()` | — | Triggers a single frame of `+attack`. |

### ⌨️ User Command (`CUserCmd`)
*Only valid within `on_create_move(cmd)`*

| Function | Args | Returns |
| :--- | :--- | :--- |
| `set_button(cmd, name, state)` | `uintptr, string, bool` | `name`: "jump", "duck", "forward", "back", "left", "right", "attack", "attack2", "reload", "use" |
| `get_button(cmd, name)` | `uintptr, string` | `bool` |
| `set_view_angles(cmd, p, y)` | `uintptr, float, float` | Sets pitch and yaw. |
| `get_view_angles(cmd)` | `uintptr` | `{pitch, yaw}` |
| `get_forward_vector(p, y)` | `float, float` | `{x,y,z}` unit vector. |
| `get_forwardmove(cmd)` | `uintptr` | `float` (-450 to 450) |
| `set_forwardmove(cmd, v)` | `uintptr, float` | — |
| `get_sidemove(cmd)` | `uintptr` | `float` (-450 to 450) |
| `set_sidemove(cmd, v)` | `uintptr, float` | — |
| `get_upmove(cmd)` | `uintptr` | `float` |
| `set_upmove(cmd, v)` | `uintptr, float` | — |
| `get_buttons_raw(cmd)` | `uintptr` | `int` (bitfield) |
| `set_buttons_raw(cmd, i)` | `uintptr, int` | — |

### 🧬 Move Data (`CMoveData`)
*Only valid within `on_process_movement_pre/post(player, move)`*

| Function | Args | Returns |
| :--- | :--- | :--- |
| `move_get_velocity(move)` | `uintptr` | `{x,y,z}` |
| `move_set_velocity(move, x,y,z)`| `uintptr, f, f, f` | — |
| `move_add_velocity(move, x,y,z)`| `uintptr, f, f, f` | — |
| `move_set_z_velocity(move, z)` | `uintptr, f` | — |
| `move_get_origin(move)` | `uintptr` | `{x,y,z}` |
| `move_set_origin(move, x,y,z)` | `uintptr, f, f, f` | — |
| `move_get_buttons(move)` | `uintptr` | `int` |
| `move_set_buttons(move, i)` | `uintptr, int` | — |
| `move_get_old_buttons(move)` | `uintptr` | `int` |
| `move_get_forwardmove(move)` | `uintptr` | `float` |
| `move_set_forwardmove(move, v)` | `uintptr, f` | — |
| `move_get_sidemove(move)` | `uintptr` | `float` |
| `move_set_sidemove(move, v)` | `uintptr, f` | — |
| `move_get_upmove(move)` | `uintptr` | `float` |
| `move_set_upmove(move, v)` | `uintptr, f` | — |
| `move_get_max_speed(move)` | `uintptr` | `float` |
| `move_set_max_speed(move, v)` | `uintptr, f` | — |
| `move_get_client_max_speed(move)`| `uintptr` | `float` |
| `move_set_client_max_speed(move, v)`| `uintptr, f` | — |
| `move_get_view_angles(move)` | `uintptr` | `{pitch, yaw, roll}` |
| `move_set_view_angles(move,p,y,r)`| `uintptr, f,f,f` | — |
| `move_button_down(move, name)` | `uintptr, string` | `bool` ("jump", "duck", "forward", "back", "left", "right", "moveleft", "moveright", "attack", "attack2", "use") |
| `move_button_pressed(move, name)`| `uintptr, string` | `bool` ("jump", "duck", "forward", "back", "attack", "attack2", "use") |

---

## ⌨️ Input API (`mint.input`)

| Function | Args | Returns | Description |
| :--- | :--- | :--- | :--- |
| `is_key_down(vk)` | `int` | `bool` | Key currently held (GetAsyncKeyState). |
| `was_key_pressed(vk)`| `int` | `bool` | Key pressed since last check. |
| `is_button_down(code)`| `int` | `bool` | Source ButtonCode is held (Mouse, etc). |
| `get_button_name(code)`| `int` | `string` | Engine name for the button code. |
| `lookup_button(name)` | `string`| `int` | Map engine name (e.g. "MOUSE1") to code. |

### 📦 Predefined Constants
`VK_LBUTTON`, `VK_RBUTTON`, `VK_MBUTTON`, `VK_CANCEL`, `VK_BACK`, `VK_TAB`, `VK_CLEAR`, `VK_RETURN`, `VK_SHIFT`, `VK_CONTROL`, `VK_MENU` (Alt), `VK_PAUSE`, `VK_CAPITAL`, `VK_ESCAPE`, `VK_SPACE`, `VK_PRIOR`, `VK_NEXT`, `VK_END`, `VK_HOME`, `VK_LEFT`, `VK_UP`, `VK_RIGHT`, `VK_DOWN`, `VK_INSERT`, `VK_DELETE`, `VK_F1`..`VK_F12`, `KEY_0`..`KEY_9`, `KEY_A`..`KEY_Z`, `MOUSE_LEFT`, `MOUSE_RIGHT`, `MOUSE_MIDDLE`, `MOUSE_4`, `MOUSE_5`, `MOUSE_WHEEL_UP`, `MOUSE_WHEEL_DOWN`.

---

## ⚙️ Engine API (`mint.engine`)

### 🛠️ CVars & Commands
```lua
local cvar = mint.engine.get_convar("sv_cheats")
-- Returns: {exists=bool, value=string, float_value=float, int_value=int, default_value=string}

mint.engine.set_convar("sv_cheats", "1") -- bool (success)
mint.engine.client_cmd("noclip")
```

### 🌍 World Info
| Function | Returns | Description |
| :--- | :--- | :--- |
| `get_local_player()` | `int` | Local player entity index. |
| `get_game_dir()` | `string` | Path to game directory. |
| `get_time()` | `double` | Real wall-clock time (seconds). |
| `get_host_time()` | `double` | Game host time (paused when game is). |
| `get_frametime()` | `double` | Last frame delta (seconds). |
| `get_tick_interval()` | `float` | Fixed at `0.015`. |
| `get_host_tick()` | `int` | Host tick counter. |
| `get_map_name()` | `string` | Current map name (empty if none). |
| `get_frame_count()` | `int` | ImGui frame counter. |

### 🎯 Entity & Tracing
| Function | Args | Returns |
| :--- | :--- | :--- |
| `get_entity_info(idx)` | `int` | `{valid, index, class_name, is_dormant, origin={x,y,z}}` |
| `for_each_entity(fn)` | `func` | Loops all slots; calls `fn(ent_info)`. Return `false` to break early. |
| `trace_ray(x1..z2)` | `6 floats` | `{hit, fraction, hit_pos={x,y,z}, normal={x,y,z}, entity_index}` |
| `trace_line(x1..z2)` | `6 floats` | World-only ray trace (does not hit entities). |

---

## 🎨 Render API (`mint.render`)

All drawing calls use 0–255 integer RGBA. Use inside `on_render()` or `on_menu_draw()`.

### 📐 Layer & Screen
```lua
mint.render.set_layer("background") -- Default (behind game)
mint.render.set_layer("foreground") -- Topmost
mint.render.set_layer("window")     -- Current ImGui child window
local s = mint.render.get_screen_size() -- {width, height}
```

### 📏 2D Primitives
| Function | Args | Description |
| :--- | :--- | :--- |
| `draw_line(x1,y1, x2,y2, r,g,b,a, thick)` | `9 args` | — |
| `draw_rect(x,y,w,h, r,g,b,a, thick)` | `9 args` | Outline. |
| `draw_rect_filled(x,y,w,h, r,g,b,a)` | `8 args` | — |
| `draw_rect_rounded(x,y,w,h, r,g,b,a, rnd, thick)`| `10 args`| — |
| `draw_rect_rounded_filled(x,y,w,h, r,g,b,a, rnd)` | `9 args` | — |
| `draw_rect_gradient(x..h, r1..a1, ... r4..a4)` | `20 args`| 4-corner multi-color. |
| `draw_circle(x,y, rad, r,g,b,a, thick)` | `8 args` | — |
| `draw_circle_filled(x,y, rad, r,g,b,a)` | `7 args` | — |
| `draw_triangle(x1,y1, x2,y2, x3,y3, r,g,b,a, thick)` | `11 args`| Outline. |
| `draw_triangle_filled(x1,y1, x2,y2, x3,y3, r,g,b,a)` | `10 args`| — |
| `draw_quad(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a, thick)` | `13 args`| Outline. |
| `draw_quad_filled(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a)` | `12 args`| — |
| `draw_arc_filled(cx,cy, rad, start, end, r,g,b,a, segs)`| `10 args`| Pie section. |
| `draw_polyline(pts, r,g,b,a, closed, thick)` | `tbl, 6 args`| `pts` is `{x1,y1, x2,y2...}` |
| `draw_polygon_filled(pts, r,g,b,a)` | `tbl, 4 args`| Convex only. |
| `draw_bezier_cubic(x1..y4, r,g,b,a, thick, segs)` | `14 args`| — |
| `draw_bezier_quadratic(x1,y1, x2,y2, x3,y3, r,g,b,a, thick, segs)` | `12 args`| — |
| `push_clip_rect(x,y,w,h, intersect)` | `5 args` | Scissor start. |
| `pop_clip_rect()` | — | Scissor end. |

### 📝 Text & Special
| Function | Args | Returns |
| :--- | :--- | :--- |
| `draw_text(x,y, str, r,g,b,a, size)` | `8 args` | — |
| `draw_text_sized(x,y, str, r,g,b,a, size)` | `8 args` | Same as draw_text. |
| `text_size(str, size)` | `str, f` | `{x, y, w, h}` |
| `draw_text_rotated(cx,cy, str, r,g,b,a, size, rad)`| `9 args` | Clockwise. |
| `draw_text_wave(x,y, str, r,g,b,a, size, amp, freq, t, phase)`| `12 args` | Sine vertical wave. |
| `draw_text_rainbow_wave(...)` | `13 args` | Wave + Hue cycle. |
| `world_to_screen(x,y,z)` | `3 floats` | `{valid, x, y}` |
| `draw_line_3d(x1..z2, r,g,b,a, thick)` | `11 args` | End-to-end W2S projection. |
| `draw_image(tex_id, x,y,w,h, r,g,b,a)` | `uintptr, 8 args`| ImTextureID pointer draw. |

---

## 🖼️ GUI API (`mint.gui`)

**Pattern**: All value-producing widgets take the current value and return `{changed, new_value}`. Store the new value yourself.

### 🎛️ Widgets
```lua
-- Checkbox
local r = mint.gui.checkbox("Label", bool_value) -- r[1]=changed, r[2]=new bool

-- Sliders
local r = mint.gui.slider_float("Label", value, min, max, "%.2f")
local r = mint.gui.slider_int("Label", value, min, max, "%d") -- r[1]=changed, r[2]=new value

-- Button (returns bool directly)
if mint.gui.button("Label", width, height) then ... end

-- Text inputs
local r = mint.gui.input_text("Label", string_value)
local r = mint.gui.input_float("Label", float_value)
local r = mint.gui.input_int("Label", int_value)

-- Combo (0-based index)
local items = {"One", "Two", "Three"}
local r = mint.gui.combo("Label", current_index, items) -- r[2]=new index

-- Color edit (table of {R,G,B,A} 0-255)
local r = mint.gui.color_edit("Label", {255, 0, 0, 255})
-- r[1]=changed, r[2]=R, r[3]=G, r[4]=B, r[5]=A

-- Keybind
local r = mint.gui.keybind("Label", current_vk) -- r[2]=new_vk
```

### 🏗️ Layout
```lua
mint.gui.separator()
mint.gui.same_line(offset, spacing) -- Optional args
mint.gui.new_line()
mint.gui.spacing()
mint.gui.dummy(w, h)
mint.gui.indent(w)
mint.gui.unindent(w)
mint.gui.text("Static Text")
mint.gui.text_colored(255, 255, 0, 255, "Yellow Text")
mint.gui.text_disabled("Gray Text")
mint.gui.progress_bar(fraction, w, h, "Overlay")
```

### 💬 Tooltips
```lua
mint.gui.set_tooltip("Helpful info") -- Sets tooltip for last widget
mint.gui.begin_tooltip()
    mint.gui.text("Custom Tooltip Layout")
mint.gui.end_tooltip()
```

### 🔍 Item State
```lua
mint.gui.is_item_hovered()      -- bool
mint.gui.is_item_clicked(btn)   -- bool (0=LMB, 1=RMB, 2=MMB)
mint.gui.is_item_active()       -- bool
mint.gui.is_window_hovered()    -- bool
mint.gui.is_window_focused()    -- bool
```

### 📦 Child Regions & Groups
```lua
mint.gui.begin_child("id", w, h, border, flags) -- returns bool: visible
mint.gui.end_child()
mint.gui.begin_group()
mint.gui.end_group()
```

### 📍 Cursor & Region
```lua
local p = mint.gui.get_cursor_pos()           -- {x, y} relative to window
mint.gui.set_cursor_pos(x, y)
local p = mint.gui.get_cursor_screen_pos()    -- {x, y} absolute screen
local a = mint.gui.get_content_region_avail() -- {x, y} remaining space
```

### 🖥️ IO Info
```lua
local io = mint.gui.get_io()
-- Returns: {display_w, display_h, mouse_x, mouse_y, framerate, delta_time}
```

### 🪟 Standalone Windows
```lua
local res = mint.gui.begin_window("Title", {
    pos_x = 100, pos_y = 100,
    size_x = 400, size_y = 300,
    set_pos_once = true,
    no_titlebar = false,
    no_background = false,
    closeable = true,
    open = true -- initial state
})
if res.visible then
    mint.gui.text("Window Content")
end
mint.gui.end_window()
```

### 🎟️ Registration
*Use these exclusively inside `register()` hook.*
```lua
-- Adds a custom tab to the Mint sidebar
mint.gui.register_tab("My Tab", function()
    mint.gui.text("Custom Tab Content")
end)

-- Integration (advanced)
mint.gui.register_menu_item(tab_idx, sub_idx, "Label", function() end)
mint.gui.register_bind_indicator("Label", "+command")
```

### 🎨 Styling
```lua
mint.gui.push_style_color(mint.gui.col.WindowBg, r, g, b, a)
mint.gui.pop_style_color(count)

mint.gui.push_style_var(mint.gui.var.WindowRounding, value)
mint.gui.push_style_var_vec2(mint.gui.var.ItemSpacing, x, y)
mint.gui.pop_style_var(count)
```

**mint.gui.col.\*** — ImGuiCol constants (exact names from source):
> Text, TextDisabled, WindowBg, ChildBg, PopupBg, Border, FrameBg, FrameBgHovered, FrameBgActive, TitleBg, TitleBgActive, TitleBgCollapsed, MenuBarBg, ScrollbarBg, ScrollbarGrab, ScrollbarGrabHovered, ScrollbarGrabActive, CheckMark, SliderGrab, SliderGrabActive, Button, ButtonHovered, ButtonActive, Header, HeaderHovered, HeaderActive, Separator, SeparatorHovered, SeparatorActive, ResizeGrip, ResizeGripHovered, ResizeGripActive, PlotLines, PlotLinesHovered, PlotHistogram, PlotHistogramHovered

**mint.gui.var.\*** — ImGuiStyleVar constants (exact names from source):
> Alpha, DisabledAlpha, WindowPadding✳, WindowRounding, WindowBorderSize, WindowMinSize✳, WindowTitleAlign✳, ChildRounding, ChildBorderSize, PopupRounding, PopupBorderSize, FramePadding✳, FrameRounding, FrameBorderSize, ItemSpacing✳, ItemInnerSpacing✳, IndentSpacing, ScrollbarSize, ScrollbarRounding, GrabMinSize, GrabRounding, TabRounding, ButtonTextAlign✳, SelectableTextAlign✳

✳ = Vec2 var, use `push_style_var_vec2`

---

## ✨ Animation API (`mint.anim`)

| Function | Args | Description |
| :--- | :--- | :--- |
| `lerp(a, b, t)` | `3 floats` | Linear interpolate. |
| `clamp(v, lo, hi)` | `3 floats` | Clamp value to range. |
| `smoothstep(a, b, v)` | `3 floats` | Smooth Hermite interpolation. |
| `smootherstep(a, b, v)` | `3 floats` | Ken Perlin's smoother step. |
| `damp(curr, target, rate, dt)`| `4 floats` | Frametime-independent smooth. |
| `pulse(time, freq)` | `2 floats` | Sine pulse [0, 1]. |
| `triangle(time, freq)` | `2 floats` | Triangle wave [0, 1]. |
| `hsv(h, s, v, a)` | `3 floats, 1 int` | `{r, g, b, a}` (0-255). |
| `lerp_color(r1,g1,b1,a1, r2,g2,b2,a2, t)` | `9 args` | `{r, g, b, a}` (0-255). |
| `rolling_dial(curr, target, idx)`| `2 f, 1 i`| For speedometer digit dials. |
| `dial_split(dial_val)` | `float` | `{digit, next, frac}` for rendering. |

**Easings**: `ease_linear`, `ease_in_quad`, `ease_out_quad`, `ease_in_out_quad`, `ease_in_cubic`, `ease_out_cubic`, `ease_in_out_cubic`, `ease_in_quart`, `ease_out_quart`, `ease_in_out_quart`, `ease_in_expo`, `ease_out_expo`, `ease_in_out_expo`, `ease_in_sine`, `ease_out_sine`, `ease_in_out_sine`, `ease_in_back`, `ease_out_back`, `ease_out_elastic`, `ease_out_bounce`.

---

## 💾 Config API (`mint.config`)

Saved to `lua_scripts/config/<filename>.json`.

- `mint.config.save(key, value)`: Save primitive (bool, num, str).
- `local v = mint.config.load(key, default)`: Load value or default.
- `mint.config.save_table(name, tbl)`: Save a full Lua table (**string keys only**).
- `local t = mint.config.load_table(name, def_tbl)`: Load a full table.

---

## 📁 File IO API (`safe_io`)

Restricted read-only access to `lua_scripts/`.

- `safe_io.read_script("file.txt")`: Returns string content or `""`.
- `safe_io.script_exists("file.txt")`: Returns boolean.

---

## 🔒 Sandbox

The following globals are **REMOVED**:
`os`, `io`, `package`, `debug`, `dofile`, `loadfile`, `load`, `require`, `getfenv`, `setfenv`, `rawget`, `rawset`, `rawequal`, `newproxy`, `collectgarbage`, `gcinfo`.

**Available Libraries**: `base`, `string`, `table`, `math`.

---

## 📚 Examples

### Auto Bunnyhop
```lua
local enabled = false
function on_create_move(cmd)
    if enabled and mint.movement.is_on_ground() then
        mint.movement.set_button(cmd, "jump", true)
    end
end
function register()
    mint.gui.register_tab("Auto BHOP", function()
        local r = mint.gui.checkbox("Enable", enabled)
        if r[1] then enabled = r[2] end
    end)
end
```

### Speed HUD
```lua
function on_render()
    local screen = mint.render.get_screen_size()
    local speed = mint.movement.get_speed()
    mint.render.draw_text(screen.width / 2 - 20, screen.height - 80, 
        string.format("%.0f u/s", speed), 255, 255, 255, 255, 16)
end
```

---

## 🛠️ Troubleshooting

| Symptom | Cause | Fix |
| :--- | :--- | :--- |
| Script shows error on load | Syntax error or missing API | Check console for `[Lua]` message |
| `save_table` crashes | Integer keys (array) used | Always use string keys in config tables |
| `trace_line` returns false | Engine unavailable | Only works in-game with a map loaded |
| `register()` tabs missing | Function name mismatch | Ensure `register` is a global function |

---

<p align="center">
  <i>Every function listed is guaranteed to exist in <b>Mint v1.3.0</b>.</i><br>
  <i>Synchronized with C++ Source on 2026-05-13.</i>
</p>
