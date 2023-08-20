# GIT

## Настройка Git

Файлы настроек:

+ **/etc/gitconfig**
  общие настройки для всех(системные)
+ **~/.gitconfig**
  настройки для пользователя
+ **/proj_dir/.git/config**
  настройки для проекта

**git config** - команда для конфигурирования: 
  ключи:
   + без ключа - настройки для проекта
   + --global - настройки для отдельных пользователей
   + --system - настройка для всех пользователей

Параметры установки окончаний строк:

1) Windows:
   ```
   git config --global core.autocrlf input
   git config --global core.safecrlf true
   ```

***

# sudo

Все настройки содержатся в файле **/etc/sudoers**. Редактирование этого файла осуществляется при помощи команды **visudo**. По умолчанию данный файл открывается редактором vi, но можно переопределить через переменные окружения: **SUDO_EDITOR**, **VISUAL** или **EDITOR**.
При вызове sudo переменные окружения пользователя по умолчанию не передаются, для их передачи использовать ключ *-E*:

```
$ sudo -E pacman -Syu
```

Для передачи sudo требуемых переменных окружения нужно  добавить, например, следующую строку в файл /etc/sudoers или создать файл в каталоге **/etc/sudoers.d/** c содержанием:

```
Defaults env_keep += "*_proxy *_PROXY"
```

Что приведет к импорту всех переменных окружения подходящие указанным маскам.

Синтаксис настройки запуска команд в /etc/sudoers:

**<кто> <где>=(от чьего имени:группы)(что выполнить)**

```
user_name ALL=(ALL:ALL) ALL
%group_name ALL=(ALL:ALL) ALL
user_name HOST_NAME=(root:rootgroup) /usr/bin/shutdown
user_name HOST_NAME=(root:rootgroup) NOPASSWD:/usr/bin/shutdown
```

В /etc/sudoers можно использовать алиасы,имена с заглавных букв, они делятся на группы:

* User_Alias - алиас для группировки пользователей которые запускают команду(в нашем шаблоне <кто>)
* Host_Alias - для группировки хостов
* Runas_Alias - для группировки от чьегоимени запускается выполнение команды
* Cmnd_Alias - для группировки комманд

```
User_Aias NO_ADMIN_USERS=aser1, asuer2,!root
Host_Alias MY_HOSTS=host1,192.162.10.0/255.255.255.0
Runas_Alias RUN_US=root
Cmnd_Alias POWER=/usr/bin/reboot, /usr/bin/shutdown

NO_ADMIN_USERS MY_HOSTS=(RUN_US) POWER
```
   замечание **!root** - не root пользователь

# tmux

## tmux pluggins

+ [Tmuxp](https://github.com/tmux-python/tmuxp)
  A session manager for tmux. Built on *libtmux*.
  Для того, чтобы настроить под определенный проект tmux(создать разные окна и панели, запустить в них требуемые команды, и т.д, и т.п.), используются *yaml* файлы. 
  Пример *yaml* файла *./mysession.yaml*:
  
  ```
  session_name: 4-pane-split
  windows:
    - window_name: dev window
      layout: tiled
      shell_command_before:
        - cd ~/ # run as a first command in all panes
      panes:
        - shell_command: # pane no. 1
            - cd /var/log # run multiple commands in this pane
            - ls -al | grep \.log
        - echo second pane # pane no. 2
        - echo third pane # pane no. 3
        - echo forth pane # pane no. 4
  ```

  Загрузка данной конфигурации:
  ```
  $ tmuxp load ./mysession.yaml
  ```
  Загрузка конфигурации и назначение имени сессии:
  ```
  $ tmuxp load -s session_name ./mysession.yaml
  ```

+ [Tmux Resurrect](https://github.com/tmux-plugins/tmux-resurrect).  
  Восстанавливет tmux сессию даже после перегрузки компьютера.
  
  ```
  prefix + Ctrl-s - save
  prefix + Ctrl-r - restore
  ```

## tmux НАСТРОЙКА

[Базовый конфиг tmux](https://github.com/gpakosz/.tmux)
У данного конфига реализовано удобное копирование в системный буфер:

```
PrefixKey(Ctrl+b) Enter
После чего выделяем требуемый текст и копируем в системный буфeр обмена Ctrl+c
```

**~/.tmux.conf.local**

```
# включим подсветку активной панели
tmux_conf_theme_highlight_focused_pane=true
```

Файл настроек - **/etc/tmux.conf**

Включить мышь:

```
set -g mouse on
```
или комбинацие клавиш
```
prefixkey m
```

## tmux РАБОТА

Создать именованную сессию(ключ -d, что-бы не подключаться к сессии):

```
tmux new -s SESSION_NAME -d
```


Список запущенных сессий:

