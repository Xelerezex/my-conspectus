---

### Ресурсы:

- Основная [статья](https://levelup.gitconnected.com/how-to-combine-c-cmake-googletest-circleci-docker-and-why-e02d76c060a3), откуда брались все идеи окружения.

---

## Этап первый: Установка основных пакетов.

- Для начала обновляем всю систему. И в дальнейшем запишем это команду в `alias` как `fully`.
    
    `sudo apt-get --yes update && sudo apt-get --yes full-upgrade && sudo apt-get --yes autoremove`
    
- Устанавливаем все, основные компиляторы, контроль версий и т.д.
    
    ```Bash
    sudo apt-get install --yes \
    		apt-transport-https \
    		build-essential \
    		g++ \
    		aptitude \
    		libc++-dev \
    		git \
    		make \
    		cmake \
    		ninja-build \
    		tar \
    		curl \
    		clang \
        lld \
    		lldb \
    		libfontconfig1 \
        mesa-common-dev
    	
    ```
    
- Далее мы забьем следующее выражение `echo -e "\n"` в `alias` под именем `newline`.
- Проверяем правильность установки всех основных средств
    
    ```Bash
    g++ --version; newline;
    aptitude --version; newline;
    git --version; newline;
    make --version; newline;
    cmake --version; newline;
    ninja --version; newline;
    tar --version; newline;
    curl --version; newline;
    clang --version; newline;
    lldb --version; newline;
    lld --version; newline;
    ```
    

## Этап второй: Установка/Настройка основных редакторов текста и менеджеров версий.

[[Установка редактора Sublime Text 4]]

[[Установка git-клиента Sublime Merge]]

[[Установка редактора Visual Studio Code]]

[[Установка Docker]]

[[Установка Qt]]

## Этап третий: Создаем первый тестовый проект с полной структурой.

[[Создание и настройка проекта в VS Code]]

# Настройка VS Code.

[[Настройка Clangd с gcc для VS Code]]