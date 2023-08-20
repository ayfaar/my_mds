# Ubuntu

Узнать версию системы

```bash
lsb_release -a
```

## Обновление

Узнать требуется ли перезагрузка ПК после обновления

```bash
cat /var/run/reboot-required
```

## Работа с пакетами (packages)

Спиок установленных пользователем пакетов:

```bash
apt list --manual-installed
```

Список файлов принадлежащих приложению

```bash
dpkg --listfiles packagename
```

## Насиройка сети (NETWORK)

Вывести информацию о доступных сетевых интерфейсах

```bash
ip addr
```

Просмотреть порты:

```bash
ss -lentu
```

### netplan

Файл настроек */etc/netplan/файлы-настройки.yaml*

```
network:
    version: 2
    render: networkd
    ethernets:
        ens160:(имя сетевого интерфейса)
            dhcp4: True/No
            addresses:
            - 192.168.100.88/24
            - дополнительный IP
            - еще IP
            gateway4: 192.168.100.1
            namesevers:
                addresses:
                - IP 1
                - IP 2
                search:
                - IP1
                - IP2
                - IP3
```

сохранить и применить конфигурацию сети, если в конфигурационном файле yaml есть ошибки, то система сообщит об этом:

```
netplan apply
```

### proxy server

#### proxy server для apt

*/etc/apt/apt.conf*:

```
Acquire::http::proxy "http://user:pass@x.y.z.251:3128/";
Acquire::http::proxy "https://user:pass@x.y.z.251:3128/";

```

#### proxy server для wget

```
export http_proxy="http://user:pass@x.y.z.251:3128/"
export https_proxy="http://user:pass@x.y.z.251:3128/"
```

 Для постоянного действия настройки следует внести изменения в файл *.bashrc*

```
proxy="http://user:pass@x.y.z.251:3128/"
export http_proxy=$proxy
export https_proxy=$proxy
```

После внесения изменений в файл *.bashrc* следует выполнить команду:

```
source ~/.bashrc
```

Cброс настроек proxy сервера для wget:

```
unset http_proxy
unset https_proxy
```

***

# LUKS Disk Encryption(шифрование диска)

***

# LVM(Logical Volume Manager)

## Подготовка HDD

1. Запускаем *parted* - утилита для рабиения HDD
   + mklabel gpt
      создадим таблицу разделов  gpt
   + p
      печатает на экран информацию о диске
   + mkpart
      + создадим раздел для файлов grub, с флагом *bios_grub*
      + создадим раздел для загрузки /boot (0.5-1GB)
      + оставшееся место создадим раздел для LVM
   + quit
      выход из *parted*

## Создание LVM

1. Укажем физическое устройство, которое будет входить в наш LVM(диск или раздел)
   + *pvcreate* /dev/sda
       данная команда создает метку в начале HDD о том, что HDD будет использован в LVM
   + *pvremove* /dev/sda
       удалить метку о том, что данное устройство входит в LVM
2. Создадим группу томов, состоит из физических устройств, имеющих метку LVM, созданную при выполнении команды *pvcreate*. VG представляет из себя виртуальный жесткий диск, который мы потом разобьем на разделы(Logical Volume) и в который можно добавлят новые HDD или разделы.
   + *vgcreate* VGName /dev/sda
3. Создадим логический радел в ранее созданном VG
   + *lvcreate* -n LVName -L1000MB VGName
   + *lvcreate* -n LVName -l100%FREE VGName
       создаст LV и займет весь свободный объем VG
4. Создадим файловую систему
   + mkfs.ext4 /de/VGName/LVName

## Удаление LVM

1. *lvremove* /dev/VGName/LVName
2. *vgremove* VGName
3. *pvremove* /dev/sda

## Увеличение размера раздела

+ vgs
  проверим сколько свободного места в volumeGrope

+ pvcreate /dev/vda2
  добавим в PV новый HDD

+ lvdisplay
  выводит информацию о имеющихся LV