```
tmux ls
```

Открыть ранее закрытую сессию, если не указать имя сессии, то откроется последняя закрытая:

```
tmux attach -t SESSION_NAME
```

Уничтожить сессию с определенным именем:

```
tmux kill-session -t SESSION_NAME
```

Уничтожить все сессии:

```
tmux kill-server
```

## tmux shortcuts

PREFIX KEY - **CTRL+b**:

    ? - список кейбиндингов;

    s - вывести список сессий;
    $ - переименовать сессию;
    d - открепиться от текущей сессии;

    c - создать окно и сделать его активным;
    & - закрыть окно
    , - переименовать окно;
    n(C-l) - перейти в следующее окно;
    p(C-h) - прейти в предыдущее окно;
    (tab) - переключение между 2мя последними окнами;
    w - вывести список окон в текущей сессии;

    "(_) - разделить окно на панели по горизонтали;
    %(-) - резделить окно на панели по вертикали;
    h,j,k,l - перемещение между панелями;
    o - переключиться на следуэщую панель;
    x - закрыть текущую панель;
    H,J,K,L - изменнение размеров панелей;
    ! - открыть панель в новом окне;
    + - развернуть/свернуть текущую панель;

    [ - скопировать в буфер tmux;
    ] - вставить из буфера tmux;
    b - список буферов tmux;
    p - ставить последний скопированный буфер;
    P - выбрать какой буфер вставить;


***

# OPENSSH

**~/.ssh/known_hosts**
**~/.ssh/authorized_keys**
**~/.ssh/config**

Вывод информации о подключениях ssh:

```
$ journalctl -fu ssh
```

или

```
# tail -f /var/log/auth.log
```

Сгенерировать хост ключи:

```
# ssh-keygen -A
```

**~/.ssh/known_hosts** - в данном файле хранятся открытый host key компьютера, к которым мы раньше подключались(на каждой строке отдельный компьютер). На каждой строчке находится отдельный отпечаток. Эта технология используется от атак "человек посередине", если злоумышленник подменит компьютер, к которому мы ранее подключались, то ssh агент при подключении заметит это и прервет подключение.


Для того чтобы получить фингерпринт публичного ключа конкретного хоста:
1) получаем публичные ключи:
```
# ssh-keyscan IP-addr
```
2) При подключении к удаленному серверу в первый раз указывается с какого ключа делается фингерпринт для файла known_hosts, мы можем посчитать его на целевой машине(к которой подключаемся), :
```
# ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key*
```

Пример подключения по ssh:

```
# ssh -p 2222 -i /path/to/key user@192.168.100.100

   где, -p - номер ssh порта удаленного сервера  
        -i - путь к приватному ключу
```



Для того, чтобы постоянно не вводить параметры для, каждого отдельного сервера, с которым хотим установить ssh соединение, то нужно создатьфайл **~/.ssh/config**, пример содержания:

```
Host server1
   HostName 192.168.100.100
   User user
   Port 2222
   IdentityFile /path/to/privat/key
   IdentitiesOnly Yes

Host server2
   HosName 192.168.100.102
   User user2
   Port 2222
   IdentityFile /path/to/private/key
   IdentitiesOnly yes
```

Для более безопасного подключения следует настроить доступ по ключу. Предпочтительный тип ключа - **ed25519**, данный ключ является более безопасный чем **rsa** ключи.  
Сгенерируем пару ключей(приватный и публичный):

```
$ ssh-keygen -t ed25519 -C "Comment for key"
```

Желательно задать мастер пароль для закрытого ключа, если он не был задан в процессе создания или нужно изменить, то следует выполнить команду:

```
$ ssh-keygen -p -f ~/.ssh/key_file
```

Далее при помощи команды *ssh-copy-id* копиуем сгенерированный нами публичный ключ на сервер, с которым мы хотим устанавливать соединение по ключу. Данная команда добавит на новую строку открытый ключ пользователя в файл ***~/.ssh/authorized_keys*(аналогично можно просто скопировать вручную в данный файл открытый ключ):

```
$ ssh-copy-id -i ~/path/to/key user_name@192.168.100.100
```

или

```
$ cat ~/.ssh/id_ed25519.pub | ssh user_name@192.168.100.10 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Для того, чтобы постоянно не вводить пароль, то для текущей сесси терминала следует запустить **ssh-agent** и подключить к нему требуемые ключи:

```
$ eval "$(ssh-agent)"
$ ssh-add /path/to/key_file
```

Отключим доступ по паролю и доступ рут пользователя, правим файл **/etc/ssh/sshd_config**:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Разное:

Посмотреть отпечаток публичного ключа:

```
$ ssh-keygen -lf /path/to/file
$ ssh-keygen -lF IP_addr
```

P.S. Следует обратить внимание, чтобы права доступа для каталога *~/.ssh* и для приватных ключей в нем, были только для пользователя, который создал их.

***

# ZSH

## Установка ZSH

   ```
   $ sudo apt-get update
   $ sudo apt install zsh
   $ chsh -s $(which zsh)
   ```

## [Oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)

### Устновка Oh-my-zsh

   ```
   $ sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

