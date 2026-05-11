# MintLua API Documentation

MintLua is a sandboxed Lua scripting system for mint that allows you to create custom movement functions, visuals, and GUI controls.

## 📋 Table of Contents

- [Getting Started](#getting-started)
- [Script Structure](#script-structure)
- [Movement API](#movement-api-mintmovement)
- [Input API](#input-api-mintinput)
- [Engine API](#engine-api-mintengine)
- [Render API](#render-api-mintrender)
- [GUI API](#gui-api-mintgui)
- [Animation API](#animation-api-mintanim)
- [Config API](#config-api-mintconfig)
- [Movement Hooks](#movement-hooks)
- [Examples](#examples)
- [Security Notes](#security-notes)
- [Troubleshooting](#troubleshooting)

## 🚀 Getting Started

1. Create a folder named `lua_scripts` in your game directory
2. Create `.lua` files in that folder
3. Scripts are automatically loaded when the game starts
4. Use the Scripts tab in the mint menu to manage, edit, and reload scripts

## 📝 Script Structure

```lua
-- Called every frame before movement is processed
function on_create_move(cmd)
    -- Your movement code here
end

-- Called every frame for rendering
function on_render()
    -- Your rendering code here
end

-- Called when the menu is being drawn
function on_menu_draw()
    -- Your GUI code here
end

-- Called once when the script is loaded
function register()
    -- Register custom tabs, menu items, etc.
end
```

## 🎮 Movement API (`mint.movement`)

### Player State

#### Get Velocity
```lua
local vel = mint.movement.get_velocity()
-- Returns: {x, y, z}
print("Velocity X:", vel.x)
print("Velocity Y:", vel.y)
print("Velocity Z:", vel.z)
```

#### Get Origin
```lua
local pos = mint.movement.get_origin()
-- Returns: {x, y, z}
print("Position X:", pos.x)
print("Position Y:", pos.y)
print("Position Z:", pos.z)
```

#### Check Ground State
```lua
local on_ground = mint.movement.is_on_ground()
-- Returns: boolean
if on_ground then
    print("Player is on ground")
end
```

#### Check Ducking State
```lua
local ducking = mint.movement.is_ducking()
-- Returns: boolean
if ducking then
    print("Player is ducking")
end
```

### Speed Calculations

#### Get 2D Speed (horizontal)
```lua
local speed = mint.movement.get_speed()
-- Returns: horizontal speed (x/y plane only)
```

#### Get 3D Speed (total)
```lua
local speed = mint.movement.get_speed_3d()
-- Returns: total speed including vertical
```

### Button Control

#### Set Button State
```lua
-- Available buttons: "jump", "duck", "forward", "back", "left", "right", "attack", "attack2", "reload", "use"
mint.movement.set_button(cmd, "jump", true)  -- Press jump
mint.movement.set_button(cmd, "duck", false) -- Release duck
```

#### Get Button State
```lua
local jumping = mint.movement.get_button(cmd, "jump")
-- Returns: boolean
```

### View Control

#### Set View Angles
```lua
mint.movement.set_view_angles(cmd, pitch, yaw)
-- pitch: vertical angle (-89 to 89)
-- yaw: horizontal angle (0 to 360)
```

#### Get View Angles
```lua
local angles = mint.movement.get_view_angles(cmd)
-- Returns: {pitch, yaw}
print("Pitch:", angles.pitch)
print("Yaw:", angles.yaw)
```

#### Get Forward Vector
```lua
local fwd = mint.movement.get_forward_vector(pitch, yaw)
-- Returns: {x, y, z} - normalized direction vector
```

### Advanced Player State Manipulation

These write directly to the player entity. Use them to implement custom mechanics like jetpacks, fake jumps, gravity flips, etc.

```lua
-- Velocity (player AbsVelocity)
mint.movement.set_velocity(x, y, z)        -- overwrite
mint.movement.add_velocity(x, y, z)        -- impulse
mint.movement.set_z_velocity(z)            -- vertical only
local vz = mint.movement.get_z_velocity()

-- Force grounded state
mint.movement.set_on_ground(true|false)

-- Max speed
mint.movement.set_max_speed(450)
local s = mint.movement.get_max_speed()
```

### CUserCmd Manipulation

Used in `on_create_move`:

```lua
mint.movement.set_forwardmove(cmd, 450)
mint.movement.set_sidemove(cmd, -450)
mint.movement.set_upmove(cmd, 0)
local f = mint.movement.get_forwardmove(cmd)

-- Raw button bits for bitfield operations
local bits = mint.movement.get_buttons_raw(cmd)
mint.movement.set_buttons_raw(cmd, bits)
```

### MoveData Manipulation

Used in `on_process_movement_pre/post`. These work on the physics simulation level — your changes affect the engine's movement calculation.

```lua
-- Velocity inside ProcessMovement
local v = mint.movement.move_get_velocity(move)   -- {x, y, z}
mint.movement.move_set_velocity(move, vx, vy, vz)
mint.movement.move_add_velocity(move, ax, ay, az)
mint.movement.move_set_z_velocity(move, vz)

-- Origin
local p = mint.movement.move_get_origin(move)     -- {x, y, z}
mint.movement.move_set_origin(move, x, y, z)

-- Move inputs
mint.movement.move_set_forwardmove(move, 450)
mint.movement.move_set_sidemove(move, 0)
mint.movement.move_set_upmove(move, 0)

-- Buttons inside the move
mint.movement.move_get_buttons(move)              -- int
mint.movement.move_set_buttons(move, b)
mint.movement.move_get_old_buttons(move)
mint.movement.move_button_down(move, "jump")     -- bool
mint.movement.move_button_pressed(move, "jump")  -- pressed this tick

-- View angles
local a = mint.movement.move_get_view_angles(move)  -- {pitch, yaw, roll}
mint.movement.move_set_view_angles(move, p, y, r)

-- Max speed
mint.movement.move_set_max_speed(move, 450)
mint.movement.move_set_client_max_speed(move, 450)
```

### Bunny Hop Control

```lua
mint.movement.set_bhop(true)   -- enables CheckJumpButton override
mint.movement.set_bhop(false)
local active = mint.movement.get_bhop()
```

## ⌨️ Input API (`mint.input`)

Direct virtual-key-code polling for hotkeys and gestures.

```lua
if mint.input.is_key_down(mint.input.KEY_SPACE) then
    -- Space is currently held
end

-- Was the key pressed since the last query?
if mint.input.was_key_pressed(mint.input.KEY_F) then
    -- F was just pressed
end
```

### Predefined Constants

- `KEY_SPACE`, `KEY_SHIFT`, `KEY_CTRL`, `KEY_ALT`
- `KEY_W`, `KEY_A`, `KEY_S`, `KEY_D`, `KEY_E`, `KEY_Q`, `KEY_F`, `KEY_R`, `KEY_X`, `KEY_C`, `KEY_V`
- `KEY_LBUTTON`, `KEY_RBUTTON`, `KEY_MBUTTON`

### Custom Key Codes

For any other key, pass the Windows VK code directly:

```lua
-- F1 key
if mint.input.is_key_down(0x70) then
    -- ...
end
```

## ⚙️ Engine API (`mint.engine`)

The `mint.engine` table provides access to engine-related information and functionality.

### ConVar Management

#### Get ConVar
```lua
local cvar = mint.engine.get_convar("sv_cheats")
-- Returns: {exists, value, float_value, int_value, default_value}
if cvar.exists then
    print("Value:", cvar.value)
    print("Float value:", cvar.float_value)
end
```

#### Set ConVar
```lua
local success = mint.engine.set_convar("sv_cheats", "1")
-- Returns: boolean
```

### Command Execution

```lua
mint.engine.client_cmd("say Hello World")
```

### Player Information

#### Get Local Player Index
```lua
local player_index = mint.engine.get_local_player()
-- Returns: player index or -1 if not found
```

### Entity Iteration

```lua
mint.engine.for_each_entity(function(entity)
    print("Entity:", entity.index)
    print("Class:", entity.class_name)
    print("Position:", entity.origin.x, entity.origin.y, entity.origin.z)
    print("Health:", entity.health)
    print("Alive:", entity.is_alive)
end)
```

### Ray Tracing

```lua
local result = mint.engine.trace_line(x1, y1, z1, x2, y2, z2)
-- Returns: {hit, fraction, hit_pos, hit_entity}
-- hit: boolean
-- fraction: 0.0 to 1.0 (how far the trace went)
-- hit_pos: {x, y, z}
-- hit_entity: entity index or -1

if result.hit then
    print("Hit at:", result.hit_pos.x, result.hit_pos.y, result.hit_pos.z)
    print("Hit entity:", result.hit_entity)
end
```

### Game Information

#### Get Map Name
```lua
local map = mint.engine.get_map_name()
-- Returns: map name string
```

#### Get Game Directory
```lua
local gamedir = mint.engine.get_game_dir()
-- Returns: game directory path
```

#### Get Current Time
```lua
local time = mint.engine.get_time()
-- Returns: current game time in seconds
```

#### Get Frame Count
```lua
local frame = mint.engine.get_frame_count()
-- Returns: current frame number
```

#### Get Tick Information
```lua
local tick_interval = mint.engine.get_tick_interval()  -- Returns: 0.015
local host_tick = mint.engine.get_host_tick()          -- Returns: host tick count
local frametime = mint.engine.get_frametime()          -- Returns: frametime
local host_time = mint.engine.get_host_time()           -- Returns: host time
```

## 🎨 Render API (`mint.render`)

### Basic Drawing Functions

#### Draw 2D Line
```lua
mint.render.draw_line(x1, y1, x2, y2, r, g, b, a, thickness)
-- x1, y1: start position
-- x2, y2: end position
-- r, g, b, a: color (0-255)
-- thickness: line thickness in pixels
```

#### Draw 2D Rectangle
```lua
mint.render.draw_rect(x, y, w, h, r, g, b, a, thickness)
-- x, y: position
-- w, h: width and height
-- thickness: border thickness
```

#### Draw Filled Rectangle
```lua
mint.render.draw_rect_filled(x, y, w, h, r, g, b, a)
```

#### Draw Circle
```lua
mint.render.draw_circle(x, y, radius, r, g, b, a, thickness)
```

#### Draw Filled Circle
```lua
mint.render.draw_circle_filled(x, y, radius, r, g, b, a)
```

### Advanced Primitives

- `draw_triangle(x1,y1,x2,y2,x3,y3,r,g,b,a,thickness)`
- `draw_triangle_filled(x1,y1,x2,y2,x3,y3,r,g,b,a)`
- `draw_quad(x1,y1,..x4,y4,r,g,b,a,thickness)` / `draw_quad_filled(...)`
- `draw_rect_rounded(x,y,w,h,r,g,b,a,rounding,thickness)`
- `draw_rect_rounded_filled(x,y,w,h,r,g,b,a,rounding)`
- `draw_rect_gradient(x,y,w,h, r1,g1,b1,a1, r2,g2,b2,a2, r3,g3,b3,a3, r4,g4,b4,a4)` (corners: TL, TR, BR, BL)
- `draw_bezier_cubic(x1,y1,x2,y2,x3,y3,x4,y4,r,g,b,a,thickness,segments)`
- `draw_bezier_quadratic(x1,y1,x2,y2,x3,y3,r,g,b,a,thickness,segments)`
- `draw_polyline(points, r,g,b,a, closed, thickness)` - `points` may be `{x1,y1,x2,y2,...}` or `{{x=,y=}, ...}`
- `draw_polygon_filled(points, r,g,b,a)` (convex)
- `draw_arc_filled(cx,cy,radius,a_start,a_end,r,g,b,a,segments)` (angles in radians)

### Text Rendering

#### Draw Text
```lua
mint.render.draw_text(x, y, "Hello World", 255, 255, 255, 255, 16)
-- x, y: position
-- text: string to draw
-- r, g, b, a: color (0-255)
-- size: font size
```

#### Advanced Text Functions

- `text_size(text, size)` -> `{x,y,w,h}`
- `draw_text_sized(x,y,text,r,g,b,a,size)`
- `draw_text_rotated(cx,cy,text,r,g,b,a,size,angle_rad)`
- `draw_text_wave(x,y,text, r,g,b,a, size, amplitude, frequency, time, phase_per_char)` - per-character vertical sine wave
- `draw_text_rainbow_wave(x,y,text, alpha, size, amplitude, frequency, time, phase_per_char, hue_speed, hue_per_char, saturation, value)` - wave plus per-character HSV cycling

### 3D Rendering

#### World to Screen Conversion
```lua
local screen = mint.render.world_to_screen(x, y, z)
-- Returns: {valid, x, y}
-- valid: boolean (true if point is in front of camera)
-- x, y: screen coordinates

if screen.valid then
    mint.render.draw_text(screen.x, screen.y, "Entity Here", 255, 0, 0, 255, 14)
end
```

#### Draw 3D Line
```lua
mint.render.draw_line_3d(x1, y1, z1, x2, y2, z2, r, g, b, a, thickness)
-- Draws a line in 3D world space
```

### Layer Selection

```lua
mint.render.set_layer(layer)
-- layer: "background" (default), "foreground" (drawn on top of everything), or "window" (current ImGui window's drawlist)
```

### Clipping

```lua
mint.render.push_clip_rect(x,y,w,h,intersect)
mint.render.pop_clip_rect()
```

### Screen Information

```lua
local screen = mint.render.get_screen_size()
-- Returns: {width, height}
print("Screen width:", screen.width)
print("Screen height:", screen.height)
```

## 🖼️ GUI API (`mint.gui`)

> **Important:** GUI controls use a **value-in / value-out** pattern. They take the current value as input and return a table `{changed, new_value}`. You must store the returned value yourself.

### Basic Widgets

#### Checkbox
```lua
local enabled = false

-- Returns {changed, new_value}
local result = mint.gui.checkbox("Enable Feature", enabled)
if result[1] then          -- result[1] = changed
    enabled = result[2]    -- result[2] = new value
end
```

#### Slider Float
```lua
local value = 1.5

-- Returns {changed, new_value}
local result = mint.gui.slider_float("Value", value, 0.0, 10.0, "%.2f")
if result[1] then
    value = result[2]
end
```

#### Slider Int
```lua
local value = 5

-- Returns {changed, new_value}
local result = mint.gui.slider_int("Count", value, 0, 100, "%d")
if result[1] then
    value = result[2]
end
```

#### Button
```lua
-- Button just returns a boolean (no value to track)
if mint.gui.button("Click Me", 100, 30) then
    print("Button clicked!")
end
```

#### Input Text
```lua
local text = "Hello"

-- Returns {changed, new_text}
local result = mint.gui.input_text("Text", text)
if result[1] then
    text = result[2]
end
```

#### Input Float
```lua
local value = 1.0

-- Returns {changed, new_value}
local result = mint.gui.input_float("Float", value)
if result[1] then
    value = result[2]
end
```

#### Input Int
```lua
local value = 42

-- Returns {changed, new_value}
local result = mint.gui.input_int("Integer", value)
if result[1] then
    value = result[2]
end
```

#### Combo Box
```lua
local current = 0  -- 0-based index
local items = {"Option 1", "Option 2", "Option 3"}

-- Returns {changed, new_index}
local result = mint.gui.combo("Select", current, items)
if result[1] then
    current = result[2]
    print("Selected:", items[current + 1])  -- Lua tables are 1-indexed
end
```

#### Color Edit
```lua
local color = {255, 0, 0, 255}  -- {R, G, B, A}

-- Returns {changed, R, G, B, A}
local result = mint.gui.color_edit("Color", color)
if result[1] then
    color = {result[2], result[3], result[4], result[5]}
end
```

### Layout Helpers

```lua
mint.gui.separator()
mint.gui.same_line()
mint.gui.new_line()
mint.gui.text("Some text")
mint.gui.text_colored(255, 0, 0, 255, "Red text")
```

### Advanced GUI Features

#### Standalone Windows

```lua
local res = mint.gui.begin_window("My Window", {
    size_x = 320, size_y = 240,
    pos_x = 80, pos_y = 80,
    closeable = true,
    no_titlebar = false,
    no_resize   = false,
    no_move     = false,
    no_scrollbar = false,
    no_collapse  = false,
    no_background = false,
    always_autoresize = false,
    no_inputs = false,
    no_focus_on_appearing = false,
    bg_alpha = 0.95,
    set_pos_once  = true,
    set_size_once = true,
    open = true,
})
if res.visible then
    -- widgets here
end
mint.gui.end_window()
-- res.open will be false if the user clicked the X (when `closeable=true`)
```

#### Layout & State Helpers

- `begin_group()` / `end_group()`
- `indent(w)` / `unindent(w)`
- `spacing()` / `dummy(w,h)`
- `set_tooltip(text)` / `begin_tooltip()` / `end_tooltip()`
- `is_item_hovered()` / `is_item_clicked(button)` / `is_item_active()`
- `is_window_hovered()` / `is_window_focused()`
- `progress_bar(fraction, w, h, overlay_text?)`
- `get_io()` -> `{display_w, display_h, mouse_x, mouse_y, framerate, delta_time}`

#### Custom Styling

```lua
mint.gui.push_style_color(mint.gui.col.WindowBg, 14,14,18,235)
mint.gui.push_style_var(mint.gui.var.WindowRounding, 10)
mint.gui.push_style_var_vec2(mint.gui.var.WindowPadding, 14, 14)
-- ... your widgets ...
mint.gui.pop_style_var(2)
mint.gui.pop_style_color(1)
```

**Color Constants:** `mint.gui.col.*` mirrors `ImGuiCol_*` (Text, WindowBg, ChildBg, Border, FrameBg, Button, ButtonHovered, ButtonActive, Header, Separator, ResizeGrip, ScrollbarBg, CheckMark, SliderGrab, PlotLines, PlotHistogram, ...).

**Style Constants:** `mint.gui.var.*` mirrors `ImGuiStyleVar_*` (Alpha, WindowPadding, WindowRounding, WindowBorderSize, FramePadding, FrameRounding, ItemSpacing, ItemInnerSpacing, IndentSpacing, ScrollbarSize, ScrollbarRounding, GrabMinSize, GrabRounding, TabRounding, ButtonTextAlign, ...). Use `push_style_var_vec2` for any value that is a `Vec2` (WindowPadding, FramePadding, ItemSpacing, etc.).

## ✨ Animation API (`mint.anim`)

Helper math + tweening primitives for smooth, frame-rate independent animations.

### Math Helpers

```lua
mint.anim.lerp(a, b, t)              -- Linear interpolation
mint.anim.clamp(v, lo, hi)           -- Clamp value to range
mint.anim.smoothstep(a, b, v)       -- Smooth step interpolation
mint.anim.smootherstep(a, b, v)     -- Smoother step interpolation
mint.anim.damp(current, target, rate, dt)  -- Frametime-independent exponential smoothing (rate ~ 8-15 is snappy)
```

### Easing Functions

Input/output in `[0,1]`:

- `ease_linear`
- `ease_in_quad`, `ease_out_quad`, `ease_in_out_quad`
- `ease_in_cubic`, `ease_out_cubic`, `ease_in_out_cubic`
- `ease_in_quart`, `ease_out_quart`, `ease_in_out_quart`
- `ease_in_expo`, `ease_out_expo`, `ease_in_out_expo`
- `ease_in_sine`, `ease_out_sine`, `ease_in_out_sine`
- `ease_in_back`, `ease_out_back`
- `ease_out_elastic`, `ease_out_bounce`

### Periodic Helpers

```lua
mint.anim.pulse(time, frequency)      -- 0..1 sinusoid
mint.anim.triangle(time, frequency)   -- 0..1 triangle wave
```

### Rolling Digit Dial

For odometer-style number tickers (same effect used on velocity graph's takeoff readout):

```lua
local dial   = mint.anim.rolling_dial(current_value, target_value, digit_index)
local split  = mint.anim.dial_split(dial) -- {digit, next, frac}
-- draw `split.digit` shifted up by `split.frac * size`,
-- and `split.next` shifted up by `(1 - split.frac) * size`,
-- both clipped to a single-digit-wide rect.
```

### Color Helpers

```lua
mint.anim.hsv(h, s, v, a)                  -- {r,g,b,a} (h,s,v in 0..1, a in 0..255)
mint.anim.lerp_color(r1,g1,b1,a1, r2,g2,b2,a2, t)  -- {r,g,b,a}
```

## 💾 Config API (`mint.config`)

Persistent storage for script-specific settings.

### Functions

#### Save Single Value
```lua
mint.config.save(key, value)
-- key (string): The configuration key
-- value (bool, number, string, or table): The value to save
```

#### Load Single Value
```lua
local value = mint.config.load(key, default_value)
-- key (string): The configuration key
-- default_value: The default value to return if the key doesn't exist
-- Returns: The stored value or the default
```

#### Save Table
```lua
mint.config.save_table(table_name, table)
-- table_name (string): The name of the table section
-- table: The table to save (supports bool, number, string values)
```

#### Load Table
```lua
local table = mint.config.load_table(table_name, default_table)
-- table_name (string): The name of the table section
-- default_table: The default table to return if the section doesn't exist
-- Returns: The stored table or the default
```

### Config File Location

Config files are stored in `lua_scripts/config/<script_name>.json`. Each script has its own config file, so settings don't conflict between scripts.

### Example

```lua
-- Load settings on script startup
local settings = mint.config.load_table("my_settings", {
    enabled = false,
    speed = 100.0,
    name = "default"
})

local enabled = settings.enabled
local speed = settings.speed
local name = settings.name

-- Save settings when changed
function register()
    mint.gui.register_tab("My Script", function()
        local r = mint.gui.checkbox("Enabled", enabled)
        if r[1] then
            enabled = r[2]
            mint.config.save_table("my_settings", {
                enabled = enabled,
                speed = speed,
                name = name
            })
        end
    end)
end
```

## 🔗 Movement Hooks

In addition to `on_create_move(cmd)`, you can declare these hooks:

```lua
-- Called every tick BEFORE the engine simulates movement.
-- Use this to override velocity, modify buttons in the move data, etc.
function on_process_movement_pre(player, move) end

-- Called every tick AFTER physics. Use this to read final state or to
-- apply custom forces (jetpack-style).
function on_process_movement_post(player, move) end

-- Edge-triggered when leaving / touching ground.
function on_jump() end
function on_land() end
```

Both `player` and `move` are opaque pointer handles you pass back into the `mint.movement.move_*` API.


### Registration Functions

#### Register Custom Tab
```lua
function register()
    mint.gui.register_tab("My Custom Tab", function()
        mint.gui.text("This is my custom tab!")
        if mint.gui.button("Click Me", 100, 30) then
            print("Clicked!")
        end
    end)
end
```

#### Register Menu Item
```lua
function register()
    -- Add to existing tab (0=Movement, 1=Visuals, 2=Cheats, 3=Settings, 4=Sounds, 5=Recorder, 7=Scripts)
    mint.gui.register_menu_item(0, 0, "My Feature", function()
        mint.gui.text("This appears in Movement > PHYSICS")
    end)
end
```

#### Register Bind Indicator
```lua
function register()
    mint.gui.register_bind_indicator("My Feature", "+my_command")
    -- Adds a bind button in the Movement Assists panel
end
```


## 📚 Examples

### Example 1: Auto Bunnyhop
```lua
local auto_bhop_enabled = false

function on_create_move(cmd)
    if auto_bhop_enabled and mint.movement.is_on_ground() then
        mint.movement.set_button(cmd, "jump", true)
    else
        mint.movement.set_button(cmd, "jump", false)
    end
end

function register()
    mint.gui.register_tab("Auto BHOP", function()
        local result = mint.gui.checkbox("Enable Auto Bhop", auto_bhop_enabled)
        if result[1] then
            auto_bhop_enabled = result[2]
        end
    end)
end
```

### Example 2: Speed Indicator
```lua
function on_render()
    local screen = mint.render.get_screen_size()
    local speed = mint.movement.get_speed()
    
    local text = string.format("Speed: %.1f", speed)
    mint.render.draw_text(10, 10, text, 255, 255, 255, 255, 16)
end
```

### Example 3: Auto-Duck on Fast Fall
```lua
local auto_duck_threshold = -300

function on_create_move(cmd)
    local vel = mint.movement.get_velocity()
    if vel.z < auto_duck_threshold then
        mint.movement.set_button(cmd, "duck", true)
    end
end

function register()
    mint.gui.register_tab("Auto Duck", function()
        local result = mint.gui.slider_float("Fall Threshold", auto_duck_threshold, -1000, 0, "%.0f")
        if result[1] then
            auto_duck_threshold = result[2]
        end
    end)
end
```

### Example 4: Custom Crosshair
```lua
local show_crosshair = true
local crosshair_size = 10
local crosshair_color = {0, 255, 0, 255}

function on_render()
    if not show_crosshair then
        return
    end
    
    local screen = mint.render.get_screen_size()
    local cx = screen.width / 2
    local cy = screen.height / 2
    
    local r, g, b, a = crosshair_color[1], crosshair_color[2], crosshair_color[3], crosshair_color[4]
    
    mint.render.draw_line(cx - crosshair_size, cy, cx + crosshair_size, cy, r, g, b, a, 1)
    mint.render.draw_line(cx, cy - crosshair_size, cx, cy + crosshair_size, r, g, b, a, 1)
end

function register()
    mint.gui.register_tab("Custom Crosshair", function()
        local r1 = mint.gui.checkbox("Show Crosshair", show_crosshair)
        if r1[1] then show_crosshair = r1[2] end

        local r2 = mint.gui.slider_int("Size", crosshair_size, 5, 50, "%d")
        if r2[1] then crosshair_size = r2[2] end

        local r3 = mint.gui.color_edit("Color", crosshair_color)
        if r3[1] then
            crosshair_color = {r3[2], r3[3], r3[4], r3[5]}
        end
    end)
end
```

### Example 5: Entity ESP
```lua
local show_esp = true
local max_distance = 1000

function on_render()
    if not show_esp then
        return
    end
    
    mint.engine.for_each_entity(function(entity)
        if not entity.is_alive then
            return
        end
        
        local screen = mint.render.world_to_screen(entity.origin.x, entity.origin.y, entity.origin.z)
        
        if screen.valid then
            local text = string.format("%s [%d HP]", entity.class_name, entity.health)
            mint.render.draw_text(screen.x, screen.y, text, 255, 255, 255, 255, 14)
        end
    end)
end

function register()
    mint.gui.register_tab("Entity ESP", function()
        local r1 = mint.gui.checkbox("Show ESP", show_esp)
        if r1[1] then show_esp = r1[2] end

        local r2 = mint.gui.slider_int("Max Distance", max_distance, 100, 5000, "%d")
        if r2[1] then max_distance = r2[2] end
    end)
end
```

### Example 6: Speed Display with Color
```lua
function on_render()
    local screen = mint.render.get_screen_size()
    local speed = mint.movement.get_speed()
    
    local text = string.format("Speed: %.1f", speed)
    
    -- Color based on speed
    local r, g, b = 255, 255, 255
    if speed > 500 then
        r, g, b = 255, 0, 0  -- Red for fast
    elseif speed > 300 then
        r, g, b = 255, 255, 0  -- Yellow for medium
    else
        r, g, b = 0, 255, 0  -- Green for slow
    end
    
    mint.render.draw_text(10, 10, text, r, g, b, 255, 16)
end
```

## 🔒 Security Notes

- Scripts run in a sandboxed environment
- File I/O is restricted to read-only access to the `lua_scripts` folder
- Dangerous modules (os, io, package, debug) are removed
- Scripts cannot execute system commands or access arbitrary files
- Always review scripts before loading them

## 💡 Tips

1. **Hot-Reloading**: Scripts automatically reload when you save changes in the editor
2. **Error Handling**: Check the Script Manager for syntax errors and runtime errors
3. **Performance**: Keep `on_create_move` and `on_render` functions efficient
4. **Persistence**: Variables in scripts reset when the script reloads
5. **Testing**: Use the in-game editor to quickly test changes

## 🛠️ Troubleshooting

### Script not loading?
- Check the Script Manager for error messages
- Ensure the file has a `.lua` extension
- Verify the file is in the correct `lua_scripts` folder

### Script causing crashes?
- Disable the script in the Script Manager
- Check that GUI controls use the correct value-in/value-out pattern (see GUI API section)
- Check for infinite loops in your code
- Ensure you're not calling invalid API functions
- Lua errors are printed to the game console with a `[Lua]` prefix

### GUI not appearing?
- Make sure you're calling `mint.gui.register_tab()` in the `register()` function
- Check that the script has no errors preventing it from loading
