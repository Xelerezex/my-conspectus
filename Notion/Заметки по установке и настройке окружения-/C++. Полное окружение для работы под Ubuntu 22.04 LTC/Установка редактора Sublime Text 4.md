---

### Ресурсы:

- Официальный [сайт](https://www.sublimetext.com/docs/linux_repositories.html) редактора Sublime Text 4.

---

## Этапы установки редактора.

- Добавляем их репозиторий:
    
    `wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/sublimehq-archive.gpg > /dev/null`
    
- Подсасываем стабильную версию с сайта:
    
    `echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list`
    
- Обновляем сорсы:
    
    `sudo apt-get update`
    
- Устанавливаем расширения для apt:
    
    `sudo apt-get install apt-transport-https`
    
- Устанавливаем редактор:
    
    `sudo apt-get install sublime-text`
    

---

## Этапы настройки редактора.

### Установка редактора - редактором по умолчанию в системе:

- В первую очередь, стоит сделать саблайм редактором по умолчанию, в системе:
    
    Набираем следующую команду в терминале:
    
    `sudo subl /usr/share/applications/defaults.list`
    
- Зажимаем `ctrl + f` слове `gedit` и изменяем все вхождения `gedit`'а на `sublime_text`.
- Так же есть способ такой:
    - Нажимаем на нужный файл **правой кнопкой мыши.**
    - Выбираем **свойства/properties**.
    - Заходим в раздел **Открыть с помощью/Open with**.
    - Выбираем `Sublime Text` и нажимаем кнопку **Установить по умолчанию/Set by default**.

### Полная настройка редактора, для комфортной работы:

- Сначала нужно найти в верхнем тулбаре поле `Tools` и в нем нажать `install package control`.
- Вводим в редакторе сочетание клавиш `ctrl + shift + p`, начинаем вводить в появившемся окне: `Package Control: Install Package`.
    
    После этого появится поле, где нужно будет вводить имя пакета, и будет происходить его установка
    
    Пакеты, которые следует установить:
    
    1. `SublimeRELP`
    2. `Predawn`
    3. `Material Theme`
    4. `BracketHighlighter`
    5. `SideBarEnhancements`
    6. `Anaconda`
    7. `Gist`
    8. `Markdown Preview`
        
        Далее переходим `Preferences` → `Key Bindings` и вводим следующую строку
        
        `{"keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"}},`.
        
        Это нужно, что бы через сочетание `alt+m` файлы разметки отображались в браузере, который стоит по умолчанию.
        
    9. `SQLTools`
- Следующим шагом нужно записать все настройки редактора.
- Заходим в `Preferences` → `Settings` и в часть редактора `Preferences.sublime-settings` вводим, следующий текст:
    
    ```C++
    {
        "theme": "Material-Theme-Darker.sublime-theme",
        "color_scheme": "Packages/Predawn/predawn.tmTheme",
    
        "show_definitions": true,
    
        "bold_folder_labels": true,
        "caret_extra_width": 1,
        "caret_style": "phase",
        "close_windows_when_empty": false,
        "copy_with_empty_selection": false,
        "drag_text": false,
        "draw_minimap_border": true,
        "draw_white_space": "none",
        "enable_tab_scrolling": false,
        "ensure_newline_at_eof_on_save": true,
        "file_exclude_patterns":
        [
            "*.pyc",
            "*.pyo",
            "*.exe",
            "*.dll",
            "*.obj",
            "*.lib",
            "*.so",
            "*.dylib",
            "*.ncb",
            "*.sdf",
            "*.suo",
            "*.pdb",
            "*.idb",
            ".DS_Store",
            "*.class",
            "*.psd",
            "*.sublime-workspace"
        ],
        "font_face": "Source Code Pro",
        "font_options":
        [
            "no_round"
        ],
        "font_size": 12,
        "highlight_line": true,
        "highlight_modified_tabs": true,
        "ignored_packages":
        [
            "ActionScript",
            "AppleScript",
            "ASP",
            "D",
            "Diff",
            "Erlang",
            "Graphviz",
            "Groovy",
            "HTML-CSS-JS Prettify",
            "Lisp",
            "Lua",
            "Objective-C",
            "OCaml",
            "Rails",
            "Ruby",
            "Vintage",
        ],
        "installed_packages":
        [
            "Anaconda",
            "BracketHighlighter",
            "Material Theme",
            "Predawn",
            "SideBarEnhancements"
        ],
        "line_padding_bottom": 1,
        "line_padding_top": 1,
        "match_brackets_content": false,
        "match_selection": false,
        "match_tags": false,
        "material_theme_accent_graphite": true,
        "material_theme_compact_sidebar": true,
        "mini_diff": false,
        "open_files_in_new_window": false,
        "overlay_scroll_bars": "enabled",
        "preview_on_click": false,
        "scroll_past_end": true,
        "scroll_speed": 5.0,
        "show_encoding": true,
        "show_errors_inline": false,
        "show_full_path": false,
        "sidebar_default": true,
        "swallow_startup_errors": true,
        "translate_tabs_to_spaces": true,
        "trim_trailing_white_space_on_save": true,
        "use_simple_full_screen": true,
        "word_wrap": false,
    }
    ```
    
- И напоследок добавим две build-системы для _Python_ и _C++_
    
    Открываем `Tools` → `Build System` → `New Build System…`
    
    В открывшийся файл добавляем строки:
    
    ```C++
    {
        "cmd": ["/usr/bin/python3", "-u", "$file"],
    	  "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    	  "quiet": true
    }
    ```
    
    И сохраняем файл как `Python3.sublime-build`
    
    ---
    
    Создадим еще один файл при помощи `Tools` → `Build System` → `New Build System…`
    
    Назовем его `C++17.sublime-build` и вставим следующий код:
    
    ```C++
    {
        "shell_cmd": "clang++ -ltbb -pthread -Wall -Wextra -O2 -fwrapv -Wfloat-equal -Wconversion -std=c++17 \"${file}\" -o \"${file_path}/${file_base_name}\"",
        "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
        "working_dir": "${file_path}",
        "selector": "source.c++",
    
        "variants":
        [
            {
                "name": "Run",
                "shell_cmd": "clang++ -ltbb -pthread -std=c++17 -Wall -Wextra -O2 -fwrapv -Wfloat-equal -Wconversion \"${file}\" -o \"${file_path}/${file_base_name}\" && \"${file_path}/${file_base_name}\""
            }
        ]
    }
    ```