# 🌿 MintLua API

[![Version](https://img.shields.io/badge/version-1.3.0-green.svg)](https://github.com/OutOfBoundsOffice/SourcePauseTool)
[![Platform](https://img.shields.io/badge/platform-Windows-blue.svg)]()
[![Sandbox](https://img.shields.io/badge/security-sandboxed-orange.svg)]()

**MintLua** is a high-performance, sandboxed Lua scripting engine for `mint`. It allows developers to extend game functionality with custom movement mechanics, advanced visuals, and interactive GUI controls—all without touching C++.

---

## 📑 Table of Contents

- [🚀 Getting Started](#-getting-started)
- [📝 Script Structure](#-script-structure)
- [🎮 Movement API](#-movement-api-mintmovement)
- [⌨️ Input API](#-input-api-mintinput)
- [⚙️ Engine API](#-engine-api-mintengine)
- [🎨 Render API](#-render-api-mintrender)
- [🖼️ GUI API](#-gui-api-mintgui)
- [✨ Animation API](#-animation-api-mintanim)
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
-- Hook into the input system (per-tick)
function on_create_move(cmd) end

-- Hook into physics simulation (pre-step)
function on_process_movement_pre(player, move) end

-- Hook into physics simulation (post-step)
function on_process_movement_post(player, move) end

-- Draw custom HUD elements (every frame)
function on_render() end

-- Add custom controls to the mint menu
function on_menu_draw() end

-- Initialize your script and register tabs
function register() end

-- Ground state event triggers
function on_jump() end
function on_land() end
```

> [!NOTE]
> `cmd`, `player`, and `move` are opaque integer handles used in the `mint.movement.*` API calls.

---

## 🎮 Movement API (`mint.movement`)

The core of `mint`'s power. Directly manipulate player physics and inputs.

### 🔍 Read State
| Function | Return Type | Description |
| :--- | :--- | :--- |
| `get_velocity()` | `{x, y, z}` | Current Absolute Velocity |
| `get_origin()` | `{x, y, z}` | Current Absolute Origin |
| `is_on_ground()` | `bool` | True if touching the floor |
| `get_speed()` | `float` | Horizontal speed (2D) |
| `get_max_speed()` | `float` | Current engine max speed cap |

### ⚡ Write State
```lua
mint.movement.set_velocity(x, y, z)   -- Overwrite absolute velocity
mint.movement.add_velocity(x, y, z)   -- Apply an impulse/force
mint.movement.set_z_velocity(z)       -- Set vertical speed only
mint.movement.set_on_ground(bool)     -- Force ground/air state
```

### 🖱️ CUserCmd (Input Manipulation)
*Use these inside `on_create_move(cmd)`*
```lua
mint.movement.set_button(cmd, "jump", true)     -- Force a button state
mint.movement.set_view_angles(cmd, pitch, yaw)  -- Change where you're looking
mint.movement.force_attack()                    -- Reliable single-frame attack
```

---

## ⌨️ Input API (`mint.input`)

Detect keyboard and mouse interaction with ease.

```lua
if mint.input.is_key_down(VK_SPACE) then
    print("Space is held!")
end

if mint.input.was_key_pressed(KEY_F) then
    print("F was just pressed.")
end
```

**Supported Constants**:
- `VK_SPACE`, `VK_SHIFT`, `VK_CONTROL`, `VK_ESCAPE`, `VK_F1`..`VK_F12`
- `KEY_A`..`KEY_Z`, `KEY_0`..`KEY_9`
- `MOUSE_LEFT`, `MOUSE_RIGHT`, `MOUSE_WHEEL_UP`, `MOUSE_WHEEL_DOWN`

---

## ⚙️ Engine API (`mint.engine`)

Interact with the Source Engine core.

### 🛠️ Console & Variables
```lua
local cheats = mint.engine.get_convar("sv_cheats")
mint.engine.set_convar("mat_fullbright", "1")
mint.engine.client_cmd("say Hello from Lua!")
```

### 📍 Spatial Queries (Ray Tracing)
```lua
-- Trace a line from point A to B (World + Entities)
local tr = mint.engine.trace_ray(x1,y1,z1, x2,y2,z2)
if tr.hit then
    print("Hit entity at distance: ", tr.fraction)
end
```

---

## 🎨 Render API (`mint.render`)

Draw beautiful overlays, ESPs, and HUD elements. Use 0–255 integer RGBA.

### 🖼️ Drawing
```lua
mint.render.draw_line(x1,y1, x2,y2, 255,176,0,255, 2)
mint.render.draw_rect_filled(x,y,w,h, 0,0,0,150)
mint.render.draw_circle_filled(cx,cy, radius, 255,255,255,200)
```

### 📝 Text Effects
```lua
-- Rainbow waving text for that "gamer" aesthetic
mint.render.draw_text_rainbow_wave(x, y, "MINT", 255, 24, 
    5, 2, mint.engine.get_time(), 0.5, 1, 0.1, 1, 1)
```

---

## 🖼️ GUI API (`mint.gui`)

Build custom menus using ImGui-powered widgets.

```lua
function register()
    mint.gui.register_tab("My Script", function()
        mint.gui.text("Settings")
        local r = mint.gui.checkbox("Enabled", is_enabled)
        if r[1] then is_enabled = r[2] end
        
        if mint.gui.button("Reset Fuel", 100, 20) then fuel = 100 end
    end)
end
```

---

## ✨ Animation API (`mint.anim`)

Make your scripts feel fluid with built-in easing and smoothing.

```lua
-- Frametime-independent exponential smoothing
value = mint.anim.damp(value, target, 10, dt)

-- Easing functions
local alpha = mint.anim.ease_in_out_cubic(progress)
```

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

### 🏎️ Speed Readout
```lua
function on_render()
    local s = mint.render.get_screen_size()
    local vel = mint.movement.get_speed()
    mint.render.draw_text(s.width/2, s.height-100, string.format("%.0f u/s", vel), 255, 255, 255, 255, 20)
end
```

---

## 🛠️ Troubleshooting

| Symptom | Likely Cause | Solution |
| :--- | :--- | :--- |
| **`[Lua] Error`** | Syntax error | Check the game console for line numbers |
| **Tab Crashes** | Logic error in menu | Check console while the tab is open |
| **`save_table` Fail** | Non-string keys | Ensure table keys are strings, not numbers |
| **Trace is `nil`** | Map not loaded | Ray tracing requires an active game session |

---

<p align="center">
  <i>Generated with ❤️ for the mint community.</i>
</p>
