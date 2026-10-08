ui_library.lua
A single-file Roblox (executor) UI library: windows, tabs, columns, sections, a full set of
elements, live theming, config saving, notifications, a watermark and a loading screen.
Everything lives in `ui_library.lua`. It returns one table, `library`.
```lua
local library = loadstring(readfile("ui_library.lua"))()
-- or: loadstring(game:HttpGet("https://raw.githubusercontent.com/YOU/REPO/main/ui_library.lua"))()
```
Requires an executor with `cloneref`, `writefile` / `readfile` / `listfiles` / `makefolder` / `isfile`.
`gethui` and `getcustomasset` are used if present. Without the file functions, configs won't save
but everything else works.
---
Quick start (with loading screen)
```lua
local library = loadstring(readfile("ui_library.lua"))()
library.directory = "mymenu"                       -- folder for configs (optional)

local loader = library:loader({ title = "My Script", status = "starting..." })

loader.run({
    { name = "building menu", callback = function()
        local window  = library:window({ name = "My Script" })
        local tab     = window:tab({ name = "main" })
        local section = tab:column():section({ name = "settings" })

        section:toggle({ name = "enabled", flag = "enabled",
            callback = function(value) print("enabled:", value) end })
        section:slider({ name = "amount", flag = "amount", min = 0, max = 100, default = 50 })

        tab.open_tab()
    end },
})
```
Without a loading screen, just call `library:window(...)` directly.
Read any value at any time with `library.flags["flag_name"]`.
---
Loading screen
```lua
local loader = library:loader({
    title = "My Script",        -- big accent-colored title
    status = "starting...",     -- small text under it
    done_text = "done",         -- optional, shown at 100%
    hold = 0.45,                -- optional, seconds to wait at 100% before fading
    stop_on_error = true,       -- optional, close if a step fails (set false to keep going)
})
```
Method	What it does
`loader.set_progress(fraction, text?)`	Moves the bar (0 to 1) and optionally changes the status text
`loader.set_status(text)`	Changes only the status text
`loader.run(steps, on_done?)`	Runs `{ { name = "...", callback = fn, delay = 0.12 }, ... }` in order, updating progress
`loader.finish(callback?)`	Fills the bar, fades out, destroys, then calls `callback`
`loader.destroy()`	Closes it immediately
It uses the same frames, theme colors and font as the window, so presets and `library:set_theme`
change it too. If a step errors, the loader shows `failed: <step>`, prints the error, and closes.
---
API
Window and layout
Call	Notes
`library:window{ name, size }`	Returns `window`. Resizable from the bottom-right corner.
`window:tab{ name }`	Returns `tab`. Call `tab.open_tab()` to select it.
`tab:column()`	Returns a column.
`column:section{ name }`	Returns a section (a titled box of elements).
`column:multi_section{ names = {"a","b"} }`	Returns one section per name (sub-tabs).
`window.set_menu_visibility(bool)`	Show / hide the menu.
`window.set_size(width, height)`	Pass `nil` to keep one axis. Clamped to `library.min_window_size`.
`window.center()`	Centers the window on screen.
`library:watermark{ text }`	Returns object with `change_text(str)`.
`library:notification{ text, time }`	Pop-up message.
Elements (called on a section)
Call	Options
`section:toggle{}`	`name, flag, default, tooltip, callback(bool)` - chain `:keybind{}` or `:colorpicker{}`
`section:slider{}`	`name, flag, min, max, default, interval, suffix, callback(number)`
`section:dropdown{}`	`name, flag, items, default, multi, scrolling, callback(value)` - `:refresh_options(list)`
`section:label{}`	`name` - chain `:keybind{}` / `:colorpicker{}`; `.change_text(str)`
`section:button{}`	`name, callback()`
`section:button_holder{}`	Row that holds several buttons side by side
`section:textbox{}`	`flag, placeholder, default, callback(text)`
`section:list{}`	`flag, items, placeholder, size, callback(value)` - searchable, `.refresh_options(list)`
`section:playerlist{}`	Built-in player list
`x:colorpicker{}`	`name, flag, color, alpha, callback(color, alpha)`
`x:keybind{}`	`name, flag, key, mode ("toggle"/"hold"/"always"), default, callback(active)`
Every element returns an object whose `.set(value)` changes it from code (and runs its callback).
Flag value types: toggle = boolean, slider = number, dropdown = string (table when `multi = true`),
textbox = string, colorpicker = `{ Color, Transparency }`, keybind = `{ active, mode, key }`.
If you leave out `flag`, a unique one is generated (`library:next_flag()`).
---
Customization
```lua
-- Build a full appearance panel in one call
library:theme_editor(column:section({ name = "appearance" }), window)

-- or drive it from code
library:set_theme({ accent = Color3.fromRGB(80,140,255), low_contrast = Color3.fromRGB(18,22,36) })
library:set_contrast(low, high)           -- background gradient only
library:set_font(Enum.Font.Code)          -- or a Font object; restyles existing + future text
library:load_font("https://.../font.ttf") -- call BEFORE creating the window
library:export_theme()                    -- JSON string
library:import_theme(json)                -- returns true/false
```
Theme keys: `accent, outline, inline, text, text_outline, glow, low_contrast, high_contrast`.
Built-in presets: `default, midnight, crimson, emerald, mono`. Add your own with
`library.theme_presets.myname = { ...same keys... }`.
`library.min_window_size = { x = 380, y = 300 }` sets the smallest size the resize handle allows.
---
Configs
```lua
library.config_holder = section:list({ flag = "config_list" })   -- optional list UI
library:config_list_update()                                     -- refresh it from disk

writefile(library.directory .. "/configs/name.cfg", library:get_config())
library:load_config(readfile(library.directory .. "/configs/name.cfg"))
```
Everything created with a flag is saved and restored. Files go in `<library.directory>/configs`
(default directory: `uilib`).
---
Running code on changes
```lua
-- per change
section:toggle({ name = "x", flag = "x", callback = function(value) end })

-- per frame; library:connection() disconnects it for you on unload
library:connection(game:GetService("RunService").RenderStepped, function()
    if not library.flags.x then return end
end)
```
Unloading
```lua
for _, gui in library.guis do gui:Destroy() end
for _, con in library.connections do con:Disconnect() end
```
---
Adding your own element
```lua
function library:my_element(options)
    local cfg = {
        name = options.name or "x",
        flag = options.flag or library:next_flag(),
        callback = options.callback or function() end,
    }

    local label = library:create("TextLabel", {
        Parent = self.holder, Text = cfg.name, FontFace = library.font,
        Size = UDim2.new(1, 0, 0, 14), BackgroundTransparency = 1,
    })

    function cfg.set(value)
        library.flags[cfg.flag] = value
        cfg.callback(value)
    end

    library.flags[cfg.flag] = nil
    library.config_flags[cfg.flag] = cfg.set    -- saves / loads with configs
    return setmetatable(cfg, library)
end
```
`library:create` themes text automatically; use `library:apply_theme(obj, "accent", "BackgroundColor3")`
for other themed properties.
`library.themes.preset` holds the current colors; `library:update_theme(name, color)` recolors live.
---
Notes and limitations
Untested here: the code was only checked for balanced blocks, since no Lua runtime was available.
If something errors, the console message and line number will point to the cause.
`visible = options.visible or true` in the element constructors is always true.
No UI-scale or per-element text-size option: dropdown popups live outside the window and would misalign.
`library.directory` defaults to `uilib`; set it before saving configs.
