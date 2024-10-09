---

### Ресурсы:

1. [Install docker on Ubuntu](https://docs.docker.com/desktop/install/ubuntu/).
2. [Install docker on Ubuntu: set up repository](https://docs.docker.com/engine/install/ubuntu/#set-up-the-repository).
3. [Uninstall docker engine](https://docs.docker.com/engine/install/ubuntu/#uninstall-docker-engine).
4. [Manage docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user).

---

Здесь будет описана установка `docker`, команды будут исполняться на `ubuntu 22.04`.

### Настроем репозиторий на рабочей машине:

1. Первым делом нужно удалить потенциальные старые версии докера:
    
    ```Bash
    sudo apt-get remove docker docker-engine docker.io containerd \ 
    										runc docker-desktop
    rm -r $HOME/.docker/desktop
    sudo rm /usr/local/bin/com.docker.cli
    ```
    
2. На всякий случай скачаем `gnome-terminal`, если его вдруг нет в системе:
    
    ```Bash
    sudo apt install gnome-terminal
    ```
    
3. Теперь обновим индексы `apt` и докачаем нужные зависимости:
    
    ```Bash
    sudo apt-get update
    
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```
    
4. Установим официальный `docker GPG key`:
    
    ```Bash
    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```
    
5. Настроем репозиторий:
    
    ```Bash
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
    

### Установим сам `docker engine`:

1. Обновим индексы `apt`:
    
    ```Bash
    sudo apt-get update
    ```
    
    - Если вдруг возникла ошибка при исполнении `sudo apt-get update` во время обновления `GPG`, надо ввести следующие команды:
        
        ```Bash
        sudo chmod a+r /etc/apt/keyrings/docker.gpg
        sudo apt-get update
        ```
        
2. Теперь установим последние версии `docker engine`, `docker compose` и контейнеризацию:
    
    ```Bash
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    
      
    
3. Проверим правильность установки `docker`: 
    
    ```Bash
    sudo docker run hello-world
    ```
    
    Должно вывести что-то типа того:
    
    ```Bash
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    
    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.
    
    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash
    
    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/
    
    For more examples and ideas, visit:
    https://docs.docker.com/get-started/
    ```
    

### Первичная настройка после установки:

1. Можно настроить автозапуск `docker`, при загрузке системы:
    
    ```Bash
    sudo systemctl enable docker.service
    sudo systemctl enable containerd.service
    ```
    
    Отключить данную опцию можно так:
    
    ```Bash
    sudo systemctl disable docker.service
    sudo systemctl disable containerd.service
    ```
    
    Проверить работает ли docker в данный момент можно следующей командой:
    
    ```Bash
    sudo service docker status
    ```
    
2. Настроем `docker` как пользователя, которому не нужно разрешения от `root` . Это нужно для его связи с VS Code. Но стоит быть аккуратным, так как данная операция расшатывает безопасность:
    
    ```Bash
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```