+ vgdisplay
   выводит инфо о VG, ищем там Free PE

+ vgextend VG-Name /dev/vda2
  расширим размер логической группы за счет HDD или раздела HDD, которые ранее были добавлены в PV

+ *lvextend* -r -L+500MB /dev/VGName/LVName
                  или
 *lvextend* -l +100%FREE /dev/VolGroup00/lv_root
  увеличить размер LV на 500MB. -r - ключ автомматически увеличивает размер файловой системы под размер раздела
+ *resize2fs* /dev/VGName/LVName
      увеличит размер файловой системы ext2, ext3, ext4
+ *xfs_growfs* /dev/VGName/LVName
      увеличит размер файловой системы xfs

## Уменьшение размера раздела

+ xfs не уменьшается
+ передоперацией раздел должен быть отмонтирован, если раздел корневой, то грузимся с лив сд
+ проверим распознала ли система тома LVM
    *vgdisplay*
    *lvdisplay*
+ проверим файловую систему на ошибки
    *e2fsck -f* /dev/VGName/LVName
+ уменьшим раздел
    *lvreduce* -r -L-500M /dev/VGNAme/LVName

## Добавление нового PV в существующую VG

    *vgextend* VGName /dev/sda /dev/sdb ...

## Снапшоты LVM

В снапшоте содержатся данные, которые были на диске в момент создания снапшота
    + Создать снапшот
        *lvcreate -s -n snapshotName -L128M /dev/VGName/LVName*
            -L128M - размер буфера в который буду записываться изменения, произошедшие в файловой системе во время создания снапшота.
            !!!!размер буфера нужно выбрать что бы он не переполнился
            Снапшот создался с возможностью чтения/записи
        *lvcreate -s -p r -n snapshotName -L128M /dev/VGName/LVName*
            Снапшот только для чтения *-p r*
    + созданный снапшот можно смонтировать в нужный каталог

## Перенос данных в границах одной VG на новый HDD

Сначало нужно убедиться хватает ли нам свободного места в нашей VG для переноса данных с диска, который мы хотим заменить, должно быть >= размера данных диска который меняем
    + pvmove /dev/sdb -v
        /dev/sdb - устройство которое мы выкенем и данные с которого хотим перераспределить в VG
    + vgreduce VGName /dev/sdb
        удалить из VGName диск /dev/sdb

## Аналог RAID-0

данные записываются на физические устройства распределенно
    + lvcreate -i2 -I32 -L10G -n LVName VGName
        -i2 - данные будут распределены между двумя жесткими дисками
        -I32 - данные записываются блоками по 32кб на каждый диск поочередно

## Команды для работы с LVM

    1. *vgdispaly*
    2. *pvdisplay*
    3. *lvdisplay*
    4. *lvreduce*
    5. *lvremove* /dev/VGName/LVName
    6. *vgremove* VGName
    7. *pvremove* /dev/sda

***

# Настройка SSH

**~/.ssh/known_hosts**
**~/.ssh/authorized_keys**
**~/.ssh/config**

Генерация host_keys

```
# ssh-keygen -A
```

Вывод информации о подключениях ssh:

```
journalctl -fu ssh
```

или

```
# tail -f /var/log/auth.log
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
ssh-keygen -t ed25519 -C "Comment for key"
```

Желательно задать мастер пароль для закрытого ключа, если он не был задан в процессе создания или нужно изменить, то следует выполнить команду:

```
ssh-keygen -p -f ~/.ssh/key_file
```

Далее при помощи команды *ssh-copy-id* копиуем сгенерированный нами публичный ключ на сервер, с которым мы хотим устанавливать соединение по ключу. Данная команда добавит на новую строку открытый ключ пользователя в файл ***~/.ssh/authorized_keys*(аналогично можно просто скопировать вручную в данный файл открытый ключ):

```
ssh-copy-id -i ~/path/to/key user_name@192.168.100.100
```

или

