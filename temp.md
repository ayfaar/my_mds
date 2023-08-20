# Arch Linux

## Установка

[Загрузим свежий образ Arch Linux](https://archlinux.org/download/)

Определим режим загрузки(BIOS, EFI)
Если команда отработает без ошибок, то режим загрузки EFI

```bash
ls /sys/firmware/efi/efivars
```

По умолчанию Arch запускает сам SSH сервер, проверим включен ли он:

```bash
systemctl status sshd.service
```

Для того, чтобы в дальнейшем при установке можно было бы подключться к компьютеру
по SSH следует задать пароль для суперпользователя:

```bash
passwd
```

## Настройка сети

### Настройка статическоо IP

Проверим существующие сетевые адаптеры и их состония(вкл или выкл)

```bash
ip link
```

Проверим присвоены ли какие IP адреса сетевым адаптерам:

```bash
ip addr show
```

Удалим, если требуется ненужный IP(может быть получен по DHCP):

```bash
ip addr del <interface-IP-address>/<network_mask> dev <interface_name>
```

Зададим интерфейсу требуемый адрес:

```bash
ip addr add <interface-IP-address><network_mask> dev <interface_name>
```

### Настройка маршрутизатора по умолчанию

Проверим какие существуют сетевые ммаршруты:

```bash
ip route show
```

Удалить ненужные маршруты можно командой:

```bash
ip route del <IP_addr>/<net_mask> dev <interface_name>
```

Настроим маршрутизатор по молчанию:

```bash
ip route add <gateway_IP> dev <interface_name>
ip route add <network_addre>/<net_mask> via <gateway_IP>
```

  где, <network_addre>/<net_mask> подставим слово **default**, что эквивалентно IPv4 сеи 0/0

### Настройка proxy

```bash
export http_proxy=http://IP:3128
export https_proxy=http://IP:3128
```

## Настройка времени (Update the system clock)

Если нужно добавить другой сервер времени для синхронизации, правим файл **/etc/systemd/timesyncd.conf**.

Включим синхронизацию времени по сети:

```bash
timedatectl set-ntp true
```

Проверим состояние службы синхронизации времени:

```bash
timedatectl status
systemctl status systemd-timesyncd.service
```

После чего следует перезапустить сервис синхронизации:

```bash
systemctl restart systemd-timesyncd.service
```

## Разметка диска

Узнаем какие диски у нас есть:

```bash
lsblk
```

### Для систем с UEFI и gpt разметкой

Отформатируем при помощи **cgdisk** наш HDD следующим образом:

  1) 100MB EFI partition # Hex code EF00
  2) 1024MB Boot partition # Hex code 8300
  3) 100% size partiton # (to be encrypted) Hex code 8e00(LVM)

Отформатируем EFI и Boot разделы:

```bash
mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

### Для систем с BIOS и gpt разметкой диска

1. Запустим **parted** для разметки диска:

```bash
parted /dev/sda
```

2. Создадим таблицу разделов *gpt*

```bash
mklabel gpt
```

3. Создадим раздел для **grub_bios**(не менее 1MiB) и активируем его

```bash
mkpart bios_grub 1MiB 2Mib
set 1 bios_grub on
```

4. Все остальное место на диске оставим для LVM

```bash
mkpart LVM 2MiB 100%
set 2 lvm on
```

### Шифрование(LVM on LUKS)

[LVM on LUKS](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS)

Оценим какой алгоритм шифрования испоьзовать с точки зрения производительности

```bash
cryptsetup benchmark
```

Зашифруем созданный ранее раздел, содав LUKS контейнер

```bash
cryptsetup --cipher aes-xts-plain64 --key-size 512 luksFormat /dev/sda3
```

Подключим зашифрованный раздел, после выполнения данной команды в появится блочное устройство **/dev/mapper/luks**:

```bash
crytsetup open /dev/sda3/ luks
```

luks - имя LUKS устройства.

Создадим разделы при промощи LVM:

```bash
pvcreate /dev/mapper/luks
vgcreate systemVG /dev/mapper/luks
lvcreate --size 4G systemVG --name swap
lvcreate --size 40G systemVG --name root
lvcreate --size 10G systemVG --name usr
lvcreate --size 10G systemVG --name tmp
lvcreate --size 20G systemVG --name var
lvcreate -l +50%FREE systemVG --name home
```

Отформатируем созданные разделы:

```bash
mkswap /dev/mapper/systemVG-swap
mkfs.ext4 /dev/mapper/systemVG-home
mkfs.ext4 /dev/mapper/systemVG-root
mkfs.ext4 /dev/mapper/systemVG-usr
mkfs.ext4 /dev/mapper/systemVG-tmp
mkfs.ext4 /dev/mapper/systemVG-var
```

Смонтируем разделы и включим свап

```bash
swapon /dev/mapper/systemVG-swap
mount /dev/mapper/systemVG-root /mnt
mount --mkdir /dev/sda2 /mnt/boot
mount --mkdir /dev/sda1 /mnt/boot/efi
mount --mkdir /dev/mapper/systemVG-usr /mnt/usr
mount --mkdir /dev/mapper/systemVG-tmp /mnt/tmp
mount --mkdir /dev/mapper/systemVG-var /mnt/var
mount --mkdir /dev/mapper/systemVG-home /mnt/home
```

## Установка системы

```bash
pacstrap /mnt base base-devel linux lvm2 linux-firmware grub-efi-x86_64 zsh vim nano git openssh efibootmgr dialog
```

linux-firmware можно не устанавливать, если ставим систему на виртуальную машину.

## Настройка системы

### Генерирование fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

```bash
arch-chroot /mnt
```

### Time zone

```bash
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc -u
```

### Локализация

Отредактируем файл **/etc/locale.gen**, раскоментировав нужные локали и запустим:

```bash
locale-gen
```

Создадим файл */etc/locale.conf* и добавим в него нужную нам системную локаль:

```al
LANG=en_US.UTF-8
```

Настроим шрифт и раскладку клавиатуры(раскладки находятся в каталоге */usr/share/kbd/keymaps/*, */usr/share/kbd/consolefonts/*), правим файл **/etc/vconsole.conf**:

```al
FONT=UniCyr_8x16
KEYMAP=ruwin_alt_sh-UTF-8
```

### Настройка сети

#### Зададим имя хоста

```bash
echo "r2d2" >> /etc/hostname
```

### initramfs

Правим файл **/etc/mkinitcpio.conf**:

```text
MODULES=(ext4)
HOOKS=(base udev ... block encrypt lvm2 filesystems)
```

```bash
mkinitcpio -P
```

### Root password

```bash
passwd
```

### Установка загрузчика

Установим grub:

```bash
grub-install /dev/sda
```

Правим файл **/etc/default/grub**:

```al
GRUB_PRELOAD_MODULES="... lvm"
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sdX3:luks:allow-discards"
```

Запустим команду:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Выход и перезагрузка

Выходим из системы и перезагружаемся:

```bash
exit
umount -R /mnt
swapoff -a
```

***

## Настройка после установки

### Настройка сети после установки

### Настройка proxy после установки

```bash
export http_proxy=http://IP:3128
export https_proxy=http://IP:3128
```

#### proxy через sudo

Переменные окружения пользователя доступны только для пользователя и при использовании sudo они теряются, чтобы избежать этого добавим следующую строку в файл **/etc/sudoers.d/05_proxy** или вместе с sudo использовать ключь "-E"

```al
Defaults env_keep += "*_proxy *_PROXY"
```

#### curl and pacman

Можно задать переменную окружения **all_proxy** для работы pacman и curl(используется pacman). Зададим данную переменную постоянно и для всей систнмы, для этого правим файл **/etc/environment**:

```al
all_proxy="socks5://your.proxy:1080"
```

### systemd-networkd

Файлы настроек сетевых интерфейсов находятся в каталоге */etc/systemd/network/* с именем *foo.network*

Создадим файл настроек для сетевой карты *ens33.network* в указанном выше каталоге,(вместо ens33 в названии файла можно использовать любое другое. ens33 - это системное имя сетевого адаптера), со следующим содержимым:

```al
[Match]
Name=ens33

