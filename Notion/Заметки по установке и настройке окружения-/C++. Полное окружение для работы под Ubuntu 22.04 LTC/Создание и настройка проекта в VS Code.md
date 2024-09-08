### Вариант, настройки автоматической:

- После установки всех `extensions` . Надо будет создать папку `.vscode`. В ней создать файл `c_cpp_properties.json` и внести в него следующие строки:
    
    ```JSON
    {
        "configurations": [
            {
                "name": "Linux",
                "includePath": [
                    "${workspaceFolder}/**"
                ],
                "defines": [],
                "compilerPath": "/usr/bin/clang++",
                "cStandard": "c17",
                "cppStandard": "c++20",
                "intelliSenseMode": "linux-clang-x64",
                "configurationProvider": "ms-vscode.cmake-tools",
                "compilerArgs": [
    	              "-stdlib=libc++",    # <-- Это очень важная строка на линуксе
                    "-pedantic"          # без неё может не находится библиотека 
                ]                        # STL
            }
        ],
        "version": 4
    }
    ```
    

  

### Вариант, создать с нуля своими руками:

- Заходим в папку, где будет создан тестовый проект для Visual Studio Code и в его терминале выполняем команду:
    
    ```Bash
    mkdir src && mkdir tests && mkdir third_party && mkdir build && \
    touch main.cpp && touch CMakeLists.txt
    ```
    
- в будущем допишу…