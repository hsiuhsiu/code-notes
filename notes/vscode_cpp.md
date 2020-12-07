---
title: C++ in VSCode
tags:
  - c++
  - vscode
emoji:
---

- Follow the [official instruction](https://code.visualstudio.com/docs/cpp/config-clang-mac)

  - Note that compilation is per project setting
  - To use c++17, add `"--std=c++17"` to `tasks.json`
  - As of 2020-12-06, there is a [bug](https://github.com/JacksonKearl/testissues/issues/16845) for running debugger. Have to change the label of a comilation task. But there was another issue. Thus I just ran with terminal instead.

- Per Language setting:

  - ```json
    "[cpp]": {
        "breadcrumbs.showArrays": true,
        "editor.wordBasedSuggestions": false,
        "editor.suggest.insertMode": "replace",
        "editor.semanticHighlighting.enabled": true,
        "editor.tabSize": 2
    }
    ```

- C++ formatter
  - In setting, search `C_Cpp.clang_format_fallbackStyle`
  - Change the value to `{ BasedOnStyle: Google, IndentWidth: 2 }` (The default one may be `Visual Studio`)