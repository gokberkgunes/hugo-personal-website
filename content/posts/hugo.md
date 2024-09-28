---
title: "Starting out with Hugo"
tags: ["self-hosting", "hugo"]
date: 2022-06-03T17:27:59+07:00
---
## Introduction
In this _Starting out with Hugo_ guide, we will be setting Hugo, which
self-claims that Hugo is the world's fastest framework for building websites.
For us, Hugo's main advantage is that it eases whole website generation and
maintainance. 

The main content are written effortlessly by using markdown. Markdown usage
totally cuts the requirement of HTML in the website. Still, for some further
options, such as embedding videos or svg files, we might require writing small
codes. These codes incorporate Hugo's own syntax in Go though they are
straightforward.

The selection and usage of Go in Hugo makes it fast and safe. We can have
a live demo that shows our changes on the fly. When we are satisfied,
a second-long compilation will give us our website in HTML, CSS and if needed
JS. 

At this point, I should note that I wrote this website is written using Hugo
without a single line of JS, yet the flexibility and features I have are
mindblowing. For example, I have syntax highlighting, selection of code-blocks
with a single click, embedded-video, able to write/render Latex code. Please
note that I do all of these inside markdown using aferomentioned quick codes
and so on. Please check the [source code](https://github.com/gokberkgunes/hugo-personal-website) of my website if you want to see the
options.

## Installation and Preperation
Let's start with download Hugo. I used pacman in this example since I use an
arch-based GNU/Linux distribution. Of course we may find the other ways
described in the [official website].
```sh
pacman -Sy hugo
```

Create a new website. The configuration file uses file format of toml by
default, if we are more accustomed to yaml, we can set it up by adding
`--format yaml` to the below command. Nonetheless, let's continue our guide
with toml given that it is the default.
```sh
hugo new site $website_name
```

Download a theme and initialize it. Themes may be found on the [official
website](https://themes.gohugo.io/), and should be installed as a git
submodule. You should fill `$git_link` and `$theme_name` below using the theme.
```sh
cd my-website
git init
git submodule add $git_link themes/$theme_name
```

Also, we should tell Hugo that we have a new theme. Simply, let's modify our
configuration file to represent this.
```txtt
config.toml
```
```toml
theme = "$theme_name"
```

Now, we can live-demo our website and see how great it looks like. From this
point on, whenever we modify any file our website will get dynamically updated.
```sh
hugo server &
firefox http://localhost:1313/
```

Finally, we can create out new post, usually posts are located under the folder
`content`. Below, the command to create a new post is given.
```sh
hugo new posts/first.md
```

Above command gets very useful when we introduce user-defaults to Hugo in
`archetypes/default.md`. Currently, we have only three settings in the default
template.
```txtt
archetypes/default.md
```
```go
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

<!-- [^1]: [Hugo Website](https://www.website.com) -->