или

   ```
   sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

### Установка плагинов на примере **Zsh-syntax-highlighting**:

   ```
   git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
   ```

активируем плагин, правим *~/.zshrc* и перезапускаем zsh:

   ```
   plugins=( [plugins...] zsh-syntax-highlighting)
   ```


### Топ плагинов zsh:

+ [ohmyzsh/plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins):
   * [sudo](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/sudo)
   * [copypath](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copypath)
   * [copyfile](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copyfile)
   * [copybuffer](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/copybuffer)
   * [dirhistory](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/dirhistory)
   * [history](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/history)
   * [aliases](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/aliases)
   * [jsontools](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/jsontools)

+ [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)
+ [zsh-syntax-highlighting ](https://github.com/zsh-users/zsh-syntax-highlighting)
+ [zsh-completions](https://github.com/zsh-users/zsh-completions)
+ [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
+ [k](https://github.com/supercrabtree/k)

***

# nftables

## ADDRESS FAMILIES

Свойство тблицы, Определяет тип обрабатываемого пакета:

* ip - ip4 пакеты
* ip6 - ip6 пакеты
* inet - ip4 и ip6 пакеты
* arp - arp пакеты
* bridge - фреймы OSI 2го уровня
* netdev - все пакеты в стеке протоколов(удобно фильтровать весь трафик)


Каждый **ADDRESS FAMILY** содержит так называемые **хуки(hooks)**, которые указывают, когда следует обрабатывать пакет.

Семейство адресов *ip/ip6/inet* содержит следующие хуки(hooks):

* prerouting - для обработки всех пакеттов поступивших в систему
* input - обработка всех локально входящих пакетов
* output - обработка всех локально исходящих пакетов
* forward - пакеты предназначенные для другого хоста
* postrouting - обработка всех исходящих пакетов
* ingress - для обработки всех пакетов, испольуется перед *prerouting*

## RULESET

```
{list|flush} ruleset [family]
```

Ключевое слово для вызова всех правил, цепочек и таблиц, можно уточнить семейство протоколов

Вывод команды *list ruleset* можно передать в команду *nft -f*

## TABLES

**Таблица** - контейнер для цепочек, сетов и других данных. Таблица идентифицируется по ее имени и семейству.

```
{add|create} table [family] TABLE_NAME [{comment user_comment;} {flags 'flag;}]
{list|flush|delete} table [family] TABLE_NAME
```

## CHAINS

Цепочка - контейнер для правил.

```
{add|create} chain [family] TABLE_NAME CHAIN_NAME [{type filter \
                                                    hook input \
                                                    [device *device*]
                                                    priority filter; \
                                                    [policy drop]; \
                                                    [comment *comment;*]}]
rename chain [family] table chain newname
```

## RULES

```
{add | insert} rule [family] table chain [handle handle | index index] statement ... [comment comment]
replace rule [family] table chain handle handle statement ... [comment comment]
delete rule [family] table chain handle handle
```

## nftables разное

Полезные кейворды для поиска по ману:

* PAYLOAD EXPRESSIOS
* META EXPRESSIONS
* CONNTRACK EXPRESSIONS
* VERDICT STATEMENTS
* DATA TYPES
* LIMIT
* LOG

### nftables основные команды

Описание данных, например возможные флаги  tcp(переменная tcp флагов tcp_flag):

```
# nft describe tcp_flag
# nft describe ct_state
```

Просмотр текущих правил:

```
# nft --handle list ruleset
```

Просмотр правил в json формате:

```
#nft -j  list ruleset
```

Сделать дамп текущих настороек в файл:

```
# nft -s list ruleset >> /tmp/nftables
```

Применить конфигурацию из файла:

```
# echo "flush ruleset" > /tmp/nftables 
# nft -f path_to_file
```

#### nft TABLEs

Создать таблицу:

```
# nft add table family_type table_name
```

Просмотретьвсе таблицы:

```
# nft list tables
```

Просмотреть содержимое таблицы:

```
# nft list table family_type table_name
```

Удалить таблицу:

```
# nft delete table family_type table_name
```

Затереть правила в таблице:

```
# nft flush table family_type table_name
```

#### nft CHAINs

Цепочки бывают главные(**base**) и обычные(**general**). Основные цепочки - входная точка пакетов из сетевого стека(при создании цепочки явно указыывается hook). В то время как обычные цепочки служат для сортировки и организации правил про помощи операторов **jump**.

Создание базовой(**base**) цепочки:

```
# nft add chain family_type table_name chain_name '{type chain_type hook hook_type priority prioryty_value; comment "comment"}'
```

Создание обычной(**regular**) цепочки:

```
nft add chain family_type table_name chain_name
```

Просмотр содержимого цепочки:

```
# nft list chain family_type table_name chain_name
```

Редактирование цепочи, просто следует указать пераметр и его новое начение для существующей цепочки('{policy drop;}'):

```
# nft chain family_type table_name chain_name '{policy drop;}'
```

Удаление цепочки:

```
# nft delete chain family_type table_name chain_name
```

Затирание содержимого цепочки:

```
# nft flush chain family_type table_name chain_name
```

#### nft RULEs

Добавить правило после правила с handle_value:

```
# nft add rule family_type table_name chain_name handle handle_value statement 
```

Добавить правило перед правилом с handle_value:

```
# nft insert rule family_type table_name chain_name handle handle_value statement 
```

*statement* - сюда входит выражение(expression) после истинности которого выполняется *verdict statement*(accept, drop, queue, jump chain_name, jump)

Удалить правило

```
# nft delete rule family_type table_name chain_name handle handle_number
```

#### nft SETs

Создать сет:

```
# nft add set family_type table_name set_name '{type set_type; flags flags;}'
```

Добавить элемент в сет:

```
# nft add element family_type table_name set_name {5.6.7.8./32}
```

### Пример настройки фаервола для сервера

1. Разрешим нашему серверу пинговать другие компьютеры и, чтобы его могли пинговать.
2. Разрешим отправлять запросы и получать ответы по протоколу DNS c конкретного DNS сервера.
3. Разрешим автоматически обновлять системные часы по протоколу NTP.
4. Разрешим скачивать обновления с репозиториев, указанных в файлах настройки менеджеров пакетов.
5. Явно отбрасываем входящий трафик, превышающий скоростные ограничения
6. Узлы, занимающиеся подборкой паролей к SSH, будем определять с помощью утилиты fail2ban. Утилита сама будет конфигурировать nftables для блокирования и разблокирования атакующих узлов.

**/etc/nftable.conf**

```
flush ruleset

table inet FIREWALL {

    define ssh_port = 22222

    set ssh_clients {
        type ipv4_addr
        elements = { 10.15.41.250  }
    }

    chain INPUT {

        type filter hook input priority filter; policy drop;

        #Разрешим трафик на лупбек интерфейс
        #counter выводит инфо о количестве пакетов и их размере,
        #которые удовлетворяют данному правилу
        iifname "lo" counter accept

        #Явно запретим трафик с обратным адресом локальной сети не относящегося
        #к адресу лупбек инттерфейса
        ip saddr 127.0.0.0/8 drop

        #Запретим ICMP трафик превышащий лимит более 5 пакетов в секунду
        meta l4proto  icmp limit rate over 5/second drop
        meta l4proto ipv6-icmp limit rate over 5/second drop

        #Явно отбросим новые ssh соединения превышающие предел подключений юолее 10 в минуту
        tcp dport $ssh_port ct state new  limit rate over 10/minute drop

        #Разрешить трафик по установленным соединениям
        ct state established,related accept

        #Разрешим ICMP и IGMP трафик
        meta l4proto ipv6-icmp accept
        meta l4proto icmp accept
        ip protocol igmp accept

        #Разрешим заходить по ssh разрешенным клиентам
        ip saddr @ssh_clients tcp dport $ssh_port accept

        #Откроем доступ к диапазону портов tcp и  со счетчиком
        tcp dport 10000-11000 counter accept
        udp dport {2033, 3091, 80}

        #разрешим пакеты  между временем
        meta hour "12:30"-"12:35" counter accept

        #отбросим пинговые запросы резмером выше 93 байт(размер icmp #header учитывается)
        icmp type echo-request meta length 93-65535 counter drop

        #отбросим icmp пакеты более 10 в минуту, первые 5 пакетов разрешим
        ip protocol icmp limit rate 10/minute burst 5 packets counter drop
    }

    chain FORWARD {
       type filter hook forward priority filter; policy drop;
    }

}

```

***

# Cygwin

[сайт](https://www.cygwin.com/)
[apt-cyg - утилита для администрирования cygwin приложений через командную строку](https://github.com/transcode-open/apt-cyg)


Установка без админ прав использовать ключь **--no-admin**:
```
setup-x86_64.exe --no-admin
```

Обновление установленных приложений:
```
setup-x86_64.exe --no-admin --root .\cygwin64 -q --upgrade-also
```
  --root - ключь, после которого следует указать корневой каталог cygwin

# 







