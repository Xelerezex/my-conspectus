---

Основной ответ, что надо написать в `settings.json`:

[https://stackoverflow.com/questions/51885784/how-to-setup-vs-code-for-c-with-clangd-support](https://stackoverflow.com/questions/51885784/how-to-setup-vs-code-for-c-with-clangd-support)

---

Добавление путей для `gcc` хедеров:

[https://github.com/clangd/clangd/issues/1564](https://github.com/clangd/clangd/issues/1564)

[https://stackoverflow.com/questions/62624352/can-i-use-gcc-compiler-and-clangd-language-server](https://stackoverflow.com/questions/62624352/can-i-use-gcc-compiler-and-clangd-language-server)

---

### Обновление зависимостей:

---

Установка последней версии `gcc`:

[https://www.dedicatedcore.com/blog/install-gcc-compiler-ubuntu/](https://www.dedicatedcore.com/blog/install-gcc-compiler-ubuntu/)

---

Установка последней версии `clang`:

[https://ubuntuhandbook.org/index.php/2023/09/how-to-install-clang-17-or-16-in-ubuntu-22-04-20-04/](https://ubuntuhandbook.org/index.php/2023/09/how-to-install-clang-17-or-16-in-ubuntu-22-04-20-04/)

Скрипт обновления кланга в системе:

[https://github.com/ShangjinTang/dotfiles/blob/05ef87daae29475244c276db5d406b58c52be445/linux/ubuntu/22.04/bin/update-alternatives-clang](https://github.com/ShangjinTang/dotfiles/blob/05ef87daae29475244c276db5d406b58c52be445/linux/ubuntu/22.04/bin/update-alternatives-clang)

---

### Итоговые файлы для VS Code:

```XML
Hover:
  ShowAKA: Yes
CompileFlags:
  Add: [
    -std=c++20,
    -I/usr/include/c++/13,
    -I/usr/include/x86_64-linux-gnu/c++/13,
    -I/usr/include/c++/13/backward,
    -I/usr/lib/gcc/x86_64-linux-gnu/13/include,
    -I/usr/local/include,
    -I/usr/include/x86_64-linux-gnu,
    -I/usr/include,
  ]
Diagnostics:
  UnusedIncludes: Strict
  ClangTidy:
    Add: [
      bugprone*,
      hicpp*,
      clang*,
      cppcoreguidelines*,
      hicpp*,
      misc*,
      modernize*,
      readability*,
    ]
    Remove: [
      cppcoreguidelines-avoid-const-or-ref-data-members,
      cppcoreguidelines-avoid-do-while,
      cppcoreguidelines-owning-memory,
      cppcoreguidelines-pro-bounds-array-to-pointer-decay,
      hicpp-named-parameter,
      hicpp-no-array-decay,
      hicpp-signed-bitwise,
      misc-const-correctness,
      misc-non-private-member-variables-in-classes,
      modernize-use-default-member-init,
      modernize-use-trailing-return-type,
      readability-convert-member-functions-to-static,
      readability-identifier-length,
      readability-named-parameter,
      readability-qualified-auto,
      readability-redundant-access-specifiers,
      readability-use-anyofallof,
    ]
```

```JSON
{
    // Cmake Extension setup:
    "cmake.useCMakePresets": "always",
    "cmake.sourceDirectory": "/home/heather/workspace/personal_projects/opengl_space/app/static",
    "cmake.buildDirectory": "/home/heather/workspace/build/opengl_space/gcc/vscode",

    // IntelliSense setup:
    "C_Cpp.intelliSenseEngine": "disabled",

    // ClangD setup:
    "clangd.path": "/usr/lib/llvm-17/bin/clangd",
    "clangd.arguments": [ 
        "--enable-config",
        "--compile-commands-dir=/home/heather/workspace/build/opengl_space/gcc/vscode",
        "--clang-tidy",
        "-j=4"
    ],
    "clangd.checkUpdates": true,
    "clangd.inactiveRegions.useBackgroundHighlight": true,
    "clangd.onConfigChanged": "restart",


    // "lldb.displayFormat": "auto",
    // "lldb.showDisassembly": "auto",
    // "lldb.dereferencePointers": true,
    // "lldb.consoleMode": "commands"
}
```

- Чисто что бы был, для референса `CMakePresets.json`
    
    ```JSON
    {
        "version": 3,
        "configurePresets": [
            {
                "name": "linux-base",
                "displayName": "Linux Debug",
                "description": "For Linux",
                "generator": "Ninja",
                "binaryDir": "/home/heather/workspace/build/opengl_space/gcc/vscode",
                "cacheVariables": {
                    "CMAKE_C_COMPILER": "/usr/bin/gcc",
                    "CMAKE_CXX_COMPILER": "/usr/bin/g++",
                    "CMAKE_EXPORT_COMPILE_COMMANDS": "1",
                    "CMAKE_BINARY_DIR": "/home/heather/workspace/build/opengl_space/gcc/vscode",
                    "CMAKE_PREFIX_PATH": "/usr/local/lib/Qt5_15_2_gcc/lib/cmake/Qt5"
                },
                "condition": {
                    "type": "equals",
                    "lhs": "${hostSystemName}",
                    "rhs": "Linux"
                },
                "vendor": {
                    "microsoft.com/VisualStudioSettings/CMake/1.0": {
                        "enableMicrosoftCodeAnalysis": false,
                        "disableExternalAnalysis": true,
                        "enableClangTidyCodeAnalysis": true,
                        "clangTidyChecks": "bugprone-*,clang-*,cppcoreguidelines-avoid-c-arrays,cppcoreguidelines-avoid-goto,cppcoreguidelines-avoid-magic-numbers,cppcoreguidelines-avoid-non-const-global-variables,cppcoreguidelines-avoid-reference-coroutine-parameters,cppcoreguidelines-c-copy-assignment-signature,cppcoreguidelines-explicit-virtual-functions,cppcoreguidelines-init-variables,cppcoreguidelines-interfaces-global-init,cppcoreguidelines-macro-usage,cppcoreguidelines-narrowing-conversions,cppcoreguidelines-no-malloc,cppcoreguidelines-non-private-member-variables-in-classes,cppcoreguidelines-prefer-member-initializer,cppcoreguidelines-pro-bounds-constant-array-index,cppcoreguidelines-pro-bounds-pointer-arithmetic,cppcoreguidelines-pro-type-*,cppcoreguidelines-slicing,cppcoreguidelines-special-member-functions,cppcoreguidelines-virtual-class-destructor,hicpp-avoid-*,hicpp-braces-around-statements,hicpp-deprecated-headers,hicpp-exception-baseclass,hicpp-explicit-conversions,hicpp-function-size,hicpp-invalid-access-moved,hicpp-member-init,hicpp-move-const-arg,hicpp-multiway-paths-covered,hicpp-new-delete-operators,hicpp-no-assembler,hicpp-no-malloc,hicpp-noexcept-move,hicpp-special-member-functions,hicpp-static-assert,hicpp-undelegated-constructor,hicpp-uppercase-literal-suffix,hicpp-use-*,hicpp-vararg,misc-confusable-identifiers,misc-definitions-in-headers,misc-misleading-*,misc-misplaced-const,misc-new-delete-overloads,misc-no-recursion,misc-non-copyable-objects,misc-redundant-expression,misc-static-assert,misc-throw-by-value-catch-by-reference,misc-unconventional-assign-operator,misc-uniqueptr-reset-release,misc-unused-*,misc-use-anonymous-namespace,modernize-avoid-*,modernize-concat-nested-namespaces,modernize-deprecated-*,modernize-loop-convert,modernize-macro-to-enum,modernize-make-*,modernize-pass-by-value,modernize-raw-string-literal,modernize-redundant-void-arg,modernize-replace-*,modernize-return-braced-init-list,modernize-shrink-to-fit,modernize-unary-static-assert,modernize-use-auto,modernize-use-bool-literals,modernize-use-emplace,modernize-use-equals-*,modernize-use-nodiscard,modernize-use-noexcept,modernize-use-nullptr,modernize-use-override,modernize-use-transparent-functors,modernize-use-uncaught-exceptions,modernize-use-using,readability-avoid-const-params-in-decls,readability-braces-around-statements,readability-const-return-type,readability-container-*,readability-delete-null-pointer,readability-duplicate-include,readability-else-after-return,readability-function-*,readability-identifier-naming,readability-implicit-bool-conversion,readability-inconsistent-declaration-parameter-name,readability-isolate-declaration,readability-magic-numbers,readability-make-member-function-const,readability-misleading-indentation,readability-misplaced-array-index,readability-non-const-parameter,readability-redundant-control-flow,readability-redundant-declaration,readability-redundant-function-ptr-dereference,readability-redundant-member-init,readability-redundant-preprocessor,readability-redundant-smartptr-get,readability-redundant-string-*,readability-simplify-*,readability-static-*,readability-string-compare,readability-suspicious-call-argument,readability-uniqueptr-delete-release,readability-uppercase-literal-suffix"
                    }
                }
            },
            {
                "name": "windows-base",
                "description": "Target Windows with the Visual Studio Code development environment.",
                "hidden": true,
                "generator": "Ninja",
                "binaryDir": "C:/workspace/build/msvc22/opengl_space",
                "cacheVariables": {
                    "CMAKE_C_COMPILER": "cl.exe",
                    "CMAKE_CXX_COMPILER": "cl.exe",
                    "CMAKE_EXPORT_COMPILE_COMMANDS": "1",
                    "CMAKE_BINARY_DIR": "C:/workspace/build/msvc22/opengl_space"
                },
                "condition": {
                    "type": "equals",
                    "lhs": "${hostSystemName}",
                    "rhs": "Windows"
                }
            },
            {
                "name": "x64-debug-win",
                "displayName": "x64 Debug",
                "description": "Target Windows (64-bit) with the Visual Studio development environment. (Debug)",
                "inherits": "windows-base",
                "architecture": {
                    "value": "x64",
                    "strategy": "external"
                },
                "cacheVariables": {
                    "CMAKE_BUILD_TYPE": "Debug"
                }
            },
            {
                "name": "x64-release-win",
                "displayName": "x64 Release",
                "description": "Target Windows (64-bit) with the Visual Studio development environment. (RelWithDebInfo)",
                "inherits": "x64-debug-win",
                "cacheVariables": {
                    "CMAKE_BUILD_TYPE": "Release"
                }
            },
            {
                "name": "x86-debug-win",
                "displayName": "x86 Debug",
                "description": "Target Windows (32-bit) with the Visual Studio development environment. (Debug)",
                "inherits": "windows-base",
                "architecture": {
                    "value": "x86",
                    "strategy": "external"
                },
                "cacheVariables": {
                    "CMAKE_BUILD_TYPE": "Debug"
                }
            },
            {
                "name": "x86-release-win",
                "displayName": "x86 Release",
                "description": "Target Windows (32-bit) with the Visual Studio development environment. (RelWithDebInfo)",
                "inherits": "x86-debug-win",
                "cacheVariables": {
                    "CMAKE_BUILD_TYPE": "Release"
                }
            },
            {
                "name": "x64-debug-linux",
                "displayName": "x64-debug-linux",
                "description": "Linux Debug",
                "inherits": "linux-base",
                "architecture": {
                    "value": "x64",
                    "strategy": "external"
                },
                "cacheVariables": {
                    "CMAKE_BUILD_TYPE": "Debug"
                }
            }
        ],
        "buildPresets": [
            {
                "name": "x64-debug-win",
                "configurePreset": "x64-debug-win",
                "inheritConfigureEnvironment": true
            },
            {
                "name": "x64-debug-linux",
                "configurePreset": "x64-debug-linux",
                "inheritConfigureEnvironment": true
            }
        ]
    }
    ```