```
cat ~/.ssh/id_ed25519.pub | ssh user_name@192.168.100.10 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Для того, чтобы постоянно не вводить пароль, то для текущей сесси терминала следует запустить **ssh-agent** и подключить к нему требуемые ключи:

```
eval "$(ssh-agent)"
ssh-add /path/to/key_file
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
ssh-keygen -lf /path/to/file
ssh-keygen -lF IP_addr
```

P.S. Следует обратить внимание, чтобы права доступа для каталога *~/.ssh* и для приватных ключей в нем, были только для пользователя, который создал их.

# Настройка локали (Localisation)

Вывести используемые системой локали

```bash
locale
```

Cгенерировать русскую локализацию ru_RU.UTF-8
доступный локализации можно найти в файле /usr/share/i18n/SUPPORTED
изменеия вступят в силу после перезагрузки ПК

```
locale-gen ru_RU.UTF-8
```

# Настройка времени

Общая информация о времени:

```bash
timedatectl status
```

1) правим файл настроек /etc/systemd/timesyncd.conf

   ```
   [Time]
   NTP=192.168.100.222 192,168.100.1
   FallbackNTP=ntp.ubuntu.com
   ```

2) вкл/выкл обновление времени по сети(если вкл, то вручную поменять время нельзя)

   ```
   timedatectl set-ntp <0>|<1>
   ```

3) Список временных зон

   ```
   timedatectl list-timezones
   ```

4) Задать временную зону:

   ```
   timedatectl set-timezone Europe/Moscow
   ```

5) Перезапустить службу времени

   ```
   systemctl restart systemd-timesyncd
   ```

# Пользователи и группы (users and groups)

Добавить группу:

```
groupadd GROUP_NAME
```

Удалить группу:

```
groupdel GROUP_NAME
```

Найти группы в системе по маске:

```
getent group | grep SEARCH_MASK
```

Список пользователей в системе:

```
# less /etc/passwd
```

Добавить пользователя в группу:

```
usermod -a -G GROUP_NAME USER_NAME
```

Добавить пользователя *user_name* в группу *wheel* и назначит шел по умолчанию *zsh*

```
# useradd -m -G wheel -s /usr/bin/zsh user_name
```

# SAMBA Server

Файл настроек */etc/samba/smb.conf.*

1) Установим samba сервер

   ```
   apt install samba
   ```

2) Делаем бэкап файла настроек:

   ```
   cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
   ```

3) Доступ к папке по логину и паролю.

   Правим файл настроек */etc/samba/smb.conf*

    ```
    [global]
      security = user
      и другие настройки
    [share]
       comment = Staff Folder
       path = /data/staff
       public = no
       writable = yes
       read only = no
       guest ok = no
       valid users = user1 @group10
    ```

        * [share] - share - имя расшареного ресурса, которое следует указывать при подключении
        * path = /data/staff — используем новый путь до папки.
        * public = no — запрещаем публичный доступ.
        * guest ok = no — не разрешаем гостевое подключение.
        * security = user - пользователь должен быть создан в linux, где установлен samba
        * valid users - пользователи или группы(@) пользователей, которым разрешено подключение

4) Добавим существующего пользователя в систему samba

   ```
   smbpasswd -a user_name
   ```

   для удаления пользователя:

   ```
   smbpasswd -x use-name
   ```

5) Перезапустим samba

   ```
   systemctl restart smbd
   ```

# Создание и деплой ман страницы из MarkDown файла(man page create from MarkDown file)

The filename of a MAN page should always be:

   ```
   [PROGRAM NAME].[SECTION NUMBER]
   ```

Для того чтобы узнать какие бывают номера разделов:

   ```
   man man
   ```

Создадим файл *hello.1.md* с содержанием(текст между "---" называется форматером и позволяет вставлять в MarkDown файл дополнительные метаданные):

   ```
   ---
   title: HELLO
   section: 1
   header: User Manual
   footer: hello 1.0.0
   date: January 12, 2022
   ---
   ```

Для создания man страницы используем программу pandoc:

   ```
   pandoc hello.1.md -s -t man -o hello.1
   ```

После чего будет создан файл справки hello.1. Для того чтобы его просмотреть через *man*:

   ```
   man -l hello.1
   ```

Файл справки может включать следующие разделы:

   ```
   CONFIGURATION
   EXIT STATUS
   RETURN VALUE
   ERRORS
   ENVIRONMENT
   FILES
   VERSIONS
   CONFORMING TO
   NOTES
   BUGS
   AUTHORS
   SEE ALSO
   ```

Сжимаем созданный файл справки при помощи Gzip:

   ```
   gzip ~/ManPageDemo/hello.1
   ```

Копируем созданный архив в каталог ман страниц:

   ```
   sudo cp ~/ManPageDemo/hello.1.gz /usr/share/man/man1/
   ```

Обновим БД man страниц:

   ```
   # mandb
   ```

Для удаления man страницы из системы следует удалить соответствующий файл справки и обновить базу man страниц.

## Установка и настройка шрифтов в консоли

## Установим утилиту **fontconfig**

   ```
   sudo apt install fontconfig
   ```

## Установим [Powerline fonts](https://github.com/powerline/fonts)

Шрифты будут скопировыны в каталог **~/.local/share/fonts**

   ```
   git clone https://github.com/powerline/fonts.git --depth=1
   cd ./fonts
   ./install.sh
   cd ..
   rm -fr ./fonts
   ```

Выполним команду:

   ```
   fc-cache -vf
   ```

Проверим установился ли шрифт апример Hack:

   ```
   fc-list  | grep Hack
   ```

# ZSH

## Установка ZSH

   ```
   sudo apt-get update
   sudo apt install zsh
   chsh -s $(which zsh)
   ```

## [Oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)

### Устновка Oh-my-zsh

   ```
   sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

