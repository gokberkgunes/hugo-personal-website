---
title: "Keyboat Layouts for Xorg & TTY"
date: 2022-07-30T20:09:05+03:00
tags: ["server", "selfhost"]
draft: false
---
This guide will lead one in following fields:
- Figure out possible available layouts,
- change layouts either temporarily or permanently,
- Alter layouts in Xorg or in TTY.

Requirements: xorg, xorg-setxkmap, root access (doas/sudo/su).
## Xorg: Keyboard Layouts and Variants
For Xorg layouts and variants and explanations is located in a text file:
`/usr/share/X11/xkb/rules/evdev.lst`.

After the decision of layout, it is recommended to pick a variant for preferred
layout. E.g. for German, **de**, picking German (dead acute) will be **de
deadacute**.

**Note:** A more tidy version of this list may be found [here](https://man.archlinux.org/man/xkeyboard-config.7).

#### Temporarily change the layout
If one wants to temporarily change the layout `setxkbmap` may be used. This is
handy for scripting and keymapping purposes.
```sh
setxkbmap tr alt
```
#### Permanently change the layout
Create the configuration file as shown.
```sh
doas touch /etc/X11/xorg.conf.d/00-keyboard.conf
```
Edit this file as root-user and add desired settings to it. An example is
   given below for demonstration purposes.
```text
Section "InputClass"
	Identifier "system-keyboard"
	MatchIsKeyboard "on"
	Option "XkbLayout" "tr,ru,us"
	Option "XkbVariant" "alt,,dvorak"
	Option "XkbOptions" "alt_space_toggle,caps:super,altwin:menuwin" #"grp:rctrl_toggle"
EndSection
```
**XkbLayout** option states the desired layouts.
**XkbVariant** indicates the layouts' variants.
**XkbOption** is used for additional settings.

## TTY: Keyboard Layouts and Variants
For TTY layouts file names in the following directory will be used:
`/usr/share/kbd/keymaps/i386`
For example, picking a qwerty layout: tralt.map.gz. Then the name is `tralt`.

#### Temporarily change the TTY layout
Temporary change might not be possible; however, at least it can be used to
ensure permanent change will work successfully. This command requires a TTY to
work and to report back correctly. (For example, swapping to 2nd TTY is
possible `CTRL+ALT+F2` in Xorg and `ALT+F2` in TTY.)
```sh
loadkeys -v "tralt"
```

#### Permanently change the TTY layout
Create the configuration file as below.
```sh
doas touch /etc/vconsole.conf
```
This file should be edited for wanted layout such as below.
```sh
KEYMAP=tralt
FONT=Lat2-Terminus16
```

**Note:** Pre-installed fonts can be found at `/usr/share/kbd/consolefonts`.

## Further Reading
