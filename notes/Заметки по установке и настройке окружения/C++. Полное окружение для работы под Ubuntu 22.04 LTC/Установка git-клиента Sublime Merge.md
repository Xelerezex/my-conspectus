---

### Ресурсы:

- Официальный [сайт](https://www.sublimemerge.com/docs/linux_repositories) git-клиента Sublime Merge

---

## Этапы установки git-клиента.

- Добавляем их репозиторий:
    
    `wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/sublimehq-archive.gpg > /dev/null`
    
- Устанавливаем расширения для apt:
    
    `sudo apt-get install apt-transport-https`
    
- Подсасываем стабильную версию с сайта:
    
    `echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list`
    
- Обновляем сорсы:
    
    `sudo apt-get update`
    
- Устанавливаем редактор:
    
    `sudo apt-get install sublime-merge`
    

---