или

   ```
   sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

### Установка плагинов на примере **Zsh-syntax-highlighting**

   ```
   git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
   ```

активируем плагин, правим *~/.zshrc* и перезапускаем zsh:

   ```
   plugins=( [plugins...] zsh-syntax-highlighting)
   ```

### Топ плагинов zsh

+ [ohmyzsh/plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins):
  + [aliases](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/aliases)
  + [alias-finder](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/alias-finder)

+ [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
+ [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
+ [k](https://github.com/supercrabtree/k)
+ [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)

# Запуск разных версий пакетов на примере python

! Замечание: данны пример выполнялся в cygwin

Для этих целей используется программа **update-alternatives**

Определим где находятся исполняемые файлы разных версий python:

   ```
   whereis python
   ```

Посмотрим есть ли альтернативные пакеты python в alternatives:

   ```
   /usr/sbin/update-alternatives --display python
   ```

Если хотим добавить новый интерпритатор то выполним:

   ```
   /usr/sbin/update-alternatives --install /usr/bin/python python /usr/bin/python3 40
   ```

По умолчанию установится альтернатива с наивысшим приоритетом(в нашем примере приоритет = 40)
можно выбрать желаемый интерпритатор:

   ```
   /usr/sbin/update-alternatives --config python
   ```

Для того, чтобы включить автоматический выбор альтернатив:

   ```
   /usr/sbin/update-alternatives --auto python
   ```

# Сеть (Network)

## systemd-networkd

Файлы настроек сетевых интерфейсов находятся в каталоге */etc/systemd/network/* с именем *foo.network*

Создадим файл настроек для сетевой карты *ens33.network* в указанном выше каталоге,(вместо ens33 в названии файла можно использовать любое другое. ens33 - это системное имя сетевого адаптера), со следующим содержимым:

```
[Match]
Name=ens33

