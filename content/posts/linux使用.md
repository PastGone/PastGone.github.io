+++
date = '2026-04-06T01:09:58+08:00'
draft = false
title = 'linux 使用'
+++
# linux 使用

## 美化

系统美化可分为  
主题，wm 主题，  
gtk：
<https://github.com/vinceliuice/WhiteSur-gtk-theme>

qt：kvantummanager + <https://github.com/vinceliuice/WhiteSur-kde/tree/master/Kvantum>

图标，<https://github.com/vinceliuice/WhiteSur-icon-theme>

鼠标指针，<https://github.com/ful1e5/apple_cursor> macOS-White

字体，  
字体选maple nomal nl nf cn，重点是nl nf cn ，不连字，带图标，支持中文，nomal 主要是宽一点。

grub 主题，感觉，没啥好看的。refind 倒是有不少好看的。

软件美化主要是输入法
fcitx5 <https://github.com/Passthem-desu/fcitx5-theme-pt-cute-light>

## 输入法

使用 fcitx5  
fcitx5 yun pingyin 和 fcitx5 rime 都不错。  
yun pingyin 联想好用  
rime 主要有词库支持  
安装时注意在
~/.profile 里启用，写一堆配置。  
可以有皮肤美化。

## 软件推荐

1. 任务管理器

经典桌面自带的，简陋如lxqt 和xfce。  
GNOME System Monitor 还行，有点难看，逻辑一般  
kde 的，别的桌面无法使用  
第三方，  
好看的，
Mission Center占用大，他不是gtk 写的吗。。。。。  
Resources功能不完善。  
所以最终选择system-monitoring-center，这个还行。功能完善，界面不丑。  
终端 tui bop  
所以卸载那些其他的，使用system-monitoring-center，bop  
2. 可视化包管理工具  
新立得 deb 够用。  
aur pamac 有缺点，占用大，但最好用  
3. 文本编辑器  
notepad 级:  
mousepad_vs_pluma,这两个都不错。  
不过。mousepad空空间占用少一些。内存占用差不多。  
pluma功能强一点点。  
编辑器级：  
太多了，  
什么notepad--，Geany，CudaText，BowPad都不错。  
IDE级：
vscode与其类似物。

## 吐槽

### 中文支持问题

1. f1 是启动时的命令输出端口，f7 是图形界面。
linux f2 -f6 是tty，默认不支持中文，所以，当系统语言环境为中文时，
使用nano时，会有问题。
 可以使用kmscon 折腾解决。
2. 中文文件夹。
中文环境时，某些系统会给你搞出 ~/桌面 之类的文件夹。一般是  
xdg-user-dirs-update
~/.config/user-dirs.dirs
~/.config/user-dirs.locale
这几个东西的锅，也挺折腾，挺麻烦。

## 其他

1. 关机，注销卡死。
ai 说 xfce 桌面比较常见，反正我是遇到很多次了。
只能长按关机键和切到tty ，输入用户名，密码登录，然后使用命令注销。恶心。
2. arch  linux 所谓的包多，是假象。
官方包一点不多。
aur 有时副作用多。很多软件没有原生包，只能解包，打包，下载源码编译，反正各种构建。原生包是基本没有的。
debian 系，一些软件，强绑glic，低了用不了。 包比较分散，不是所有软件都有ppa。只能到官网下。
golang 打包的工具，真是好文明。
那些能通过脚本一件安装的开发工具的bin包，其实也不错。
arch 的可选依赖体验很好

---

linux 并不比win 省心，老实说。
