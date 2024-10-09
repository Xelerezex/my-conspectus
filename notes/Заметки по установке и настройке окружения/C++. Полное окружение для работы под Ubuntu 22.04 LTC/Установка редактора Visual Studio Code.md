---

### Ресурсы:

- Официальный [сайт](https://code.visualstudio.com/docs/setup/linux) с гайдом по установке.

---

## Этапы установки редактора.

- Устанавливаем нужные зависимости:
    
    `sudo apt-get install wget gpg`
    
- Скачиваем нужный пакет:
    
    `wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg`
    
- Подтягиваем нужные зависимости:
    
    `sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg`
    
- Включаем пакеты в лист автообновления:
    
    `sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'`
    
- Удаляем, более не нужный, пакет:
    
    `rm -f packages.microsoft.gpg`
    
- Устанавливаем дополнительный пакет для `apt`:
    
    `sudo apt install apt-transport-https`
    
- Обновляем систему:
    
    `sudo apt update`
    
- Скачиваем редактор:
    
    `sudo apt install code # or code-insiders`
    

---

## Этапы настройки окружения.

- Заходим в `Extensions` (правое меню), либо через `Ctrl + Shift + X` и там скачиваем следующие расширения:
    1. `**C/C++ Extension Pack**` **,** при установке этого пака, автоматически устанавливаются:
        1. **`C/C++ for Visual Studio Code by Microsoft`**
        2. **`C/C++ Extension UI Themes by Microsoft`**
        3. **`CMake Tools by Microsoft`**
        4. `**CMake For VisualStudio Code by twxs**`
    2. `**Dev Containers by Microsoft**`
    3. `**Remote Development by Microsoft Extension Pack**`