[Network]
DHCP=false
Address=10.15.51.99/24
Gateway=10.15.51.1
DNS=10.15.41.30 10.15.41.31
```

Перезапустим systemd-networkd:

```
# systemctl restart systemd-networkd.service
```

Сделаем сервис запускаемым при загрузке ос:

```
# systemctl enable systemd-networkd.service
```

Проверим статус сервиса

```
# systemctl status systemd-networkd.service
# networkctl status
```

## systemd-resolved

Сервис для решения сетевых имен(DNS)

Данный сервис при работе в паре с *systemd-networkd* берет значения из файла настроек systemd-networkd.
Включим данный сервис:

```
# systemctl enable systemd-resolved.service
# systemctl enable systemd-resolved.service
```

Проверим статус сервиса  systemd-resolved.service:

```
# systemctl status systemd-resolved.service
# resolvectl status
```

# Firewall

## ufw

Лог файл **/var/log/ufw.log**

Включить/выключить журнал:

```
sudo ufw logging on[off]
```

Политики брандмауэра UFW находится в файле /etc/default/ufw и могут быть изменены с помощью следующей команды.:

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Вывести статус:

```
sudo ufw status verbose
```

Вывести статус с нумерацией рправил;

```
sudo ufw status numbered
```

Вкл/выкл фаервол:

```
sudo ufw enable[disable]
```

Открыть порт (в данном примере SSH):

```
sudo ufw allow 22
```

Открыть порт для протокола tcp:

```
sudo ufw allow 2222/tcp
```

Правила могут быть добавлены с использованием нумерованного формата:

```
sudo ufw insert 1 allow 80
```

Подобным образом можно закрыть открытый порт:

```
sudo ufw deny 22
```

Для удаления правила используйте delete:

```
sudo ufw delete deny 22
```

Также можно разрешить доступ к порту с определенных компьютеров или сетей. Следующий пример разрешает на этом компьютере доступ по SSH с адреса 192.168.0.2 на любой IP адрес. Замените 192.168.0.2 на 192.168.0.0/24 чтобы разрешить доступ по SSH для всей подсети.:

        sudo ufw allow proto tcp from 192.168.0.2 to any port 22

Разрешим диапазоны портов для UFW:

        sudo ufw allow 5000:5003/tcp
        sudo ufw allow 5000:5003/udp

Разрешим доступ через определенный сетевой интерфейс:

        sudo ufw allow in on eth2 to any port 22

Запретим соединения для UFW:

        sudo ufw deny from 11.12.13.0/24
        sudo ufw deny from 11.12.13.0/24 to any port 80
        sudo ufw deny from 11.12.13.0/24 to any port 443

Удаление правила UFW:

        sudo ufw status numbered
        sudo ufw delete 1

        sudo ufw delete allow 22/tcp

Приложения, которые открывают порты, можно включать в профили ufw, которые детализируют какие порты необходимы этому приложению для корректной работы. Профили содержатся в **/etc/ufw/applications.d**, и могут быть отредактированы, если порты по умолчанию были изменены.
Чтобы посмотреть для каких приложений установлен профиль введите следующую команду в терминале:

        sudo ufw app list

Если вы хотите получить дополнительную информацию о конкретном профиле и определенных правилах, вы можете использовать следующую команду:

        $ sudo ufw app info 'Apache'


        Profile: Apache
        Title: Web Server
        Description: Apache V2 is the next generation f the omnipresent Apache web server.
        Ports:
        80/tcp

Разрешить трафик по порту, используя профиль приложения, можно следующей командой:

        sudo ufw allow 'Apache'

 Также доступен расширенный синтаксис:

        sudo ufw allow from 192.168.0.0/24 to any app Apache

Сброс брандмауэр UFW:

        sudo ufw reset

***

## Cron

[crontab.guru](https://crontab.guru/)

**Шаблон задания для Cron**

```
# |-­-минуты (0-59)
# | |----часы (0-23)
# | | |----день месяца (1-31)
# | | | |-------месяц (1-12)
# | | | | |-----день недели (0-6)
# * * * * *
```

### Часто используемые команды

Отображение содержимого crontab файла:

```
crontab -l
```

Редактирование  crontab файла

```
crontab -e
```

Редактирование  crontab файла пользователя user

```
crontab -u user -e
```

Удалить все задания:

```
crontab -r
```

### Примеры cron заданий

Чтобы выполнять команду каждую минуту, задание должно быть такое:
        * ** ** <исполняемая-команда>
Похожее задание, только команда будет вызываться каждые пять минут:
*/5* ** *<исполняемая-команда>
Вызывать команду 4 раза в час (каждые 15 минут):
*/15 ** ** <исполняемая-команда>
Чтобы выполнить команду каждый час в 30 минут, пишем:
30 ** ** <исполняемая-команда>
Т. е. команда будет выполняться не каждые 30 минут, а тогда, когда значение минут будет равно 30 (например, 10:30, 11:30, 12:30 и т. д.).

Значения времени можно комбинировать, перечислив их через запятую. Следующий код будет выполнять команду три раза в час: в 0, 5 и 10 минут.
0,5,10 ** ** <исполняемая-команда>
Выполнять команду каждый час будет следующее задание:
0 ** ** <исполняемая-команда>
Выполнение команды каждые два часа:
0 */2* ** <исполняемая-команда>
Чтобы выполнять команду каждый день (в 00:00):
0 0 ** *<исполняемая-команда>
Выполнение команды каждый день в 03:00:
0 3* ** <исполняемая-команда>
Выполнение команды каждое воскресенье (sunday):
0 0 ** SUN <исполняемая-команда>
Другой вариант задания, которое будет выполнять команду каждое воскресенье (естественно, тоже в 00:00):
0 0 ** 0 <исполняемая-команда>
Выполнение команды каждый день с понедельника по пятницу:
0 0 ** 1-5 <исполняемая-команда>
Следующее задание будет выполнять команду каждый месяц, 1-го числа в 00:00:
0 0 1 ** <исполняемая-команда>
Выполнять команду в 16:15 каждого первого числа месяца будет это задание:
15 16 1 ** <исполняемая-команда>
Выполнение команды каждые три месяца:
0 0 1 */3* <исполняемая-команда>
Выполнение команды в строго определённое время и месяц:
5 0 *4* <исполняемая-команда>
Задание будет вызывать команду в начале каждого полугодия (в 00:00 1-го дня):
0 0 1 */6* <исполняемая-команда>
Выполнение команды каждый год 1-го января в 00:00:
0 0 1 1 * <исполняемая-команда>

Ещё существуют готовые задания:

@reboot — одиночное выполнение команды при загрузке;
@yearly — раз в год;
@annually — тоже раз в год;
@monthly — раз в месяц;
@weekly — один раз в неделю;
@daily —  раз в день;
@midnight — тоже раз в день;
@hourly — раз в час.

Чтобы выполнять команду каждый раз после перезапуска сервера, используйте это задание:
@reboot <исполняемая-команда>

## NO_PUBKEY ОШИБКА(NO_PUBKEY ERROR)

Иногда при добавлении нового репозитария или истечение срока действия ключа, при попытке обновелния баз пакетов можно получить ошибку **NO_PUBKEY 467B942D3A79BD29**.

В системе информация об открытых ключах хранится в:

+ файл **/etc/apt/trusted.gpg** - в нем могу содержаться несколько открытых ключей, от разных приложений.
+ файлы с расширение **gpg** в каталоге **/etc/apt/trusted.gpg.d/**

Публичные ключи хранятся на публичных серверах общественных ключей или их можно получить на сайте разработчика, устанавливаемого програмного обеспечения:

+ [http://keyserver.ubuntu.com/](http://keyserver.ubuntu.com/)
+ [https://pgp.mit.edu/](https://pgp.mit.edu/)
+ [https://pgp.mit.edu/](https://pgp.mit.edu/)

Для того чтобы добавить в систему требуемый ключ, выполним команду(для ubuntu сервера):

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --keyserver-options http-proxy=http://10.15.41.25:3128 --recv-keys 467B942D3A79BD29
```

# SOFT LIST

1) htop
2) tmux
