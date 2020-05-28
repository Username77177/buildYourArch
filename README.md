# Настройка Arch Linux

Так как я довольно давно уже сижу на Arch Linux, я решил поделиться моими конфигами и настроками.

{TOC}
- [Настройка Arch Linux](#-arch-linux)
  - [Как я устанавливаю Arch](#---arch)
    - [Основные действия](#-)
      - [1. Подключюсь к WiFi](#1---wifi)
      - [2. Создаю разделы на диске](#2----)
      - [3. Форматирование разделов](#3--)
      - [4. Монтирую разделы](#4--)
      - [5. Качаю пакеты в раздел root](#5-----root)
    - [Дополнительные действия](#-)
      - [6.1. Добавить нового юзера](#61---)
      - [6.2. Добавить нового пользователя в группу *wheel*](#62------wheel)
        - [Качаем `sudo`](#-sudo)
        - [Добавляем пользователя в группу wheel](#----wheel)
      - [6.3. Устанавливаю локаль](#63--)
      - [6.4. Установка времени](#64--)
      - [6.5. Устанавливаю X](#65--x)
      - [6.6. Устанавливаю KDE + пакеты](#66--kde--)
      - [6.7. Устанавливаю GRUB](#67--grub)
    - [7. Перезагружаю Arch](#7--arch)
  - [Первоначальная настройка](#-)
    - [Тачпад](#)
    - [Xresources](#xresources)
    - [Локали](#)
    - [tty](#tty)
  - [Установка основного софта](#--)
    - [Приверженник Flatpak](#-flatpak)
    - [GUI-приложения](#gui-)
    - [CLI-приложения](#cli-)
  - [Конфигурация](#)
    - [Neovim](#neovim)
    - [TaskWarrior](#taskwarrior)
    - [Kitty](#kitty)
    - [Fish + OhMyFish](#fish--ohmyfish)
    - [Cmus](#cmus)
    - [Zathura](#zathura)
    - [i3-gaps](#i3-gaps)
    - [tmux](#tmux)

## Как я устанавливаю Arch

Данная установка подходит для моего ПК, однако для вашего может не подойти. Делайте всё на свой страх и риск. Конфигурации должны работать одинаково у всех.

### Основные действия

Чтобы установить диск, я выполняю следующие действия:

1. Подключаюсь к WiFi
2. Создаю разделы на диске
3. Форматирую разделы на диске
4. Монтирую разделы на диске
5. Качаю основные пакеты на раздел с **root**'ом
6. Твикаю рут
   1. Добавить нового пользователя
   2. Добавить пользователя в группу wheel
   3. Устанавливаю локаль
   4. Устанавливаю время
   5. Устанавливаю X
   6. Устанавливаю KDE
   7. Устанавливаю GRUB2
7. Перезагрузить Arch

#### 1. Подключюсь к WiFi

```bash
wifi-menu # Таким образом сразу после запуска я подключаюсь к wifi
```

#### 2. Создаю разделы на диске

```bash
cfdisk # Некоторые используют cfdisk, однако я по рекомендации товарища использую fdisk, так как он не химичит с первыми байтами в разделах. Раньше использовал cfdisk из-за TUI

fdisk /dev/sda
Welcome to fdisk (util-linux 2.35.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

n

Partition number (8-128, default 1): (Нажимаю Enter)
First sector (1226770432-1228802047, default 1226770432): (Нажимаю Enter)
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1226770432-1228802047, default 1228802047): +(Количество GB для рут)G

n

Partition number (8-128, default 1): (Нажимаю Enter)
First sector (1226770432-1228802047, default 1226770432): (Нажимаю Enter)
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1226770432-1228802047, default 1228802047): +(Количество GB для домашнего раздела)G

n

Partition number (8-128, default 1): (Нажимаю Enter)
First sector (1226770432-1228802047, default 1226770432): (Нажимаю Enter)
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1226770432-1228802047, default 1228802047): +(Количество GB для раздела поддержки)G

w
```

#### 3. Форматирование разделов

```bash
mkfs.ext4 /dev/sda1 # Форматирую диск для рута
mkfs.ext4 /dev/sda2 # Форматирую диск для домашнего раздела
mkswap /dev/sda3 # Форматирую диск для подкачки
swapon /dev/sda3 # Включаю подкачку
```

#### 4. Монтирую разделы

```bash
mount /dev/sda1 /mnt # Монтирую рут
mkdir -p /mnt/home # Создаю папку для домашнего раздела
mount dev/sda2 /mnt/home # Монтирую домашний раздел
cp install.txt /mnt/ # Копирую гайд с установкой на всякий случай
```

#### 5. Качаю пакеты в раздел root

```bash
pacstrap /mnt linux linux-firmware base base-devel
```

### Дополнительные действия

По принципу система уже функционирует, однако этого мало, для того чтобы её использовать постоянно

#### 6.1. Добавить нового юзера

```bash
useradd (ваш никнейм)
passwd (ваш никнейм)
```



#### 6.2. Добавить нового пользователя в группу *wheel*

##### Качаем `sudo`

```bash
pacman -Sy sudo vi && visudo # Редактируем visudo по надобности
```

Я всегда убираю комментарий со строки

```visudo
## Same thing without a password
%wheel ALL=(ALL) NOPASSWD: ALL
```

*Оно позволяет использовать sudo без пароля постоянно*

##### Добавляем пользователя в группу wheel

```bash
groupmems -g wheel -a (ваш никнейм)
```



#### 6.3. Устанавливаю локаль

В `/etc/locale.gen` удаляю символ `#` из строки  **#en_US.UTF-8**, затем ввожу:

```bash
locale-gen
```

А потом ввожу: `locale > /etc/locale.conf`

Данные действия просто записывают настройки локали с систему. `locale > /etc/locale.conf` перезаписывает дефолтные локали на новые.



#### 6.4. Установка времени

Линкую файлик с нужным мне регионом к /etc/localtime

```bash
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hw-clock --sys-to-hc # Синхронизирую время
```



#### 6.5. Устанавливаю X

```bash
sudo pacman -Syy xorg xclip xorg-xinit
```



#### 6.6. Устанавливаю KDE + пакеты

```bash
sudo pacman -Sy plasma firefox grub neovim cmake dolphin ark kitty
```



#### 6.7. Устанавливаю GRUB

```bash
sudo pacman -Sy grub
sudo pacman -S linux # На всякий случай переустанавливаю ядро
# У меня система с UEFI, поэтому я примонтирую с UEFI от Windows 10 и установлю grub туда
mkdir -p /boot/efi && mount /dev/sda2 /boot/efi
grub-install --target=x86_64-efi --boot-directory=/boot/efi
```

Твикаем конфигурацию **GRUB**, чтобы *не* видеть splashscreen (загрузочный экран). Удаляем `quiet splash` из `/etc/default/grub`

```bash
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3"
```

 Затем обновляем конфигурацию **GRUB**:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```



### 7. Перезагружаю Arch

Нажимаем `Ctrl + D` и пишем `reboot`.

## Первоначальная настройка

### Тачпад

Мне всегда не нравится как тачпад ведёт себя в первоначальной настройке линукс. Мне нравится когда я тяну тачпад вверх, а страницы будто идут за пальцами (инвертированные настройки по дефолту), поэтому я создал такие:

Путь: `/usr/share/X11/xorg.conf.d/40-libinput.conf`

```bash
# Match on all types of devices but joysticks
Section "InputClass"
        Identifier "libinput pointer catchall"
        MatchIsPointer "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
EndSection

Section "InputClass"
        Identifier "libinput keyboard catchall"
        MatchIsKeyboard "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
EndSection

Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Option "ClickMethod" "clickfinger"
        Option "NaturalScrolling" "true"
        Option "Tapping" "True"
        Driver "libinput"

EndSection

Section "InputClass"
        Identifier "libinput touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
EndSection

Section "InputClass"
        Identifier "libinput tablet catchall"
        MatchIsTablet "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
EndSection
```

### Xresources

Далее я настраиваю Xresources. Во многих случаях люди не применяют *Xterm* (как и я, собственно, однако, какие-то минимальные настройки у него быть должны)

Путь: `~/.Xresources`. После того, как измените `Xresources` выполните `xrdb ~/.Xresources`

```bash
! Ctrl + / Ctrl - для того, чтобы изменить размер шрифта
! Thanks, https://askubuntu.com/questions/161652/how-to-change-the-default-font-size-of-xterm 
XTerm.vt100.translations: #override \n\
  Ctrl <Key> minus: smaller-vt-font() \n\
  Ctrl <Key> plus: larger-vt-font()
! Ctrl + C и Ctrl + V
xterm*translations: #override \
    Ctrl Shift <KeyPress> C: copy-selection(CLIPBOARD) \n\
    Ctrl Shift <KeyPress> V: insert-selection(CLIPBOARD)
! При выделении текста мышью, он переносится в буфер обмена
XTerm*selectToClipboard: true

! Шрифт
xterm*faceName: Fira Code
xterm*faceSize: 15

! Dracula Xresources palette https://github.com/dracula/xresources
*.foreground: #F8F8F2
*.background: #282A36
*.color0:     #000000
*.color8:     #4D4D4D
*.color1:     #FF5555
*.color9:     #FF6E67
*.color2:     #50FA7B
*.color10:    #5AF78E
*.color3:     #F1FA8C
*.color11:    #F4F99D
*.color4:     #BD93F9
*.color12:    #CAA9FA
*.color5:     #FF79C6
*.color13:    #FF92D0
*.color6:     #8BE9FD
*.color14:    #9AEDFE
*.color7:     #BFBFBF
*.color15:    #E6E6E6
```

### Локали

Обычно, после того, как я установлю систему я выполняю следующую команду: `locale > /etc/locale.conf`, для того, чтобы убедиться, что все локали стали ровно и система при запуске будет хорошо работать.

### tty

Для того чтобы работать в tty (консоль без GUI на `Ctrl + Alt + F(1-12)`)  мне нужна приятная цветовая схема, а также поддержка кириллицы, для этого я написал маленький скрипт, который нужно положить в `~/.bashrc` или `~/.zshrc`, или в `~/.config/fish/init.fish`

Для *bash или zsh*:

```bash
 # Theme for tty
if [[ $TERM = "linux" ]]
then
	# Cyrillic Font
	setfont /usr/share/kbd/consolefonts/Cyr_a8x16.psfu.gz
	# Colors
    echo -en "\e]P0282C34" #black
    echo -en "\e]P1E06C75" #darkred
    echo -en "\e]P298C379" #darkgreen
    echo -en "\e]P3D19A66" #darkyellow
    echo -en "\e]P461AFEF" #darkblue
    echo -en "\e]P5C678DD" #darkmagenta
    echo -en "\e]P656B6C2" #darkcyan
    echo -en "\e]P7ABB2BF" #lightgrey
    echo -en "\e]P8ABB2BF" #darkgrey
    echo -en "\e]P9E06C75" #red
    echo -en "\e]PA98C379" #green
    echo -en "\e]PBD19A66" #yellow
    echo -en "\e]PC61AFEF" #blue
    echo -en "\e]PDC678DD" #magenta
    echo -en "\e]PE56B6C2" #cyan
    
    echo -en "\e]PFFFFFFF" #white
    clear #for background artifacting
  fi
```

Для fish:

```bash
 # Theme for tty
if [ $TERM = "linux" ]
	# Cyrillic Font
	setfont /usr/share/kbd/consolefonts/Cyr_a8x16.psfu.gz
	# Colors
    echo -en "\e]P0282C34" #black
    echo -en "\e]P1E06C75" #darkred
    echo -en "\e]P298C379" #darkgreen
    echo -en "\e]P3D19A66" #darkyellow
    echo -en "\e]P461AFEF" #darkblue
    echo -en "\e]P5C678DD" #darkmagenta
    echo -en "\e]P656B6C2" #darkcyan
    echo -en "\e]P7ABB2BF" #lightgrey
    echo -en "\e]P8ABB2BF" #darkgrey
    echo -en "\e]P9E06C75" #red
    echo -en "\e]PA98C379" #green
    echo -en "\e]PBD19A66" #yellow
    echo -en "\e]PC61AFEF" #blue
    echo -en "\e]PDC678DD" #magenta
    echo -en "\e]PE56B6C2" #cyan
    
    echo -en "\e]PFFFFFFF" #white
    clear #for background artifacting
  end
  
```

## Установка основного софта

Внизу перечислен список утилит, которые я обычно устанавливаю

`sudo pacman -Syy $(file_with_utilities.package)` - для того чтобы установить утилиты из файла  `file_with_utilities.package`

```bash
adobe-source-code-pro-fonts
albert
alsa-firmware
alsa-lib
alsa-oss
alsa-plugins
alsa-tools
alsa-topology-conf
alsa-ucm-conf
alsa-utils
ark
bash
bash-completion
bc
bluez
bluez-libs
bluez-qt
bluez-tools
bzip2
cantarell-fonts
clang
cmake
cmake
cmus
code
curl
dolphin
firefox
fish
flatpak
fuse-common
fuse2
fzf
gcc
gcc-libs
gd
gdb
gdb-common
gdbm
graphviz
gwenview
gzip
htop
jre-openjdk
kimageformats
kitty
libreoffice-fresh-ru
linux
linux-api-headers
linux-firmware
make
man-db
neofetch
neofetch
neon
okular
os-prober
pandoc
plasma-desktop
pulseaudio
pulseaudio-bluetooth
python
python-pynvim
qt5-base
qt5-tools
qtcreator
ranger
sddm
speedcrunch 
tar
taskwarrior
ttf-dejavu
ttf-hack
ttf-lato
ttf-opensans
ttf-roboto
umbrello
usbutils
vi
vlc
xcb-proto
xcb-util
xcb-util-cursor
xcb-util-image
xcb-util-keysyms
xcb-util-renderutil
xcb-util-wm
xclip
xclip
xdg-dbus-proxy
xdg-desktop-portal
xdg-desktop-portal-kde
xdg-utils
xf86-input-libinput
xf86-video-vesa
xfsprogs
xorg-bdftopcf
xorg-docs
xorg-font-util
xorg-font-utils
xorg-fonts-encodings
xorg-iceauth
xorg-luit
xorg-mkfontscale
xorg-server
xorg-server-common
xorg-server-devel
xorg-server-xephyr
xorg-server-xnest
xorg-server-xvfb
xorg-server-xwayland
xorg-sessreg
xorg-setxkbmap
xorg-smproxy
xorg-util-macros
xorg-x11perf
xorg-xauth
xorg-xbacklight
xorg-xcmsdb
xorg-xcursorgen
xorg-xdpyinfo
xorg-xdriinfo
xorg-xev
xorg-xgamma
xorg-xhost
xorg-xinit
xorg-xinput
xorg-xkbcomp
xorg-xkbevd
xorg-xkbutils
xorg-xkill
xorg-xlsatoms
xorg-xlsclients
xorg-xmessage
xorg-xmodmap
xorg-xpr
xorg-xprop
xorg-xrandr
xorg-xrdb
xorg-xrefresh
xorg-xset
xorg-xsetroot
xorg-xvinfo
xorg-xwd
xorg-xwininfo
xorg-xwud
xorgproto
xterm
xvidcore
zip
```

Это мой минимум для комфортной работы на ПК с KDE. У вас может быть свой, однако эти пакеты, как я считаю действительно являются минимумом.

### Приверженник Flatpak

Должен сказать, что мне больше нравится *Flatpak*, нежели *Snap*. У *Snap* дыры в безопасности, а также он сильно нагружает систему. *Flatpak* не сильно нагружает систему (по крайней мере время запуска стало намного меньше), но жрёт больше постоянной памяти (он докачивает целые среды, чтобы эмулировать там программы, однако они никак не влияют на внешнюю систему).

Так как не все пакеты из *Snap*'а есть в *Flatpak*, я также использую **AUR** (*Arch User Repository*). Для того чтобы использовать его из консоли мне нужен **Yay** (*Yet another yogurt*).

Качаем и устанавливаем Flatpak:

```bash
sudo pacman -Sy flatpak
#Добавляем основной репозиторий
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Качаем и устанавливаем yay:

```bash
sudo pacman -Sy git sudo go make gcc-go base-devel
mkdir ~/.yay
cd ~/.yay
git clone https://aur.archlinux.org/yay.git ./
makepkg -si
cd ~ && rm .yay
```



### GUI-приложения

Я использую следующий сет:

* Для написания документов я использую ровно тот же софт, что и для написания заметок: **Typora**

* Вместо обычного терминала, который используется в KDE, я использую терминал, который использует грфическое ускорение (GPU-Accelerated Terminal): **Kitty**

* Для диаграмм (к которым я часто прибегаю, ибо часто пишу код) я использую **Draw.io**

* Для того, чтобы смотреть документацию к разным ЯП оффлайн, я использую **Zeal** и **QuickDocs**
* Вторым редактором (после **Neovim**) у меня стоит **Code OSS**, версия VSCode которая избавлена от передачи метаданных
* Третьим редактором (который я использую по умолчанию в системе, потому что он её не нагружает и запускается быстро) я использую **Sublime Text 3**
* Для того, чтобы иметь быстрый доступ ко всем функциям на ПК я предпочитаю быструю панель **Albert**
* Также, мне нужны IDE. Я предпочитаю IDE от JetBrains: **WebStorm**, **CLion**, **PyCharm**, **IDEA**, **Rider**

* Калькулятор для больших расчётов - **SpeedCrunch**
* Мессенджер: **Telegram**

### CLI-приложения

* Как основной редактор: **NeoVim**. Позже я опишу его плагины, а также другие конфигурации
* Для расписания и ToDO-list'а я беру **TaskWarrior**
* Если что-то пойдет не так, и мне придётся пользоваться консольным файловым менеджером, то я предпочитаю **Ranger**
* Как основной шелл, я беру **fish**. Он удобный быстрый и гибконастраиваемый, а также в нём прекрасная система подсказок
* **Dropbox** - облако, которое я использую постоянно

## Конфигурация

Тут я приведу абсолютно всю конфигурацию, которую я использую:

### Neovim

Чтобы установить плагины в **Neovim**, нужно установить *Plug*:

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

Также, нужно будет установить зависимости, которые в будущем будут нужны для плагинов:

```bash
sudo pacman -Sy python-pynvim python-pip clang npm
```

Так выглядит init.vim, который лежит в `~/.config/nvim/`

```vim
source ~/.config/nvim/big_build/plugins.vim
source ~/.config/nvim/big_build/plugin_configs.vim
source ~/.config/nvim/big_build/coc_settings.vim
"# Binds
" Комбинация клавиш jkl - действует как Escape в режиме Insert
imap jkl <ESC>

" Апгрейд линейки
set relativenumber

" Комбинация клавиш jks - действует как Escape + сохранение 
" в режиме вставки
imap jks <ESC>:w<CR>

" Передвижение между окнами Ctrl+hjkl
map <C-h> <C-w>h
map <C-j> <C-w>j
map <C-k> <C-w>k
map <C-l> <C-w>l

" F2 чтобы выходить из PasteMode (мод для вставки текста)
set pastetoggle=<F2>

" <leader> теперь пробел
let mapleader = " "

" Дублировать все виды кавычек и скобок
inoremap ' ''<left>
inoremap " ""<left>
inoremap [ []<left>
inoremap { {}<left>
inoremap ( ()<left>
inoremap {<CR> {<CR>}<ESC>O
inoremap {;<CR> {<CR>};<ESC>O

" В нормальном режиме Space+f вызывает :NERDTree
" Ctrl+c - комментирует строки
nmap <leader>f :NERDTreeToggle<CR>
vmap <C-c> <plug>NERDCommenterToggle
nmap <C-c> <plug>NERDCommenterToggle

nmap <leader>z :Goyo 150<CR>


""""PLUGINS BINDS""""

""" ALE
" Навигация между замеченными ошибками
nmap <silent> <leader>j <Plug>(ale_next_wrap)
nmap <silent> <leader>k <Plug>(ale_previous_wrap)

""" Emmet
" Активировать дополнение Emmet на Tab+e
" Использовать Emmet (автодополнить код)
let g:user_emmet_expandabbr_key = '<tab>e'

""" MarkDown
nmap <leader>m :InstantMarkdownPreview<CR>

""" FZF
nmap <leader>ff :FZF<CR>

""" 

""" UltiSnips
let g:UltiSnipsJumpForwardTrigger = "<C-j>"
let g:UltiSnipsJumpBackwardTrigger = "<C-n>"
let g:UltiSnipsExpandTrigger = "<tab>"

"""""

"# UI config

" Делаем Vim цветным ( с темой и синтаксисом )
set nocompatible 
set termguicolors
set t_Co=256
syntax enable

" Тема для **NeoVim**
colorscheme gruvbox
"let base16colorspace=256

"# UX config

" Делаем автозапуск Omnifunc (автодополнение)
filetype plugin on
set omnifunc=syntaxcomplete#Complete

" Vim теперь использует нормальное копирование (системный буфер)
set clipboard=unnamedplus

" Делаем нормальные табы
set expandtab
set smarttab
" 1 таб = 4 пробела
set shiftwidth=2
set tabstop=2
set ai "Auto indent
set si "Smart indent
set wrap "Wrap lines

" Нормальный поиск
set ignorecase
set smartcase
set wildmode=full

```

Также, моя сборка делится на две части: *быстрая* и **мощная**. Дабы использовать их отдельно, я создал две папки: `small_build` и `big_build`. Сам я использую `big_build`, потому что сижу на довольно мощном ПК, который может выдержать несколько серверов для линтинга языков.

Содержание `big_build/plugins.vim`, файл, в котором перечисляются все плагины:

```vim
call plug#begin()
" NERDTree - быстрый просмотр файлов
Plug 'preservim/nerdtree'

" Просмотр папок
Plug 'preservim/nerdtree'
"" Дополнение к NERDTree (Git Integration)
Plug 'Xuyuanp/nerdtree-git-plugin'

" Нормальная визуализация табов
Plug 'Yggdroot/indentLine'

" Линия статуса
Plug 'itchyny/lightline.vim'

" Emmet для Vim
Plug 'mattn/emmet-vim'

" Темы
Plug 'joshdick/onedark.vim'
Plug 'dikiaap/minimalist'
Plug 'morhetz/gruvbox'

" CSS/SCSS/HTML подсветка цвета 
Plug 'gorodinskiy/vim-coloresque'

" HTML, CSS, JS beautify ( нормальное форматирование )
Plug 'maksimr/vim-jsbeautify'

" Zen mode
Plug 'junegunn/goyo.vim'
Plug 'junegunn/limelight.vim'

" Git Integration
Plug 'airblade/vim-gitgutter'

" NERDCommenter (плагин для комментирования)
Plug 'preservim/nerdcommenter'

" Сниппеты (сноски кода)
Plug 'SirVer/ultisnips' | Plug 'honza/vim-snippets'

" ALE (Проверка на синтаксические ошибки)
Plug 'dense-analysis/ale'

" i3 syntax
" Plug 'PotatoesMaster/i3-vim-syntax'

" Автодополнение (всё-таки Coc)
Plug 'neoclide/coc.nvim', {'branch': 'release'}

" Markdown
" sudo npm install instant-markdown -g
Plug 'suan/vim-instant-markdown', {'for': 'markdown'}

" UML в Vim
Plug 'aklt/plantuml-syntax'
Plug 'scrooloose/vim-slumlord'

" FuzzyFinder в Vim
Plug 'junegunn/fzf' 

" Быстрое передвижение
Plug 'easymotion/vim-easymotion'

" LaTeX
Plug 'lervag/vimtex'

" Поддержка С, C++
" Plug 'xavierd/clang_complete'

" Deoplete
" Plug 'Shougo/deoplete.nvim'

" Python3 IDE в NeoVim
"Plug 'python-mode/python-mode', { 'for': 'python', 'branch': 'develop' }

" Что-то типо Jupyter'а прямо в Vim
" Plug 'metakirby5/codi.vim'

" Поддержка синтаксиса многих ЯП
Plug 'sheerun/vim-polyglot'

" Панель на Ctrl+P
"Plug 'ctrlpvim/ctrlp.vim'

" StartScreen
Plug 'mhinz/vim-startify'

" Отладчик для Vim
"Plug 'vim-vdebug/vdebug'

call plug#end()
```

Для того, чтобы не вставлять настройки плагинов и сами плагины вместе (не делать кашу), я разделил настройки от плагинов и поместил их в `~/.config/nvim/big_build/plugin_configs.vim`

```vim
"# Plugins configs

" NERDTree
" Размер NERDTree
let g:NERDTreeWinSize=30
" NERDTree справа
let g:NERDTreeWinPos = "right"
" Немного украсим NERDTree
let NERDTreeMinimalUI = 1
let NERDTreeDirArrows = 1

" Coc-bind-download
" Функция для установки пакетов для COC
function CocPacks()
          :CocInstall coc-html coc-python coc-emmet coc-css coc-clangd coc-ultisnips
  endf

" Вызов функции с помощью <C-\>
nmap <C-\> :call CocPacks()<CR>

" ALE
" Устанавливаем, как будет показываться ошибка и предупреждение
let g:ale_sign_error = '>'
let g:ale_sign_warning = '?'
" Проверяем файлы только при сохранении (для экономии энергии)
let g:ale_lint_on_text_changed = 'never'
let g:ale_lint_on_insert_leave = 0
let g:ale_lint_on_save = 1

" Emmet
let g:user_emmet_mode='i'    "Активировать Emmet только в режиме ввода
" Активировать Emmet только в файлах .html, .css
let g:user_emmet_install_global = 0
autocmd FileType html,css EmmetInstall

" Линия статуса: конфигурация
set noshowmode " Табличка --INSERT-- больше не выводится на экран
set laststatus=2
let g:lightline = {
      \ 'colorscheme': 'gruvbox',
      \ 'active': {
      \   'left': [ [ 'mode', 'paste' ],
      \             [ 'gitbranch', 'readonly', 'filename', 'modified' ] ]
      \ },
      \ 'component_function': {
      \   'gitbranch': 'fugitive#head'
      \ },
      \ }

" NERDCommenter
" Разрешить комментировать пустые строки
let g:NERDCommentEmptyLines = 0

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

" Если линия все ещё сожержит комментарий (его знак или знаки), то
" расскоментировать далее эту строку (строки)
let g:NERDToggleCheckAllLines = 1

" Goyo + LimeLight
autocmd! User GoyoEnter Limelight
autocmd! User GoyoLeave Limelight!
" Default LimeLight config
" Color name (:help cterm-colors) or ANSI code
let g:limelight_conceal_ctermfg = 'gray'
let g:limelight_conceal_ctermfg = 240

" Color name (:help gui-colors) or RGB color
let g:limelight_conceal_guifg = 'DarkGray'
let g:limelight_conceal_guifg = '#777777'

" Default: 0.5
let g:limelight_default_coefficient = 0.7

" Number of preceding/following paragraphs to include (default: 0)
"let g:limelight_paragraph_span = 1

" Beginning/end of paragraph
"   When there's no empty line between the paragraphs
"   and each paragraph starts with indentation
let g:limelight_bop = '^\s*$' 
let g:limelight_eop = '\ze\n^\s*$'

" Highlighting priority (default: 10)
"   Set it to -1 not to overrule hlsearch
let g:limelight_priority = -1


" Deoplete
" Запускать Deoplete при запуске
"let g:deoplete#enable_at_startup = 1
"let g:deoplete#auto_completion_start_length = 1
"let g:deoplete#sources = {}
"let g:deoplete#sources._ = []

" Markdown
" Отключаем автостарт
" :InstantMarkdownPreview - включить просмотр Markdown
let g:instant_markdown_autostart = 0
" Включаем MathJax
let g:instant_markdown_mathjax = 1
" Включаем автоскролл
let g:instant_markdown_autoscroll = 0
" Обновление только после сохранения
let g:instant_markdown_slow = 1

" Indent Line (Tabs)
let g:indentLine_char_list = ['|', '¦', '┆', '┊']
let g:indentLine_conceallevel = 1

" Latex
" Язык (стандартный LaTeX)
let g:tex_flavor='latex'
" Просмотр PDF
let g:vimtex_view_method='zathura'
" Маскировка скобок
let g:vimtex_quickfix_mode=0
set conceallevel=1
let g:tex_conceal='abdmg'

" JS-Beautify
autocmd FileType javascript noremap <buffer>  <leader>b :call JsBeautify()<cr>
" for json
autocmd FileType json noremap <buffer> <leader>b :call JsonBeautify()<cr>
" for jsx
autocmd FileType jsx noremap <buffer> <leader>b :call JsxBeautify()<cr>
" for html
autocmd FileType html noremap <buffer> <leader>b :call HtmlBeautify()<cr>
" for css or scss
autocmd FileType css noremap <buffer> <leader>b :call CSSBeautify()<cr>

" OneDark
" 256 цветов для OneDark
" let g:onedark_termcolors = 16

"if (has("autocmd") && !has("gui_running"))
  "augroup colors
    "autocmd!
    "let s:background = { "gui": "#282C34", "cterm": "235", "cterm16": "0" }
    "autocmd ColorScheme * call onedark#set_highlight("Normal", { "bg": s:background }) "No `fg` setting
  "augroup END
"endif

" Startify
nmap <Leader>h :Startify<CR>
```

Также у Coc (плагин автодополнения) довольно большие настройки, которые путают любого. Как вы уже догадались, я и их отделил:

`~/.config/nvim/big_build/coc_settings.vim`

```vim
"
" 										Стандартная
" 										конфигурация
" 										COC
"
"
" if hidden is not set, TextEdit might fail.
set hidden

" Some servers have issues with backup files, see #649
set nobackup
set nowritebackup

" Better display for messages
set cmdheight=1

" You will have bad experience for diagnostic messages when it's default 4000.
set updatetime=300

" don't give |ins-completion-menu| messages.
set shortmess+=c

" always show signcolumns
set signcolumn=yes

" Use tab for trigger completion with characters ahead and navigate.
" Use command ':verbose imap <tab>' to make sure tab is not mapped by other plugin.
inoremap <silent><expr> <TAB>
      \ pumvisible() ? "\<C-n>" :
      \ <SID>check_back_space() ? "\<TAB>" :
      \ coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"

function! s:check_back_space() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction

" Use <c-space> to trigger completion.
inoremap <silent><expr> <c-space> coc#refresh()

" Use <cr> to confirm completion, `<C-g>u` means break undo chain at current position.
" Coc only does snippet and additional edit on confirm.
inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"
" Or use `complete_info` if your vim support it, like:
" inoremap <expr> <cr> complete_info()["selected"] != "-1" ? "\<C-y>" : "\<C-g>u\<CR>"

" Use `[g` and `]g` to navigate diagnostics
nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)

" Remap keys for gotos
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" Use K to show documentation in preview window
nnoremap <silent> K :call <SID>show_documentation()<CR>

function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  else
    call CocAction('doHover')
  endif
endfunction

" Highlight symbol under cursor on CursorHold
autocmd CursorHold * silent call CocActionAsync('highlight')

" Remap for rename current word
nmap <leader>rn <Plug>(coc-rename)

" Remap for format selected region
xmap <leader>f  <Plug>(coc-format-selected)
nmap <leader>f  <Plug>(coc-format-selected)

augroup mygroup
  autocmd!
  " Setup formatexpr specified filetype(s).
  autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
  " Update signature help on jump placeholder
  autocmd User CocJumpPlaceholder call CocActionAsync('showSignatureHelp')
augroup end

" Remap for do codeAction of selected region, ex: `<leader>aap` for current paragraph
xmap <leader>a  <Plug>(coc-codeaction-selected)
nmap <leader>a  <Plug>(coc-codeaction-selected)

" Remap for do codeAction of current line
nmap <leader>ac  <Plug>(coc-codeaction)
" Fix autofix problem of current line
nmap <leader>qf  <Plug>(coc-fix-current)

" Create mappings for function text object, requires document symbols feature of languageserver.
xmap if <Plug>(coc-funcobj-i)
xmap af <Plug>(coc-funcobj-a)
omap if <Plug>(coc-funcobj-i)
omap af <Plug>(coc-funcobj-a)

" Use <TAB> for select selections ranges, needs server support, like: coc-tsserver, coc-python
nmap <silent> <TAB> <Plug>(coc-range-select)
xmap <silent> <TAB> <Plug>(coc-range-select)

" Use `:Format` to format current buffer
command! -nargs=0 Format :call CocAction('format')

" Use `:Fold` to fold current buffer
command! -nargs=? Fold :call     CocAction('fold', <f-args>)

" use `:OR` for organize import of current buffer
command! -nargs=0 OR   :call     CocAction('runCommand', 'editor.action.organizeImport')

" Add status line support, for integration with other plugin, checkout `:h coc-status`
set statusline^=%{coc#status()}%{get(b:,'coc_current_function','')}

" Using CocList
" Show all diagnostics
nnoremap <silent> <space>a  :<C-u>CocList diagnostics<cr>
" Manage extensions
nnoremap <silent> <space>e  :<C-u>CocList extensions<cr>
" Show commands
nnoremap <silent> <space>c  :<C-u>CocList commands<cr>
" Find symbol of current document
nnoremap <silent> <space>o  :<C-u>CocList outline<cr>
" Search workspace symbols
nnoremap <silent> <space>s  :<C-u>CocList -I symbols<cr>
" Do default action for next item.
nnoremap <silent> <space>j  :<C-u>CocNext<CR>
" Do default action for previous item.
nnoremap <silent> <space>k  :<C-u>CocPrev<CR>
" Resume latest coc list
nnoremap <silent> <space>p  :<C-u>CocListResume<CR>

 let &t_SI = "\e[3 q"
 let &t_EI = "\e[0 q"
 set cursorline

```



Теперь те же настройки, но для меньшего билда подряд:

`~/.config/nvim/small_build/init.vim` (если вам нужен маленький билд, перенесите init.vim на директорию выше и замените старый `init.vim`)

```vim
source ~/dotfiles/config/nvim/small_build/plugins.vim
source ~/dotfiles/config/nvim/small_build/plugin_configs.vim
"# Binds
" Комбинация клавиш jkl - действует как Escape в режиме Insert
imap jkl <ESC>

" Апгрейд линейки
set relativenumber

" Комбинация клавиш jks - действует как Escape + сохранение 
" в режиме вставки
imap jks <ESC>:w<CR>

" Передвижение между окнами Ctrl+hjkl
map <C-h> <C-w>h
map <C-j> <C-w>j
map <C-k> <C-w>k
map <C-l> <C-w>l

" F2 чтобы выходить из PasteMode (мод для вставки текста)
set pastetoggle=<F2>

" <leader> теперь пробел
let mapleader = " "

" Дублировать все виды кавычек и скобок
inoremap ' ''<left>
inoremap " ""<left>
inoremap [ []<left>
inoremap { {}<left>
inoremap ( ()<left>
inoremap {<CR> {<CR>}<ESC>O
inoremap {;<CR> {<CR>};<ESC>O



""""PLUGINS BINDS""""

""" NERDTree
" В нормальном режиме Space+f вызывает :NERDTree
" Ctrl+c - комментирует строки
nmap <leader>f :NERDTreeToggle<CR>
vmap <C-c> <plug>NERDCommenterToggle
nmap <C-c> <plug>NERDCommenterToggle

""" Goyo
nmap <leader>z :Goyo 150<CR>

""" Emmet
" Активировать дополнение Emmet на Tab+e
" Использовать Emmet (автодополнить код)
let g:user_emmet_expandabbr_key = '<tab>e'

""" 

"# UI config

" Делаем Vim цветным ( с темой и синтаксисом )
set nocompatible 
set t_Co=256
syntax enable

" Тема для **NeoVim**
colorscheme oceanic-next

"# UX config

" Делаем автозапуск Omnifunc (автодополнение)
filetype plugin on
set omnifunc=syntaxcomplete#Complete

" Vim теперь использует нормальное копирование (системный буфер)
set clipboard=unnamedplus

" Делаем нормальные табы
set expandtab
set smarttab
" 1 таб = 4 пробела
set shiftwidth=4
set tabstop=4
set ai "Auto indent
set si "Smart indent
set wrap "Wrap lines

" Нормальный поиск
set ignorecase
set smartcase
set wildmode=full

```

`~/.config/nvim/small_build/plugins.vim`

```vim
call plug#begin() " NERDTree - быстрый просмотр файлов Plug 'preservim/nerdtree'

" Просмотр папок
Plug 'preservim/nerdtree'

" Линия статуса
Plug 'itchyny/lightline.vim'

" Emmet для Vim
Plug 'mattn/emmet-vim'

" Тема
Plug 'mhartington/oceanic-next'

" CSS/SCSS/HTML подсветка цвета 
Plug 'gorodinskiy/vim-coloresque'

" Zen mode
Plug 'junegunn/goyo.vim'

" Git Integration
Plug 'airblade/vim-gitgutter'

" NERDCommenter (плагин для комментирования)
Plug 'preservim/nerdcommenter'

call plug#end()
```

`~/.config/nvim/small_build/plugin_configs.vim`

```vim
"# Plugins configs

" NERDTree
" Размер NERDTree
let g:NERDTreeWinSize=30
" NERDTree справа
let g:NERDTreeWinPos = "right"
" Немного преукрасим NERDTree
let NERDTreeMinimalUI = 1
let NERDTreeDirArrows = 1

" Emmet
let g:user_emmet_mode='i'    "Активировать Emmet только в режиме ввода
" Активировать Emmet только в файлах .html, .css
let g:user_emmet_install_global = 0
autocmd FileType html,css EmmetInstall

" Линия статуса: конфигурация
set noshowmode " Табличка --INSERT-- больше не выводится на экран
set laststatus=2
let g:lightline = {
      \ 'colorscheme': 'oceanicnext',
      \ 'active': {
      \   'left': [ [ 'mode', 'paste' ],
      \             [ 'gitbranch', 'readonly', 'filename', 'modified' ] ]
      \ },
      \ 'component_function': {
      \   'gitbranch': 'fugitive#head'
      \ },
      \ }

" NERDCommenter
" Разрешить комментировать пустые строки
let g:NERDCommentEmptyLines = 0

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

" Если линия все ещё сожержит комментарий (его знак или знаки), то
" расскоментировать далее эту строку (строки)
let g:NERDToggleCheckAllLines = 1
```



После всего этого NeoVim выглядит вот так:

![Screenshot_20200527_040117](/home/username77177/Pictures/Screenshot_20200527_040117.png)

### TaskWarrior

**TaskWarrior** - утилита, которую я использую для того, чтобы распределять все свои дела

`~/.taskrc`

```bash
# [Created by task 2.5.1 5/25/2020 02:53:54]
# Taskwarrior program configuration file.
# For more documentation, see http://taskwarrior.org or try 'man task', 'man task-color',
# 'man task-sync' or 'man taskrc'

# Here is an example of entries that use the default, override and blank values
#   variable=foo   -- By specifying a value, this overrides the default
#   variable=      -- By specifying no value, this means no default
#   #variable=foo  -- By commenting out the line, or deleting it, this uses the default

# Use the command 'task show' to see all defaults and overrides

# Files
data.location=~\/Dropbox\/\/Todo-Apps\/

# Color theme (uncomment one to use)
include /usr/share/doc/task/rc/dark-16.theme
#include /usr/share/doc/task/rc/dark-256.theme
#include /usr/share/doc/task/rc/solarized-dark-256.theme
#include /usr/share/doc/task/rc/solarized-light-256.theme
#include /usr/share/doc/task/rc/no-color.theme

# Выставляю приоритеты | Changing Prioritetes
uda.priority.values=Critical, Average, Low, Minor
# Формат даты
dateformat=D-M-Y
# Padding
row.padding=2
# С какого дня начинается неделя
weekstart=Monday
editor=nvim
search.case.sensitive=no
```

![Screenshot_20200527_040249](/home/username77177/Pictures/Screenshot_20200527_040249.png)

### Kitty

**Kitty** - очень быстрый терминал, который я предпочитаю использовать постоянно

`~/.config/kitty/`

```config
#: MOD
kitty_mod alt
clear_all_shortcuts no



#: Font
font_family      JetBrains Mono                                                          
bold_font        auto                                                                    
italic_font      auto                                                                    
bold_italic_font auto

font_size 12.0

adjust_line_height  0
adjust_column_width 0

disable_ligatures never
font_features none
box_drawing_scale 0.001, 1, 1.5, 2



#: Cursor
cursor #cccccc
cursor_text_color #888
cursor_shape block
cursor_beam_thickness 1.5
cursor_underline_thickness 2.0
cursor_blink_interval -1
cursor_stop_blinking_after 15.0



#: Bell
enable_audio_bell yes
visual_bell_duration 0.0
window_alert_on_bell yes
command_on_bell none
bell_on_tab yes



#: Scrollback (History)
scrollback_lines 2000
scrollback_pager less --chop-long-lines --RAW-CONTROL-CHARS +INPUT_LINE_NUMBER
scrollback_pager_history_size 0
wheel_scroll_multiplier 5.0
touch_scroll_multiplier 1.0



#: Mouse
mouse_hide_wait 3.0

url_color #a6e22e
url_style curly

open_url_modifiers kitty_mod
open_url_with default
url_prefixes http https file ftp
copy_on_select yes
strip_trailing_spaces never
rectangle_select_modifiers ctrl+alt
terminal_select_modifiers shift
select_by_word_characters :@-./_~?&=%+#
click_interval -1.0
focus_follows_mouse no
pointer_shape_when_grabbed arrow



#: Perfomance
repaint_delay 10
input_delay 3
sync_to_monitor yes



#: Window
remember_window_size  yes
initial_window_width  640
initial_window_height 480
enabled_layouts *

window_resize_step_cells 2
window_resize_step_lines 2
window_border_width 1.0
draw_minimal_borders no
window_margin_width 0.0
single_window_margin_width -1000.0
window_padding_width 8.0

placement_strategy center
active_border_color #00ff00
inactive_border_color #cccccc
bell_border_color #ff5a00
inactive_text_alpha 1.0
hide_window_decorations no
resize_debounce_time 0.1
resize_draw_strategy static
resize_in_steps no



#: Tabs
tab_bar_edge center
tab_bar_margin_width 0.0
tab_bar_style powerline
tab_bar_min_tabs 2
tab_switch_strategy previous
tab_fade 0.25 0.5 0.75 1
tab_separator " ┇"
tab_title_template "{title}"
active_tab_title_template none

active_tab_foreground   #000
active_tab_background   #eee
active_tab_font_style   bold-italic
inactive_tab_foreground #444
inactive_tab_background #999
inactive_tab_font_style normal


tab_bar_background none



#: Theming
foreground #abb2bf 
background_opacity 0.98
background_image none
background_image_layout tiled
background_image_linear no
dynamic_background_opacity no
background_tint 0.0
dim_opacity 0.75
selection_foreground #000000
selection_background #fffacd



#: Keyboard Shortcuts
#: #: Clipboard
map kitty_mod+c copy_to_clipboard
map kitty_mod+v  paste_from_clipboard
map kitty_mod+s  paste_from_selection
map shift+insert paste_from_selection
map kitty_mod+o  pass_selection_to_program

#: #: Scroll
map kitty_mod+up        scroll_line_up
map kitty_mod+k         scroll_line_up
map kitty_mod+down      scroll_line_down
map kitty_mod+j         scroll_line_down
map kitty_mod+page_up   scroll_page_up
map kitty_mod+page_down scroll_page_down
map kitty_mod+home      scroll_home
map kitty_mod+end       scroll_end
map kitty_mod+h         show_scrollback

#: #: Window
map kitty_mod+F9 new_os_window
map kitty_mod+enter new_window
map kitty_mod+x close_window
map kitty_mod+] next_window
map kitty_mod+[ previous_window
map kitty_mod+f move_window_forward
map kitty_mod+b move_window_backward
map kitty_mod+` move_window_to_top
map kitty_mod+r start_resizing_window
map kitty_mod+1 first_window
map kitty_mod+2 second_window
map kitty_mod+3 third_window
map kitty_mod+4 fourth_window
map kitty_mod+5 fifth_window
map kitty_mod+6 sixth_window
map kitty_mod+7 seventh_window
map kitty_mod+8 eighth_window
map kitty_mod+9 ninth_window
map kitty_mod+0 tenth_window

#: #: Tab
map kitty_mod+l next_tab
map kitty_mod+h previous_tab
map kitty_mod+t new_tab
map kitty_mod+q     close_tab
map kitty_mod+.     move_tab_forward
map kitty_mod+,     move_tab_backward
map kitty_mod+shift+t set_tab_title

#: #: Layouts
map kitty_mod+l next_layout

#: #: Font Size
map kitty_mod+equal     change_font_size all +2.0
map kitty_mod+minus     change_font_size all -2.0
map kitty_mod+backspace change_font_size all 0

#:#: Other
map kitty_mod+e kitten hints
map kitty_mod+p>f kitten hints --type path --program -
map kitty_mod+p>shift+f kitten hints --type path
map kitty_mod+p>l kitten hints --type line --program -
map kitty_mod+p>w kitten hints --type word --program -
map kitty_mod+p>h kitten hints --type hash --program -
map kitty_mod+p>n kitten hints --type linenum

map f11    toggle_fullscreen
map kitty_mod+f10    toggle_maximized
map kitty_mod+u      kitten unicode_input
map kitty_mod+f2     edit_config_file
map kitty_mod+escape kitty_shell window

map kitty_mod+a>m    set_background_opacity +0.1
map kitty_mod+a>l    set_background_opacity -0.1
map kitty_mod+a>1    set_background_opacity 1
map kitty_mod+a>d    set_background_opacity default
map kitty_mod+delete clear_terminal reset active



# Base16 Gruvbox dark, medium - kitty color config
# Scheme by Dawid Kurek (dawikur@gmail.com), morhetz (https://github.com/morhetz/gruvbox)
background #282828
foreground #d5c4a1
selection_background #d5c4a1
selection_foreground #282828
url_color #bdae93
cursor #d5c4a1
active_border_color #665c54
inactive_border_color #3c3836
active_tab_background #282828
active_tab_foreground #d5c4a1
inactive_tab_background #3c3836
inactive_tab_foreground #bdae93
tab_bar_background #3c3836

# normal
color0 #282828
color1 #fb4934
color2 #b8bb26
color3 #fabd2f
color4 #83a598
color5 #d3869b
color6 #8ec07c
color7 #d5c4a1

# bright
color8 #665c54
color9 #fb4934
color10 #b8bb26
color11 #fabd2f
color12 #83a598
color13 #d3869b
color14 #8ec07c
color15 #fbf1c7

# extended base16 colors
color16 #fe8019
color17 #d65d0e
color18 #3c3836
color19 #504945
color20 #bdae93
color21 #ebdbb2
```

Кстати, я везде использую тему Gruvbox, если вы ещё не заметили.

### Fish + OhMyFish

Я использую оболочку **Fish** с дополнением **OhMyFish**, для её установки нужно выполнить:

```bash
sudo pacman -Sy fish
curl -L https://get.oh-my.fish | fish
```

### Cmus

Для того, чтобы послушать музыку, я использую консольный музыкальный плеер, у которого довольно хорошее звучание **cmus**

`~/.config/cmus/base16.theme`

```bash
### colors
### https://github.com/drewvanstone/cmus-base16/blob/master/base16.theme#L5
### {{{
set color_cmdline_error=lightred
set color_cmdline_info=lightyellow
set color_cmdline_fg=lightred
set color_cmdline_bg=237
set color_separator=238
set color_statusline_bg=blue
set color_statusline_fg=233
set color_titleline_bg=237
set color_titleline_fg=white
set color_win_bg=black
set color_win_fg=241
set color_win_cur=lightgreen
set color_win_cur_sel_bg=green
set color_win_cur_sel_fg=233
set color_win_dir=lightblue
set color_win_fg=white
set color_win_inactive_cur_sel_bg=016
set color_win_inactive_cur_sel_fg=233
set color_win_sel_bg=237
set color_win_sel_fg=white
set color_win_title_bg=237
set color_win_title_fg=lightred
### }}}
```

### Zathura

**Zathura** - идеальный pdf-ридер.

`~/.config/zathura/zathurarc`

```config
# https://github.com/rwrr/zathura-gruvbox/blob/master/zathura-gruvbox-light
set notification-error-bg       "#fbf1c7" # bg
set notification-error-fg       "#9d0006" # bright:red
set notification-warning-bg     "#fbf1c7" # bg
set notification-warning-fg     "#b57614" # bright:yellow
set notification-bg             "#fbf1c7" # bg
set notification-fg             "#79740e" # bright:green

set completion-bg               "#d5c4a1" # bg2
set completion-fg               "#3c3836" # fg
set completion-group-bg         "#ebdbb2" # bg1
set completion-group-fg         "#928374" # gray
set completion-highlight-bg     "#076678" # bright:blue
set completion-highlight-fg     "#d5c4a1" # bg2

# Define the color in index mode
set index-bg                    "#d5c4a1" # bg2
set index-fg                    "#3c3836" # fg
set index-active-bg             "#076678" # bright:blue
set index-active-fg             "#d5c4a1" # bg2

set inputbar-bg                 "#fbf1c7" # bg
set inputbar-fg                 "#3c3836" # fg

set statusbar-bg                "#d5c4a1" # bg2
set statusbar-fg                "#3c3836" # fg

set highlight-color             "#b57614" # bright:yellow
set highlight-active-color      "#af3a03" # bright:orange

set default-bg                  "#fbf1c7" # bg
set default-fg                  "#3c3836" # fg
set render-loading              true
set render-loading-bg           "#fbf1c7" # bg
set render-loading-fg           "#3c3836" # fg

# Recolor book content's color
set recolor-lightcolor          "#fbf1c7" # bg
set recolor-darkcolor           "#3c3836" # fg
set recolor                     "true"
# set recolor-keephue             true      # keep original color

####ANOTHER THEME
# https://github.com/rwrr/zathura-gruvbox/blob/master/zathura-gruvbox-dark
#set notification-error-bg       "#282828" # bg
#set notification-error-fg       "#fb4934" # bright:red
#set notification-warning-bg     "#282828" # bg
#set notification-warning-fg     "#fabd2f" # bright:yellow
#set notification-bg             "#282828" # bg
#set notification-fg             "#b8bb26" # bright:green

#set completion-bg               "#504945" # bg2
#set completion-fg               "#ebdbb2" # fg
#set completion-group-bg         "#3c3836" # bg1
#set completion-group-fg         "#928374" # gray
#set completion-highlight-bg     "#83a598" # bright:blue
#set completion-highlight-fg     "#504945" # bg2

## Define the color in index mode
#set index-bg                    "#504945" # bg2
#set index-fg                    "#ebdbb2" # fg
#set index-active-bg             "#83a598" # bright:blue
#set index-active-fg             "#504945" # bg2

#set inputbar-bg                 "#282828" # bg
#set inputbar-fg                 "#ebdbb2" # fg

#set statusbar-bg                "#504945" # bg2
#set statusbar-fg                "#ebdbb2" # fg

#set highlight-color             "#fabd2f" # bright:yellow
#set highlight-active-color      "#fe8019" # bright:orange

#set default-bg                  "#282828" # bg
#set default-fg                  "#ebdbb2" # fg
#set render-loading              true
#set render-loading-bg           "#282828" # bg
#set render-loading-fg           "#ebdbb2" # fg

## Recolor book content's color
#set recolor-lightcolor          "#282828" # bg
#set recolor-darkcolor           "#ebdbb2" # fg
#set recolor                     "true"
## set recolor-keephue             true      # keep original color




# Начальная ширина
set window-width 1000
# Расширение файла в окне по высоте
set adjust-open "best-fit"
# Отступ между страницами
set page-padding 1
# Страниц на строку
set pages-per-row 1
# Шрифт
set font "JetBrains Mono 14"
# $HOME = ~
set window-title-home-tilde true
# То, что выделяется, сразу идёт в буфер обмена
set selection-clipboard clipboard
# Отступы
set statusbar-h-padding 0
set statusbar-v-padding 0
set page-padding 1
```

### i3-gaps

**i3-gaps** - прекрасный WM. Я его не использую, так как я перешёл на KDE, однако поделиться конфигурацией не могу:

`~/.config/i3/config`

```i3
set $mod Mod4

set $workspace1 "Desktop"
set $workspace2 "Web"
set $workspace3 "Terminal"
set $workspace4 "Editors"

#Autostart
#exec_always --no-startup-id "killall -q compton; compton --config ~/.config/compton.conf"
exec_always --no-startup-id "killall -q picom; picom --config ~/dotfiles/config/picom.conf"

exec killall -q notify-osd
exec /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1
exec --no-startup-id dunst -config ~/.config/dunst/dunstrc
exec --no-startup-id /home/username77177/.dropbox-dist/dropboxd

exec --no-startup-id setxkbmap -model pc105 -layout us,ru -option grp:win_space_toggle

exec --no-startup-id feh --bg-scale ~/wallpapers/$(ls ~/wallpapers | shuf -n 1)
#exec --no-startup-id feh --bg-scale ~/wallpapers/Witcher.png

##### Автозапуск приложений
exec alacritty

font pango: JetBrains Mono 10

# Переключение между рабочими областями
bindcode $mod+90 workspace $workspace1
bindcode $mod+87 workspace $workspace2
bindcode $mod+88 workspace $workspace3
bindcode $mod+89 workspace $workspace4

# Переключение между последними рабочими областями
bindsym $mod+v workspace next
bindsym $mod+c workspace prev

# Перемещать выделенное окно между областями
bindcode $mod+Shift+90 move container to workspace $workspace1
bindcode $mod+Shift+87 move container to workspace $workspace2
bindcode $mod+Shift+88 move container to workspace $workspace3
bindcode $mod+Shift+89 move container to workspace $workspace4

# Scratchpad ( летающие float окна поверх других )
bindcode $mod+105 scratchpad show

# bar
bar {
    status_command i3blocks -c ~/dotfiles/config/i3blocks/config
    position top
    separator_symbol "|"

    colors {
        background #282829 # Фон всей панели
        separator #636e72 # Фон разделителей
        statusline #edbdd2 # Цвет текста дополнительной панели

        # Цвет: Граница, фон, текст
        #Неактивные бары
        #Бар на котором вы сейас находитесь
        #?
        #Бар на котором новое приложение
        inactive_workspace      #3c3837 #3c3837 #ebdbb3
        focused_workspace       #3c3837 #98971b #ebdbb3
        active_workspace        #000000 #000000 #ffffff
        urgent_workspace       #3c3839 #689d7a #ebdbb3
    }
    # У самого последнего разметка: Фон, шрифт, граница
}

# Темы для границ окон
# Thanks, addy-dclxvi
# https://github.com/addy-dclxvi/i3-starterpack/blob/master/.config/i3/config

# colour of border, background, text, indicator, and child_border
client.focused              #bf616a #2f343f #d8dee8 #bf616a #d8dee8
client.focused_inactive     #2f343f #2f343f #d8dee8 #2f343f #2f343f
client.unfocused            #2f343f #2f343f #d8dee8 #2f343f #2f343f
client.urgent               #2f343f #2f343f #d8dee8 #2f343f #2f343f
client.placeholder          #2f343f #2f343f #d8dee8 #2f343f #2f343f
client.background           #2f343f


# Мод + Enter чтобы запустить терминал
bindsym $mod+Return exec alacritty
bindsym $mod+Shift+Return exec rxvt-unicode
bindsym Print exec scrot -e 'mv $f ~/Pictures/Screenshot_`date +%m-%d-%y_%R:%S`.png' && notify-send 'Scrot' 'Screenshot saved!' -t 400
bindsym $mod+Shift+f exec firefox
bindsym $mod+Shift+m exec alacritty -t 'ranger' -e ranger
bindsym $mod+Shift+n exec nautilus
bindsym $mod+Shift+t exec telegram-desktop
bindsym $mod+Shift+v exec alacritty -t 'vim' -e vim
bindsym $mod+Shift+b exec boostnote
bindsym $mod+Shift+d exec discord
bindsym $mod+Shift+c exec code
bindsym $mod+Shift+p exec pcmanfm
bindsym $mod+Shift+z exec zathura
bindsym $mod+Shift+x exec zeal
bindsym $mod+semicolon exec dm-tool lock
bindsym $mod+Shift+o exec xrandr --output eDP-1 --auto

bindsym $mod+m exec dmenu_run -b -i -fn 'JetBrains Mono' -nb '#282829' -nf '#ebdbb2' -sb '#98971a' -sf '#fbf1c7'

bindsym $mod+q kill
bindsym $mod+d exec "rofi -show drun -show-icons -drun-icon-theme Mojave-CT-Night-Mode"
bindsym Mod1+Tab exec "rofi -show window -show-icons -drun-icon-theme Mojave-CT-Night-Mode -kb-accept-entry space"

# Выйти из i3
bindsym $mod+Shift+q exit
# Перезагрузить i3
bindsym $mod+r restart

# Клавиша мода + мышка для того чтобы перетаскивать окна в плаваюшем режиме
floating_modifier $mod

# Mod + x
# Поменять сторону разделения
bindsym $mod+x split toggle

# F11
bindcode 95 fullscreen toggle

# Поменять вид контейнера
bindsym $mod+s layout stacking
bindsym $mod+w layout tabbed
bindsym $mod+e layout toggle split

# Перемещение между окнами
bindsym $mod+h focus left
bindsym $mod+j focus down
bindsym $mod+k focus up
bindsym $mod+l focus right

# Перемещать окна
bindsym $mod+Shift+h move left 20px
bindsym $mod+Shift+j move down 20px
bindsym $mod+Shift+k move up 20px
bindsym $mod+Shift+l move right 20px

# Меняем размер окон
bindsym $mod+Ctrl+Left resize shrink width 20 px or 20 ppt
bindsym $mod+Ctrl+Down resize grow height 20 px or 20 ppt
bindsym $mod+Ctrl+Up resize shrink height 20 px or 20 ppt
bindsym $mod+Ctrl+Right resize grow width 20 px or 20 ppt

# 2-й вариaнт
bindsym $mod+Ctrl+h resize shrink width 20 px or 20 ppt
bindsym $mod+Ctrl+j resize grow height 20 px or 20 ppt
bindsym $mod+Ctrl+k resize shrink height 20 px or 20 ppt
bindsym $mod+Ctrl+l resize grow width 20 px or 20 ppt

# Поменять режим с тиллинга на плавающий и обратно
bindsym $mod+f floating toggle

# Сменить фокус с тиллингового на плавающее окно
bindsym $mod+mod1+f focus mode_toggle

# Фокус на окно-родитель (контейнер)
bindsym $mod+a focus parent

# M + Shift + C перезагрузить конфигурацию
bindsym $mod+p reload


### Звук
# Thanks, guys
# https://unix.stackexchange.com/questions/21089/how-to-use-command-line-to-change-volume
# https://www.reddit.com/r/i3wm/comments/3a6nh3/help_how_to_use_function_keys_in_i3_config/

bindsym XF86AudioMute exec amixer -D pulse sset Master toggle && pkill -SIGRTMIN+10 i3blocks
bindsym XF86AudioRaiseVolume exec amixer -D pulse sset Master 5%+ && pkill -SIGRTMIN+10 i3blocks
bindsym XF86AudioLowerVolume exec amixer -D pulse sset Master 5%- && pkill -SIGRTMIN+10 i3blocks

### Яркость экрана
# TODO
bindsym XF86MonBrightnessUp exec xbacklight -inc 5 # increase screen brightness
bindsym XF86MonBrightnessDown exec xbacklight -dec 5 # decrease screen brightness

### Настройка переключения музыки
bindsym XF86AudioNext exec mpc next
bindsym XF86AudioPrev exec mpc prev
bindsym XF86AudioPlay exec mpc toggle
bindsym XF86AudioStop exec mpc stop


# Workspaces
assign [class="Nautilus|Thunar|ranger|vifm|discord|Pcmanfm|zathura"] $workspace1
assign [class="firefox|TelegramDesktop|Chromium|qutebrowser|FirefoxBeta|Navigator"] $workspace2
assign [class="Alacritty|URxvt|Kitty|gnome-terminal"] $workspace3
assign [class="Emacs|Gvim|vim|nvim|Code|marktext|Boostnote|Libreoffice"] $workspace4

# Focus
assign [class="Alacritty|Firefox|TelegramDesktop|Code|Emacs|Nautilus"] focus

# Fullscreen
for_window [class="discord"] fullscreen toggle

#Floating
for_window [title="(?i)(?:copying|deleting|moving)"] floating enable
for_window [window_role="(?i)(?:pop-up|setup)"]      floating enable
for_window [class="Pavucontrol|gnome-font-viewer"] floating enable


# ___  _____          ____
#|_ _||___ /         / ___|  __ _  _ __   ___
# | |   |_ \  _____ | |  _  / _` || '_ \ / __|
# | |  ___) ||_____|| |_| || (_| || |_) |\__ \
#|___||____/         \____| \__,_|| .__/ |___/
#                                 |_|
#i3-gaps ( дополнение для того, чтобы между окнами было пространство )


# Margins
for_window [class="^.*"] border pixel 2 # Рамка для всех окон в 2px
for_window [class="i3bar"] gaps outer current set 10
# smart_gaps on
gaps inner 10 # Отступы между окнами
gaps outer 2 # Отступы от экрана
#Keybinds
# Inner gaps
bindcode $mod+mod1+21 gaps inner current plus 5
bindcode $mod+mod1+20 gaps inner current minus 5
bindcode $mod+mod1+19 gaps inner current set 0
# Outer gaps
bindcode $mod+Ctrl+21 gaps outer current plus 5
bindcode $mod+Ctrl+20 gaps outer current minus 5
bindcode $mod+Ctrl+19 gaps outer current set 0

# Standard Gaps [ Win+Ctrl+Alt+0 ]
bindcode $mod+Ctrl+mod1+19 gaps inner current set 10; gaps outer current set 2

#exec code
exec killall -q save_your_eyes.sh &
exec killall -q save_your_eyes_light.sh &
exec killall -q notes.sh &
exec sleep 3
exec $HOME/dotfiles/scripts/save_your_eyes.sh &
exec $HOME/dotfiles/scripts/save_your_eyes_light.sh &
exec $HOME/dotfiles/scripts/notes.sh show &
exec --no-startup-id dropbox &
#####
```



### tmux

**tmux** - терминальный мультиплексор, который я также уже не  использую, потому что в Kitty есть встроенный мультиплексор.

`~/tmux.conf`

```tmux
set-option -g default-shell $SHELL
# set-option -g default-shell /usr/bin/fish # fish - шелл, который будет использоваться внутри
set -g default-terminal "screen-256color" # colors!
set -s escape-time 0
set-option -g status-keys vi
set -g base-index 1

set -g history-limit 10000 # Ограничение записи истории
# Префикс теперь Ctrl + A
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix


										# Панели и окна
setw -g pane-base-index 1 #! Сделать нумерацию в соответствии с окнами
setw -g automatic-rename on   # Переименовать окно в соответствии с программой
set -g renumber-windows on    # Перерасчитать окна, когда одно из них закрыто
bind - split-window -v # Разделить окно горизонтально с помощью <->
bind _ split-window -h # Разделить окна вертикально с помощью <_>
bind -r H resize-pane -L 2 # Увеличить панель с левой стороны
bind -r J resize-pane -D 2 # Увеличить панель снизу
bind -r K resize-pane -U 2 # Увеличить панель сверху
bind -r L resize-pane -R 2 # Увеличить панель с правой стороны 


										# Навигация
bind -r h select-pane -L  # Перейти на панель слева
bind -r j select-pane -D  # Перейти на панель справа
bind -r k select-pane -U  # Перейти на панель сверху
bind -r l select-pane -R  # Перейти на панель снизу
bind > swap-pane -D       # Поменять текущую и следущую панель местами
bind < swap-pane -U       # Поменять текущую и предыдущую панель местами
bind C-f command-prompt -p find-session 'switch-client -t %%' # Ctrl + F для поиска сессии
bind -r C-h previous-window # Переключиться на предыдущее окно
bind -r C-l next-window     # Переключиться на следущее окно 
bind Tab last-window        # Переключаться между последними окнами

# Перезагрузка конфига
bind r source-file ~/.tmux.conf \; display "Configuration Reloaded!"

# Переключение между окнами с помощью Shift + стрелки
bind -n S-Left  previous-window
bind -n S-Right next-window
setw -g mode-keys vi # Использовать клавиши из Vi
setw -g mouse on # Разрешить использование мыши
setw -g monitor-activity on

										# Тема
#Theme https://github.com/morhetz/gruvbox-contrib
set -g status 'on'
set -g status-bg '#282828'
set -g status-fg '#fbf1c7'
set -g status-justify 'centre' # Оглавление окон посередине
set -g message-style fg='#98971a',bg='default' # Стиль сообщений (проверка на Ctrl + a + r)
set -g message-command-style fg='colour222',bg='colour238'
# Разделитель панелей
set -g pane-border-style fg='#1d2021'
set -g pane-active-border-style fg='#fb4934' # Красный цвет у активной панели
# Стиль окон
setw -g window-status-activity-style fg='#b16286',bg='colour235',none
setw -g window-status-separator ''
setw -g window-status-style fg='#b8bb26',bg='#3c3836',none


#Синтаксис: #[стиль текста]#специальный_идентификатор() или просто текст
# Левая панель статус-бара
set -g status-left-length '100'
    # Session, Window, Pane, 
set -g status-left "#[bg=#d5c4a1,fg=#282828] Session: #S #[fg=default,bg=#504945] Window: #[fg=#fb4934]#I #[fg=default,bg=#3c3836] Pane: #[fg=#8ec07c]#P #[fg=#3c3836,bg=default]"
# Правая панель статус-бара
set -g status-right-length '100'
    # Wifi, Time, Username
#set -g status-right "#[fg=#3c3836]#[fg=default,bg=#3c3836] #(nmcli c | grep wlp3s0 | awk '{print $1}' || echo 'No connection') #[fg=default,bg=#504945] %R:%S #[bg=#d5c4a1,fg=#282828] #H "
    # App name, Time, Username
set -g status-right "#[fg=#3c3836]#[fg=default,bg=#3c3836] #W #[fg=default,bg=#504945] %R:%S #[bg=#d5c4a1,fg=#282828] #H "
# Середина в статус-баре
setw -g window-status-format "#[bg=#282828,fg=#3c3836] #[bg=default,fg=#9897a1]   #I:#W   #[bg=#282828,fg=#3c3836] "
setw -g window-status-current-format "#[bg=#282828,fg=#3c3836] #[bg=default]   #[fg=#fb4934] #[fg=#d79921]⚡#[fg=#fb4934]: #W     #[bg=#282828,fg=#3c3836]"





# Позаимствовано у rajanand02 
# https://gist.github.com/rajanand02/9407361
#set -g status-position bottom
#set -g status-bg 'colour235'
#set -g status-justify 'centre'
#set -g status-left-length '100'
#set -g status-right-length '100'
#set -g message-style fg='colour222',bg='colour238'
#set -g message-command-style fg='colour222',bg='colour238'
#set -g pane-border-style fg='colour238'
#set -g pane-active-border-style fg='colour154'
#setw -g window-status-activity-style fg='colour154',bg='colour235',none
#setw -g window-status-separator ''
#setw -g window-status-style fg='colour121',bg='colour235',none
#set -g status-left '#[fg=colour232,bg=colour154] #S #[fg=colour154,bg=colour238,nobold,nounderscore,noitalics]#[fg=colour222,bg=colour238] #W #[fg=colour238,bg=colour235,nobold,nounderscore,noitalics]#[fg=colour121,bg=colour235] #(whoami)  #(uptime  | cut -d " " -f 1,2,3) #[fg=colour235,bg=colour235,nobold,nounderscore,noitalics]'
#set -g status-right '#[fg=colour235,bg=colour235,nobold,nounderscore,noitalics]#[fg=colour121,bg=colour235] %r  %a  %Y #[fg=colour238,bg=colour235,nobold,nounderscore,noitalics]#[fg=colour222,bg=colour238] #H #[fg=colour154,bg=colour238,nobold,nounderscore,noitalics]#[fg=colour232,bg=colour154] #(rainbarf --battery --remaining --no-rgb) '
#setw -g window-status-format '#[fg=colour235,bg=colour235,nobold,nounderscore,noitalics]#[default] #I  #W #[fg=colour235,bg=colour235,nobold,nounderscore,noitalics]'
#setw -g window-status-current-format '#[fg=colour235,bg=colour238,nobold,nounderscore,noitalics]#[fg=colour222,bg=colour238] #I  #W  #F #[fg=colour238,bg=colour235,nobold,nounderscore,noitalics]'
```

