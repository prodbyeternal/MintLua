# MintLua API Documentation

MintLua is a sandboxed Lua scripting system for mint that allows you to create custom movement functions, visuals, and GUI controls.

> **This documentation is generated from the C++ source code and is 100% accurate to the implementation.**

## 📋 Table of Contents

- [Getting Started](#getting-started)
- [Script Structure](#script-structure)
- [Movement Hooks](#movement-hooks)
- [Movement API](#movement-api-mintmovement)
- [Input API](#input-api-mintinput)
- [Engine API](#engine-api-mintengine)
- [Render API](#render-api-mintrender)
- [GUI API](#gui-api-mintgui)
- [Animation API](#animation-api-mintanim)
- [Config API](#config-api-mintconfig)
- [File IO API](#file-io-api-safe_io)
- [Sandbox](#sandbox)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

## 🚀 Getting Started

1. Place `.lua` files in the `lua_scripts` folder inside your game directory
2. Scripts are loaded automatically on game start
3. Use the **Scripts** tab in the mint menu to enable/disable, edit, and reload scripts
4. Errors are printed to the game console with a `[Lua]` prefix and shown in the Script Manager

## 📝 Script Structure

Declare any of these global functions — all are optional:

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

### Player State (no arguments)

```lua
mint.movement.get_velocity()      -- {x, y, z}  AbsVelocity
mint.movement.get_origin()        -- {x, y, z}  AbsOrigin
mint.movement.is_on_ground()      -- bool        FL_ONGROUND flag
mint.movement.is_ducking()        -- bool        FL_DUCKING flag
mint.movement.get_speed()         -- float       sqrt(vx²+vy²)
mint.movement.get_speed_3d()      -- float       sqrt(vx²+vy²+vz²)
mint.movement.get_z_velocity()    -- float       vz only
mint.movement.get_max_speed()     -- float       m_flMaxspeed
```

### Player State (write)

```lua
mint.movement.set_velocity(x, y, z)   -- overwrite AbsVelocity
mint.movement.add_velocity(x, y, z)   -- impulse (adds to AbsVelocity)
mint.movement.set_z_velocity(z)       -- vertical only
mint.movement.set_on_ground(bool)     -- force FL_ONGROUND flag
mint.movement.set_max_speed(v)        -- overwrite m_flMaxspeed
```

### Bhop

```lua
mint.movement.set_bhop(bool)   -- enable/disable Lua-controlled autojump
mint.movement.get_bhop()       -- bool
```

### CUserCmd — use inside `on_create_move(cmd)`

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

-- Raw button bitfield
local bits = mint.movement.get_buttons_raw(cmd)   -- int
mint.movement.set_buttons_raw(cmd, bits)
```

### CMoveData — use inside `on_process_movement_pre/post(player, move)`

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
```

### Predefined Key Constants

| Constant | Windows VK |
|---|---|
| `KEY_SPACE` | VK_SPACE |
| `KEY_SHIFT` | VK_SHIFT |
| `KEY_CTRL` | VK_CONTROL |
| `KEY_ALT` | VK_MENU (0x12) |
| `KEY_W` `KEY_A` `KEY_S` `KEY_D` | 'W' 'A' 'S' 'D' |
| `KEY_E` `KEY_Q` `KEY_F` `KEY_R` | 'E' 'Q' 'F' 'R' |
| `KEY_X` `KEY_C` `KEY_V` | 'X' 'C' 'V' |
| `KEY_LBUTTON` | VK_LBUTTON |
| `KEY_RBUTTON` | VK_RBUTTON |
| `KEY_MBUTTON` | VK_MBUTTON |

For any other key, pass the Windows VK code as an integer directly (e.g. `0x70` for F1).

---

## ⚙️ Engine API (`mint.engine`)

### ConVar

```lua
local cvar = mint.engine.get_convar("sv_cheats")
-- Returns: {exists=bool, value=string, float_value=float, int_value=int, default_value=string}

local ok = mint.engine.set_convar("sv_cheats", "1")  -- bool
```

### Commands

```lua
mint.engine.client_cmd("noclip")
```

### Info

```lua
local idx = mint.engine.get_local_player()   -- int: local player entity index
local dir = mint.engine.get_game_dir()       -- string: game directory path
local t   = mint.engine.get_time()           -- double: real wall-clock time (seconds)
local ht  = mint.engine.get_host_time()      -- double: game host time
local ft  = mint.engine.get_frametime()      -- double: last frame delta (seconds)
local ti  = mint.engine.get_tick_interval()  -- float: always 0.015
local htk = mint.engine.get_host_tick()      -- int: host tick counter
```

> ⚠️ **Not implemented:** `get_map_name`, `get_frame_count`, `for_each_entity`, `trace_line` — these do **not** exist in the engine API. Calling them will crash the script.

---

## 🎨 Render API (`mint.render`)

All drawing calls use 0–255 integer RGBA. All must be made inside `on_render()` or `on_menu_draw()`.

### Layer

```lua
mint.render.set_layer("background")   -- default, behind game world
mint.render.set_layer("foreground")   -- above everything
mint.render.set_layer("window")       -- current ImGui window draw list
```

### Screen

```lua
local s = mint.render.get_screen_size()  -- {width, height}
```

### 2D Primitives

```lua
mint.render.draw_line(x1,y1, x2,y2, r,g,b,a, thickness)
mint.render.draw_rect(x,y,w,h, r,g,b,a, thickness)
mint.render.draw_rect_filled(x,y,w,h, r,g,b,a)
mint.render.draw_rect_rounded(x,y,w,h, r,g,b,a, rounding, thickness)
mint.render.draw_rect_rounded_filled(x,y,w,h, r,g,b,a, rounding)
mint.render.draw_rect_gradient(x,y,w,h,
    r1,g1,b1,a1,   -- top-left
    r2,g2,b2,a2,   -- top-right
    r3,g3,b3,a3,   -- bottom-right
    r4,g4,b4,a4)   -- bottom-left
mint.render.draw_circle(x,y, radius, r,g,b,a, thickness)
mint.render.draw_circle_filled(x,y, radius, r,g,b,a)
mint.render.draw_triangle(x1,y1, x2,y2, x3,y3, r,g,b,a, thickness)
mint.render.draw_triangle_filled(x1,y1, x2,y2, x3,y3, r,g,b,a)
mint.render.draw_quad(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a, thickness)
mint.render.draw_quad_filled(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a)
mint.render.draw_arc_filled(cx,cy, radius, a_start, a_end, r,g,b,a, segments)  -- angles in radians
mint.render.draw_bezier_cubic(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a, thickness, segments)
mint.render.draw_bezier_quadratic(x1,y1, x2,y2, x3,y3, r,g,b,a, thickness, segments)

-- points: {x1,y1,x2,y2,...} flat  OR  {{x=,y=},{x=,y=},...} table-of-tables
mint.render.draw_polyline(points, r,g,b,a, closed, thickness)
mint.render.draw_polygon_filled(points, r,g,b,a)   -- must be convex
```

### Clipping

```lua
mint.render.push_clip_rect(x,y,w,h, intersect)
mint.render.pop_clip_rect()
```

### Text

```lua
mint.render.draw_text(x,y, text, r,g,b,a, size)
mint.render.draw_text_sized(x,y, text, r,g,b,a, size)   -- same as draw_text

-- Measure text (returns {x, y, w, h}  — w and h are the same as x and y)
local m = mint.render.text_size(text, size)   -- m.w = pixel width, m.h = pixel height

-- Rotated text (angle_rad is clockwise)
mint.render.draw_text_rotated(cx,cy, text, r,g,b,a, size, angle_rad)

-- Vertical sine wave per character
mint.render.draw_text_wave(x,y, text, r,g,b,a, size, amplitude, frequency, time, phase_per_char)

-- Rainbow + wave (hue cycles over time, shifts across characters)
mint.render.draw_text_rainbow_wave(x,y, text, alpha, size,
    amplitude, frequency, time, phase_per_char,
    hue_speed, hue_per_char, saturation, value)
```

### 3D / World

```lua
-- Project world position to screen
local s = mint.render.world_to_screen(x, y, z)
-- Returns: {valid=bool, x=float, y=float}
-- valid is false when the point is behind the camera or engine unavailable

-- Draw line between two world positions (projects both endpoints)
mint.render.draw_line_3d(x1,y1,z1, x2,y2,z2, r,g,b,a, thickness)

-- Draw a texture (requires a valid ImTextureID pointer as integer)
mint.render.draw_image(tex_id, x,y,w,h, r,g,b,a)
```

---

## 🖼️ GUI API (`mint.gui`)

> **Pattern:** All value-producing widgets take the current value and return `{changed, new_value}`. Store the new value yourself.

### Widgets

```lua
-- Checkbox
local r = mint.gui.checkbox("Label", bool_value)
-- r[1]=changed, r[2]=new bool

-- Sliders
local r = mint.gui.slider_float("Label", value, min, max, "%.2f")
local r = mint.gui.slider_int("Label", value, min, max, "%d")
-- r[1]=changed, r[2]=new value

-- Button (returns bool directly, no value-in/out)
if mint.gui.button("Label", width, height) then ... end

-- Text inputs
local r = mint.gui.input_text("Label", string_value)   -- r[1]=changed, r[2]=new string
local r = mint.gui.input_float("Label", float_value)   -- r[1]=changed, r[2]=new float
local r = mint.gui.input_int("Label", int_value)       -- r[1]=changed, r[2]=new int

-- Combo (0-based index)
local items = {"A", "B", "C"}
local r = mint.gui.combo("Label", current_index, items)
-- r[1]=changed, r[2]=new 0-based index

-- Color edit (table of {R,G,B,A} 0-255)
local r = mint.gui.color_edit("Label", {255, 0, 0, 255})
-- r[1]=changed, r[2]=R, r[3]=G, r[4]=B, r[5]=A
```

### Layout

```lua
mint.gui.separator()
mint.gui.same_line(offset, spacing)   -- offset and spacing are optional (pass 0 to use defaults)
mint.gui.new_line()
mint.gui.spacing()
mint.gui.dummy(w, h)
mint.gui.indent(w)
mint.gui.unindent(w)
mint.gui.text("text")
mint.gui.text_colored(r, g, b, a, "text")
mint.gui.progress_bar(fraction, w, h, overlay_string)   -- overlay_string is optional
```

### Tooltip

```lua
mint.gui.set_tooltip("text")    -- sets tooltip for the last widget
mint.gui.begin_tooltip()
    -- widgets here
mint.gui.end_tooltip()
```

### Item State

```lua
mint.gui.is_item_hovered()      -- bool
mint.gui.is_item_clicked(btn)   -- bool (btn: 0=LMB, 1=RMB, 2=MMB)
mint.gui.is_item_active()       -- bool
mint.gui.is_window_hovered()    -- bool
mint.gui.is_window_focused()    -- bool
```

### Child Regions

```lua
mint.gui.begin_child("id", w, h, border, flags)  -- bool: visible
mint.gui.end_child()
mint.gui.begin_group()
mint.gui.end_group()
```

### Cursor

```lua
local p = mint.gui.get_cursor_pos()           -- {x, y}  relative to window
mint.gui.set_cursor_pos(x, y)
local p = mint.gui.get_cursor_screen_pos()    -- {x, y}  absolute screen
local a = mint.gui.get_content_region_avail() -- {x, y}  remaining space
```

### IO

```lua
local io = mint.gui.get_io()
-- {display_w, display_h, mouse_x, mouse_y, framerate, delta_time}
```

### Standalone Windows

```lua
local res = mint.gui.begin_window("Title", {
    pos_x = 80,   pos_y = 80,
    size_x = 320, size_y = 240,
    set_pos_once  = true,    -- only set position on first open
    set_size_once = true,    -- only set size on first open
    closeable     = false,   -- show X button; res.open becomes false when clicked
    open          = true,    -- initial open state when closeable=true
    no_titlebar   = false,
    no_resize     = false,
    no_move       = false,
    no_scrollbar  = false,
    no_collapse   = false,
    no_background = false,
    always_autoresize = false,
    no_inputs     = false,
    no_focus_on_appearing = false,
    bg_alpha      = 0.95,
})
-- res.visible: bool — draw content when true
-- res.open:    bool — false when user clicked X (closeable only)
if res.visible then
    -- your widgets
end
mint.gui.end_window()
```

### Registration (call only inside `register()`)

```lua
-- Adds a new sidebar tab
mint.gui.register_tab("Tab Name", function()
    -- draw callback, called every frame while tab is active
end)

-- Adds item to an existing tab's subtab (stub — currently no-op in implementation)
mint.gui.register_menu_item(tab_index, subtab_index, "Label", function() end)

-- Adds a bind indicator entry (stub — currently no-op in implementation)
mint.gui.register_bind_indicator("Label", "+command")
```

### Styling

```lua
mint.gui.push_style_color(mint.gui.col.WindowBg, r, g, b, a)
mint.gui.pop_style_color(count)

mint.gui.push_style_var(mint.gui.var.WindowRounding, value)
mint.gui.push_style_var_vec2(mint.gui.var.WindowPadding, x, y)  -- for Vec2 vars
mint.gui.pop_style_var(count)
```

**`mint.gui.col.*`** — ImGuiCol constants (exact names from source):
`Text`, `TextDisabled`, `WindowBg`, `ChildBg`, `PopupBg`, `Border`, `FrameBg`, `FrameBgHovered`, `FrameBgActive`, `TitleBg`, `TitleBgActive`, `TitleBgCollapsed`, `MenuBarBg`, `ScrollbarBg`, `ScrollbarGrab`, `ScrollbarGrabHovered`, `ScrollbarGrabActive`, `CheckMark`, `SliderGrab`, `SliderGrabActive`, `Button`, `ButtonHovered`, `ButtonActive`, `Header`, `HeaderHovered`, `HeaderActive`, `Separator`, `SeparatorHovered`, `SeparatorActive`, `ResizeGrip`, `ResizeGripHovered`, `ResizeGripActive`, `PlotLines`, `PlotLinesHovered`, `PlotHistogram`, `PlotHistogramHovered`

**`mint.gui.var.*`** — ImGuiStyleVar constants (exact names from source):
`Alpha`, `DisabledAlpha`, `WindowPadding`✳, `WindowRounding`, `WindowBorderSize`, `WindowMinSize`✳, `WindowTitleAlign`✳, `ChildRounding`, `ChildBorderSize`, `PopupRounding`, `PopupBorderSize`, `FramePadding`✳, `FrameRounding`, `FrameBorderSize`, `ItemSpacing`✳, `ItemInnerSpacing`✳, `IndentSpacing`, `ScrollbarSize`, `ScrollbarRounding`, `GrabMinSize`, `GrabRounding`, `TabRounding`, `ButtonTextAlign`✳, `SelectableTextAlign`✳

✳ = Vec2 var, use `push_style_var_vec2`

---

## ✨ Animation API (`mint.anim`)

### Math

```lua
mint.anim.lerp(a, b, t)                     -- float: linear interpolation
mint.anim.clamp(v, lo, hi)                  -- float: clamped value
mint.anim.smoothstep(a, b, v)               -- float: smooth Hermite interpolation
mint.anim.smootherstep(a, b, v)             -- float: Ken Perlin's smoother step
mint.anim.damp(current, target, rate, dt)   -- float: frametime-independent exponential smooth
                                            --   rate ~8-15 is snappy, higher = faster
```

### Easing (input t in [0,1], output in [0,1])

```lua
mint.anim.ease_linear(t)
mint.anim.ease_in_quad(t)      mint.anim.ease_out_quad(t)      mint.anim.ease_in_out_quad(t)
mint.anim.ease_in_cubic(t)     mint.anim.ease_out_cubic(t)     mint.anim.ease_in_out_cubic(t)
mint.anim.ease_in_quart(t)     mint.anim.ease_out_quart(t)     mint.anim.ease_in_out_quart(t)
mint.anim.ease_in_expo(t)      mint.anim.ease_out_expo(t)      mint.anim.ease_in_out_expo(t)
mint.anim.ease_in_sine(t)      mint.anim.ease_out_sine(t)      mint.anim.ease_in_out_sine(t)
mint.anim.ease_in_back(t)      mint.anim.ease_out_back(t)
mint.anim.ease_out_elastic(t)
mint.anim.ease_out_bounce(t)
```

### Periodic

```lua
mint.anim.pulse(time, frequency)     -- float [0,1]: sinusoidal (0.5 + 0.5*sin)
mint.anim.triangle(time, frequency)  -- float [0,1]: triangle wave
```

### Rolling Digit Dial

```lua
local dial  = mint.anim.rolling_dial(current, target, digit_index)  -- float
local split = mint.anim.dial_split(dial)
-- split.digit  int:   the floor digit (0-9)
-- split.next   int:   next digit (digit+1) % 10
-- split.frac   float: scroll fraction [0,1]
```

### Color

```lua
-- HSV to RGBA. h,s,v in [0,1]. a is integer 0-255.
-- Returns integer-indexed table: {[1]=R, [2]=G, [3]=B, [4]=A}  (0-255)
local c = mint.anim.hsv(h, s, v, a)

-- Lerp between two RGBA colors. All inputs are integers 0-255.
-- Returns integer-indexed table: {[1]=R, [2]=G, [3]=B, [4]=A}
local c = mint.anim.lerp_color(r1,g1,b1,a1, r2,g2,b2,a2, t)
```

---

## 💾 Config API (`mint.config`)

Settings are saved to `lua_scripts/config/<script_filename>.json`. Each script has its own file.

```lua
-- Save / load a single value
mint.config.save(key, value)              -- value: bool, number, or string
local v = mint.config.load(key, default)  -- returns stored value or default

-- Save / load an entire flat table
-- IMPORTANT: table values must be bool, number, or string.
-- Integer-keyed tables (arrays) will CRASH save_table — use string keys only.
mint.config.save_table(section_name, {key1=val1, key2=val2})
local t = mint.config.load_table(section_name, {key1=default1, key2=default2})
```

**Config table key restriction:** `save_table` iterates keys with `.as<std::string>()`. If any key is an integer (e.g. a Lua array `{1,2,3}`), it will throw and crash the script. Always use string keys.

---

## 📁 File IO API (`safe_io`)

A minimal read-only file API is available as the global `safe_io` (not under `mint`). It is restricted to the `lua_scripts` directory only — path traversal attempts return empty/false.

```lua
-- Read a file from the lua_scripts directory (returns string, or "" on failure)
local content = safe_io.read_script("myfile.txt")

-- Check if a file exists in the lua_scripts directory
local exists = safe_io.script_exists("myfile.txt")
```

---

## 🔒 Sandbox

The following Lua globals are **removed** and will be `nil`:

`os`, `io`, `package`, `debug`, `dofile`, `loadfile`, `load`, `require`, `getfenv`, `setfenv`, `rawget`, `rawset`, `rawequal`, `newproxy`, `collectgarbage`, `gcinfo`

Available standard libraries: `base`, `string`, `table`, `math`

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
    local text = string.format("%.0f u/s", speed)
    mint.render.draw_text(screen.width / 2 - 20, screen.height - 80, text, 255, 255, 255, 255, 16)
end
```

### Custom Crosshair with Config
```lua
local cfg = mint.config.load_table("crosshair", {
    enabled = true,
    size = 10,
    r = 0, g = 255, b = 0, a = 255,
})
local enabled = cfg.enabled
local size    = cfg.size
local color   = {cfg.r, cfg.g, cfg.b, cfg.a}

function on_render()
    if not enabled then return end
    local s  = mint.render.get_screen_size()
    local cx = s.width / 2
    local cy = s.height / 2
    local r, g, b, a = color[1], color[2], color[3], color[4]
    mint.render.draw_line(cx - size, cy, cx + size, cy, r, g, b, a, 1)
    mint.render.draw_line(cx, cy - size, cx, cy + size, r, g, b, a, 1)
end

function register()
    mint.gui.register_tab("Crosshair", function()
        local r1 = mint.gui.checkbox("Enabled", enabled)
        if r1[1] then enabled = r1[2] end

        local r2 = mint.gui.slider_int("Size", size, 1, 50, "%d")
        if r2[1] then size = r2[2] end

        local r3 = mint.gui.color_edit("Color", color)
        if r3[1] then
            color = {r3[2], r3[3], r3[4], r3[5]}
            mint.config.save_table("crosshair", {
                enabled=enabled, size=size,
                r=color[1], g=color[2], b=color[3], a=color[4]
            })
        end
    end)
end
```

### Jetpack (uses `on_process_movement_post`)
```lua
local enabled = false
local fuel = 100.0

function on_process_movement_post(player, move)
    if not enabled then return end
    local dt = mint.engine.get_frametime()
    if dt <= 0 then dt = 0.015 end
    if mint.movement.is_on_ground() then
        fuel = math.min(100, fuel + 40 * dt)
        return
    end
    if mint.movement.move_button_down(move, "jump") and fuel > 0 then
        local v = mint.movement.move_get_velocity(move)
        if v.z < 350 then mint.movement.move_set_z_velocity(move, 350) end
        fuel = math.max(0, fuel - 60 * dt)
    end
end

function register()
    mint.gui.register_tab("Jetpack", function()
        local r = mint.gui.checkbox("Enabled", enabled)
        if r[1] then enabled = r[2] end
        mint.gui.progress_bar(fuel / 100, 200, 18, string.format("Fuel %.0f%%", fuel))
    end)
end
```

---

## 🛠️ Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Script shows error on load | Syntax error or missing API | Check game console for `[Lua]` message |
| Tab crashes when clicked | Error inside tab draw callback | Check console — error is shown in tab too |
| `save_table` crashes | Integer-keyed (array) table passed | Use only string keys in the table |
| Calling `for_each_entity` crashes | Not implemented in engine API | Remove it — it does not exist |
| Calling `get_map_name` / `get_frame_count` crashes | Not implemented | Remove them — they do not exist |
| Script stops running after one error | `hasError` flag set, script disabled | Fix the error and reload/re-enable |
| `register()` tabs not appearing | `register()` not called at enable time | Ensure `register` is a global function |
| Config not saving colors | Passing `{r,g,b,a}` array to `save_table` | Flatten to string keys: `r=255, g=0...` |
