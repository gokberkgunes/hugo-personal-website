---
title: "Tex Latex Arch"
date: 2024-10-11T16:01:32+03:00
authors: ["Gokberk Gunes", ]
tags: []
header_image:
draft: true
---

In this post, we will install LaTeX to our Arch Linux or its derivatives.
Transiently, this guide shall also accompany a way to manage our bibliography.
Lastly, we will set a bash script to convert the bibliography and .tex file
into a pdf.


Install the basic tex files.
```sh
sudo pacman -Sy texlive-basic
sudo pacman -Sy texlive-latexrecommended
```

Above packages for sure will not be enough for us. For example when I try to
use `siunitx` package, I was told that file `sixunitx.sty` is not found.
These files are generally bundled together in Arch repositories and served, and
the hard part lies in to find which packages to install. 

Luckily, we can use `pacman -F <file_name>` to find out which packages serve
our required file. For example, when I run this command for `siunitx.sty`,
I get below printed in my terminal.
```shtop
pacman -F siunitx.sty
```
```txtb
world/texlive-mathscience 2024.2-2 (texlive)
    usr/share/texmf-dist/tex/latex/siunitx/siunitx.sty
extra/texlive-mathscience 2024.2-2 (texlive)
    usr/share/texmf-dist/tex/latex/siunitx/siunitx.sty
```

Below is a list of some extra packages we might install our computers to proof
future requirements. 
```sh
sudo pacman -Sy texlive-mathscience
sudo pacman -Sy texlive-langenglish
sudo pacman -Sy texlive-latexextra texlive-pictures
sudo pacman -Sy texlive-xetex texlive-luatex
sudo pacman -Sy texlive-fontsrecommended texlive-fontsextra
```

Install `biber` for bibliography management. This program should replace BibTex.
```sh
sudo pacman -Sy biber texlive-bibtexextra
```
