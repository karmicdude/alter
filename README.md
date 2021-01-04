
# Alter Keyboard Layouts

## Layouts

### Alter RU Typewriter

`Diktor`-based layout, which underwent changes over the years. With my experience, I have come to realize that I need to use dynamic zones and move away from static finger placement. So I reworked a lot of things. The main changes were to the left hand, symbols and signs. 

Trust me, it's much better than the vanilla `Diktor` layout.

[![Alter RU Typewriter](https://i.imgur.com/6Al1nBq.png)](https://i.imgur.com/canOIzT.png){:target="_blank"}

### Alter US Typewriter

`Qwerty`-based layout. Actually, there aren't many changes and I plan to switch to customized `Dvorak` (but it's not sure) after the new [Moonlander](https://www.zsa.io/moonlander) programmable keyboard is delivered to me. Anyway, I'm typing on it now.

[![Alter US Typewriter](https://i.imgur.com/qEsuTTV.png)](https://i.imgur.com/mbz7c9J.png){:target="_blank"}

## Installation

### Linux

#### Problematics
Changes affect to `XKB` subsystem configuration files, located  in `/usr/share/X11/xkb`.
The configuration file set comes with the specific package of your Linux distribution.

Updating a package through the package manager will replace the modified files with the original ones and this creates some headache.

I created a patch [alter.patch](https://github.com/karmicdude/alter/blob/main/alter.patch) that makes changes to the `XKB` configuration files.

I propose several ways of solving this issue. You can choose any way that suits you.

#### Package manager hook (Recommended)

I use `Arch Linux` as my main distro, so I will give an example based on it. 

The `XKB` subsystem is handled by the `xkeyboard-config` package.

```bash
# Create the direcotry for alter-layout patch file
sudo mkdir -p /usr/share/alter-layout
# Download latest patch file from repo
sudo curl -LJo \
	/usr/share/alter-layout/alter.patch \
	https://raw.githubusercontent.com/karmicdude/alter/main/alter.patch
# Or you can do it manually.
# Clone repo and copy patch file
git clone https://github.com/karmicdude/alter.git
sudo cp alter/alter.patch /usr/share/alter-layout/


# Create the hooks directory for pacman
sudo mkdir -p /etc/pacman.d/hooks
# Create the pacman hook config
cat << EOF | sudo tee /etc/pacman.d/hooks/layouts.hook
[Trigger]
Operation=Install
Operation=Upgrade
Type=Package
Target=xkeyboard-config

[Action]
Description=Recovery Custom Keyboard Layouts
When=PostTransaction
Exec=/bin/sh -c 'patch -d/ -s -p0 < /usr/share/alter-layout/alter.patch'
EOF
```
Now when you update the system, the `XKB` configuration files will be replaced by the original ones, but afterwards a hook will run which will return everything back to the way it was.

```bash
:: Processing package changes...  
(1/1) reinstalling xkeyboard-config [##############################################] 100%  
:: Running post-transaction hooks...  
(1/2) Arming ConditionNeedsUpdate...  
(2/2) Recovery Custom Keyboard Layouts   <== this is it
```

#### Package manager skip options

Apply the patch file first. 

```bash
sudo patch -d/ -s -p0 < /path/to/arch.patch
```
Now we need to somehow block files changes.

With `pacman` you can use one of two options to [skip files](https://wiki.archlinux.org/index.php/Pacman#Skip_file_from_being_upgraded) from being `updated/installed` to system - `NoUpgrade` or `NoExtract` options in `/etc/pacman.conf`. For example:

```bash
# Uncomment/Add the lines. 
# The missing root slash is not an mistake. See. man pacman.conf
NoUpgrade = usr/share/X11/xkb/symbols/us usr/share/X11/xkb/symbols/ru
NoUpgrade = usr/share/X11/xkb/rules/evdev.lst usr/share/X11/xkb/rules/evdev.xml
NoUpgrade = usr/share/X11/xkb/rules/base.lst usr/share/X11/xkb/rules/base.xml
```
So you can go further and block the package update at all (but it totally crazy imho):

```bash
IgnorePkg = xkeyboard-config
```

I think it is better to patch changes like a first case example than to keep an outdated version of the system files.

#### Yet another desperate trick

You can also simply take off the write access to set of files. This may be useful if your package manager does not allow you to do tricks like the ones above.

```bash
sudo chmod ugo-w \
	/usr/share/X11/xkb/symbols/{us,ru} \
	/usr/share/X11/xkb/rules/{evdev,base}.{lst,xml}
# But I suspect that the package manager will try to correct the permissions 
# A more reliable way is to set the immutability attribute.
sudo chattr +i \
	/usr/share/X11/xkb/symbols/{us,ru} \
	/usr/share/X11/xkb/rules/{evdev,base}.{lst,xml}
```
