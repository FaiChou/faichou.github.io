---
layout: post
title: "Sublime And VSCode Configuration"
date: 2017-08-24
---

### Sublime

```json
{
  "always_show_minimap_viewport": true,
  "bold_folder_labels": true,
  "color_scheme": "Packages/User/SublimeLinter/Dracula (SL).tmTheme",
  "font_face": "Source Code Pro",
  "font_options":
  [
    "gray_antialias"
  ],
  "font_size": 17,
  "ignored_packages":
  [
    "JSLint",
    "Vintage"
  ],
  "indent_guide_options":
  [
    "draw_normal",
    "draw_active"
  ],
  "line_padding_bottom": 2,
  "line_padding_top": 2,
  "overlay_scroll_bars": "enabled",
  "tab_size": 4,
  "theme": "Default.sublime-theme",
  "translate_tabs_to_spaces": true
}
```

### Sublime Python3 Build System

```json
{
    "cmd": ["python3", "-u", "$file"],
    "file_regex": "^[ ]File \"(...?)\", line ([0-9]*)",
    "selector": "source.python"
}
```

### VSCode

```json
{
  "window.zoomLevel": 1,
  "editor.fontFamily": "Source Code Pro, Menlo, Monaco, 'Courier New', monospace",
  "editor.lineHeight": 20,
  "workbench.colorTheme": "Dracula Soft",
  "workbench.iconTheme": "vscode-icons",
  "editor.tabSize": 2,
  "workbench.startupEditor": "newUntitledFile",
  "editor.fontSize": 14,
  "terminal.integrated.fontSize": 14,
  "vsicons.dontShowNewVersionMessage": true,
  "workbench.editor.enablePreviewFromQuickOpen": false
}
```