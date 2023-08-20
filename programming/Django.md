### Informative git prompt for bash

[bash-git-prompt](https://github.com/magicmonty/bash-git-prompt)

клонируем репозиторий:

    git clone https://github.com/magicmonty/bash-git-prompt.git ~/.bash-git-prompt --depth=1

Редактируем файл ~/.bashrc:

    if [ -f "$HOME/.bash-git-prompt/gitprompt.sh" ]; then
        GIT_PROMPT_ONLY_IN_REPO=1
        source $HOME/.bash-git-prompt/gitprompt.sh
    fi

    GIT_PROMPT_ONLY_IN_REPO=1

Обновляем настройки:

    source ~/.bash-git-prompt/gitprompt.sh


## Работа c GIT

Инициализация и заливка на гитхаб:

    $ git init
    $ git add .
    $ git commit -m "Initial commit."
    $ git remote add origin https://github.com/ayfaar/blogengine.git
    $ git push -u origin master

Создадим ветку для разработки с именем develop, переключимся на нее и зальем на гитхаб.

    $ git checkout -b develop
    $ git push -u origin develop

***

# VIM


## Плагины

	+ [vim-plug](https://github.com/junegunn/vim-plug)

# KHKKH

