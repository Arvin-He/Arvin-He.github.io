---
title: vscode插件和配置
date: 2017-05-25 10:14:46
tags: [Tools]
categories: 工具
---
### 插件
Align
Beautify
C/C++
Emoji Code
HTML Snippets
JavaScript(ES6) code snippets
jsx
One Dark Theme
Path Intellisense
Python
Reactjs code snippets
Runner
vscode-icons

### 配置
{
    "[cpp]": {
        "editor.quickSuggestions": false
    },
    "[c]": {
        "editor.quickSuggestions": false
    },
    // "editor.wordWrap": "bounded",
    // "editor.wordWrapColumn": 80,
    // 控制字体系列
    "editor.fontFamily": "Consolas, 'Courier New', monospace, '宋体'",
    // 以像素为单位控制字号。
    "editor.fontSize": 16,
    // 控制选取范围是否有圆角
    "editor.roundedSelection": false,
    // 建议小组件的字号
    "editor.suggestFontSize": 14,
    // 在“打开的编辑器”窗格中显示的编辑器数量。将其设置为 0 可隐藏窗格。
    "explorer.openEditors.visible": 0,
    // 是否已启用自动刷新
    "git.autorefresh": false,
    // 是否启用了自动提取。
    "git.autofetch": false,
    // 以像素为单位控制终端的字号，这是 editor.fontSize 的默认值。
    "terminal.integrated.fontSize": 16,
    // 控制终端游标是否闪烁。
    "terminal.integrated.cursorBlinking": true,
    // 一个制表符等于的空格数。该设置在 `editor.detectIndentation` 启用时根据文件内容进行重写。
    "editor.tabSize": 4,
    "editor.minimap.enabled": false,
    // Tab Size
    "beautify.tabSize": 4,

    "python.linting.pylintEnabled": false,
    "python.linting.pep8Enabled": true,
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 1000,
    "workbench.iconTheme": "vscode-icons",
    "window.zoomLevel": 1,
    "workbench.colorTheme": "Monokai"
}
