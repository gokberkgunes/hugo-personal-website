---
title: "Starting out with Hugo"
tags: ["server", "selfhost"]
date: 2020-09-03T17:27:59+07:00
---
1. Download hugo, using pacman in this example. Check official website for
   other ways.[^1]
```sh
pacman -Sy hugo
```

2. Create a new website. Configuration file uses `toml` by default, `yml`
   format is possible with appending `-f yml` to below command.
```sh
hugo new site my-website
```

3. Download a theme and initialize it.
```shell
cd my-website
git init
git submodule add git-link-here themes/theme-name
```

4. Set the theme on toml file.
```sh
printf "theme = "theme-name"\n" >> config.toml
```

6. Change your settings inside `config.yml` or `config.toml`.

7. Check your website, it will be dynamically updated whenever a file is changed.
```sh
hugo server & ; firefox http://localhost:1313/
```

9. All post should be located at the folder called `content`, create a new file.
```sh
hugo new posts/first.md
```
10. This file consists three settings, default template could be changed at
    `archetypes/default.md`.
```toml
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
---
```

## Where to head next?
You may want to refresh your markdown knowledge, possibly you wish to engineer
your own theme. Below websites should help with markdown and theme creation.
Github theme is a good start for the designing a theme, and it comes with
a video. CSS and HTML skills are another story if own theme is being designed.
* [Learn markdown: Github](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
* [Markdown More](https://learn.netlify.app/en/cont/markdown/)
* [Hugo theme creation: Youtube](https://youtu.be/wcMqrb3v2SM)
* [Hugo starter theme: Github](https://github.com/ericmurphyxyz/hugo-starter-theme)
* [Hugo guides](https://digitaldrummerj.me/series/blogging-with-hugo/)
* [CSS references](https://www.w3schools.com/CSSref/pr_border-bottom.asp)
* [Hugo's Page Resources](https://gohugo.io/content-management/page-resources/)
* [Hugo Subsections](https://github.com/guayom/hugo-sub-sections)
* [Short Codes](https://hongtaoh.com/en/2020/11/03/custom-blocks-hugo/)

<!-- https://discourse.gohugo.io/t/section-page-with-subpages-and-navigation/11902/2 -->
<!-- https://gohugo.io/templates/lists#example-project-directory -->

[^1]: [Hugo Website](https://gohugo.io/getting-started/installing/)
