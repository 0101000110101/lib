# ui_library.lua

A Roblox (executor) UI library extracted from your script: windows, tabs, columns, sections,
toggles, sliders, dropdowns, color pickers, keybinds, textboxes, buttons, labels, lists,
notifications, watermark, config save/load and live theming.

Only the UI code was extracted. Game features, the anticheat bypass and remote asset
downloads were left out. Files: `ui_library.lua` (the library), `example.lua` (demo).

## Quick start
```lua
local library = loadstring(readfile("ui_library.lua"))()
local window  = library:window({ name = "my menu" })
local tab     = window:tab({ name = "main" })
local section = tab:column():section({ name = "stuff" })
section:toggle({ name = "enabled", flag = "enabled", callback = function(v) end })
tab.open_tab()
```
Read any value any time with `library.flags["flag_name"]`.

## API
| Call | Options | Notes |
|---|---|---|
| `library:window{name,size}` | | `window.set_menu_visibility(bool)` shows/hides the menu |
| `library:watermark{text}` | | `watermark.change_text(str)` |
| `library:notification{text,time}` | | |
| `window:tab{name}` | | `tab.open_tab()` |
| `tab:column()` | | |
| `column:section{name}` | | |
| `column:multi_section{names={...}}` | | returns one section per name (sub-tabs) |
| `section:toggle{name,flag,default,tooltip,callback}` | | chain `:keybind{}` / `:colorpicker{}` |
| `section:slider{name,flag,min,max,default,interval,suffix,callback}` | | |
| `section:dropdown{name,flag,items,default,multi,scrolling,callback}` | | `refresh_options(list)` |
| `section:label{name}` | | chain `:colorpicker{}` / `:keybind{}`; `change_text(str)` |
| `x:colorpicker{name,flag,color,alpha,callback(color,alpha)}` | | |
| `x:keybind{name,flag,key,mode,callback}` | | modes: toggle / hold / always |
| `section:textbox{flag,placeholder,default,callback}` | | |
| `section:button{name,callback}` | | group with `section:button_holder{}` |
| `section:list{flag,items,callback}` | | searchable list, `refresh_options(list)` |
| `section:playerlist{}` | | needs `Players` (included) |

Every element's `cfg.set(value)` sets it programmatically (this also runs the callback).

## Adding your own element
Follow the existing pattern in `ui_library.lua`:
```lua
function library:my_element(options)
    local cfg = { name = options.name or "x", flag = options.flag or library:next_flag(),
                  callback = options.callback or function() end }
    local frame = library:create("TextLabel", { Parent = self.holder, Text = cfg.name,
                  FontFace = library.font, Size = UDim2.new(1, 0, 0, 14), BackgroundTransparency = 1 })
    function cfg.set(v) flags[cfg.flag] = v; cfg.callback(v) end
    flags[cfg.flag] = nil
    library.config_flags[cfg.flag] = cfg.set   -- makes it save/load with configs
    return setmetatable(cfg, library)
end
```
* `library:create` auto-themes text; use `library:apply_theme(obj, "accent", "BackgroundColor3")` for theming.
* Register `config_flags[flag] = setter` so configs save/load it.
* `library.themes.preset` holds current colors; `library:update_theme(name, color)` recolors live.

## Configs
```lua
library.config_holder = section:list({ flag = "config_list" })  -- optional list UI
library:config_list_update()
writefile(library.directory .. "/configs/name.cfg", library:get_config())
library:load_config(readfile(library.directory .. "/configs/name.cfg"))
```

## Changes from the original
* Removed `nebula` / `headshots` references (they crashed `new_item`/`new_drawing`).
* Font: built-in Roblox font by default; `library:load_font(url)` to use your own `.ttf`.
* Placeholder flags are auto-generated unique names (`library:next_flag()`).
* Added `library.themes`, `library.keys`, `library.config_holder`, `library.drawings`, `library.instances`.
* Known quirk left as is: `visible = options.visible or true` is always true.
* Untested: no Lua interpreter was available here, so it has only had a block-balance check.
  Run `example.lua` in your executor and tell me about any errors.

## Customization (new)
```lua
-- one call builds a full appearance panel (presets, every color, width/height sliders, font, theme import/export)
library:theme_editor(settings_column:section({ name = "appearance" }), window)
```
Or drive things from code:
```lua
library:set_theme({ accent = Color3.fromRGB(80,140,255), low_contrast = Color3.fromRGB(18,22,36), high_contrast = Color3.fromRGB(26,32,50) })
library:set_contrast(low, high)          -- window background gradient only
library:set_font(Enum.Font.Code)         -- or a Font object
library.theme_presets.myname = { ... }   -- add your own preset (same keys as "default")
library.min_window_size = { x = 380, y = 300 }  -- smallest size the resize handle allows
window.set_size(700, 500)  window.center()
library:export_theme() / library:import_theme(json)
```
Theme keys: `accent, outline, inline, text, text_outline, glow, low_contrast, high_contrast`.
The window can also be resized by dragging its bottom-right corner.
Not included: UI scale (popups like dropdowns are parented outside the window and would misalign) and per-element text size.