[Network]
DHCP=false
Address=10.15.51.99/24
Gateway=10.15.51.1
DNS=10.15.41.30 10.15.41.31
```

Перезапустим systemd-networkd:

```bash
systemctl restart systemd-networkd.service
```

Сделаем сервис запускаемым при загрузке ос:

```bash
systemctl enable systemd-networkd.service
```

Проверим статус сервиса

```bash
systemctl status systemd-networkd.service
networkctl status
```

### systemd-resolved

Сервис для решения сетевых имен(DNS)

Данный сервис при работе в паре с *systemd-networkd* берет значения из файла настроек systemd-networkd.
Включим данный сервис:

```bash
systemctl enable systemd-resolved.service
```

Проверим статус сервиса  systemd-resolved.service:

```bash
systemctl status systemd-resolved.service
resolvectl status
systemd-resolve --status
```

Отключим протокол LLMNR (Link-local Multicast Name Resolution)(используется для того, чтобы резолвить имена хостов, находящихся в той же подсети, что и хостпосылающий пакет. -порт 5355)
Глобальное отключение, ghfdbv afqk **/etc/systemd/resolved.conf**:

### /etc/hosts

### Настройка времени после установки (Update the system clock)

Если нужно добавить другой сервер времени для синхронизации, правим файл **/etc/systemd/timesyncd.conf**.

Включим синхронизацию времени по сети:

```bash
timedatectl set-ntp true
```

Проверим состояние службы синхронизации времени:

```bash
timedatectl status
systemctl status systemd-timesyncd.service
```

После чего следует перезапустить сервис синхронизации:

```bash
systemctl restart systemd-timesyncd.service
```

## Обновимся

```bash
# pacman -Syyu
```

## Добавим пользователя

```bash
# useradd -m -G wheel -s /usr/bin/zsh user_name
# passwd user_name
```

Добавим группу wheel в **/etc/sudoers**

```bash
visudo /etc/sudoers
```

```bash
%wheel ALL=(ALL:ALL) ALL
```

## Настроим SSH

Сгенерируем если требуется ключи хоста(по умолчанию находятся в **/etc/ssh**)

```bash
# ssh-keygen -A
```

Сгенерируем пару ключей для дальнейшего подключения к данному хосту:

```bash
# ssh-keygen -t ed25519 -C "Comment"
```

Скопируем сгенерированный ключ на сервер к которому будем подключаться:

```bash
# ssh-copy-id -i ~/.ssh/key_file user@host
```

Правим файл **/etc/ssh/sshd_config**:

```bash
AllowUsers userName
Port 22
HostKey /etc/ssh/ssh_host_ed25519
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

## Настроим zsh

Установим ohmyzsh и основные плагины смотри *app.md*

## Настройки безопасности

### Заблокируем аккаунт root

```bash
# passwd -l user_name_root
```

для разблокировки использовать ключ "-u"

### microcode (если это не виртуальная машина)

```bash
# pacman -S amd-ucode intel-ucode
# grub-mkconfig -o /boot/grub/grub.cfg
```

### права для вновь созданных файлов

Для изменения прав доступа создаваемых пользователем файла следует задать **umask** 077, что соответствет разрешению читать файлы и папки только польователю создавшего его. На глобальном уровне данная настройка делается в файле */etc/profile*, на пользовательском - в конфигурационном файле оболочки(в нашем случае .zshrc)

```bash
# umask 077
```

Посмотреть текущую маску в удобном виде:

```bash
# umask -S
```

### Разрешим только группе wheel использовать su для подключения к root аккаунте

Правим файл */etc/pam.d/su* и */etc/pam.d/su-l*, раскоментировав строку:

```al
auth required pam_wheel.so use_uid
```

### Ограничим возможности подключений польователей используя PAM, правим файл **/etc/security/access.conf**

```al
+:user_name:LOCAL
+:user_name: IP_ADDDR_1 IP_ADDR_2 IP_ADDR_3
-:ALL:ALL
```

Здесь пользователю user_name разрешено локальное подключение и удаленное с трех указанных IP, остальным доступ закрыт

***

## Pacman

Настройка списка серверов обновления, выбранных для России и Британии , по протоколу https отсортированных по скорости(требуется пакет **pacman-contrib**)

```bash
#  curl -s "https://archlinux.org/mirrorlist/?country=FR&country=GB&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 7 - | sudo  tee  /etc/pacman.d/mirrorlist
```

## Обновление системы

```bash
pacman -Syyu
```

Если, обновиться получается только по рутом, то под пользователем следует проверить следующую команду:

```bash
sudo -E pacman -Syyu
```

## Удаление пакетов

```bash
pacman -R package_name
```

с зависимостями, которые не используются другими пакетами:

```bash
pacman -Rs package_name
```

## Поиск пакетов

Установленных:

```bash
pacman -Qs string1 string2 ...
```

В репозиториях

```bash
pacman -Ss string1 string2 ...
```

## Список файлов установленного пакетов

```bash
pacman -Ql package_name
```

## Какому пакету принадлежит файл

```bash
pacman -Qo /path/to/file_name
```

## Список явно  установленных пакетов

```bash
pacman -Qe
```

## Список последних 20 установленных пакетов

```bash
expac --timefmt='%Y-%m-%d %T' '%l\t%n' | sort | tail -n 20
```

***

* openssh
* pacman-contrib
* man-db
* python2
* python
* zsh
* xclip
* xsel
* nftables
* bind(nslookup) - работа с DNS
