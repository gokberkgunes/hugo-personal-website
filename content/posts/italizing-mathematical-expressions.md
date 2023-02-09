---
title: "Italizing Mathematical Expressions"
date: 2023-02-09T22:49:14+03:00
authors: ["Gokberk Gunes", ]
tags: []
draft: true
---
When scientists write down their mathematical expressions they face the dillema
of italizing the character or not.
This article will try to guide when we should use roman characters, i.e.,
uptight font, and, when should we avoid them when we write our text.

## Mathematical Text

If we are writing an equation, a mathematical expression, we should use italic
letters when we have an intent to show there is multiplication between these
letters.

For example, consider below expression .
```latex
\( xyz \)
```
This expression means we multiply all the letters with each other.
```latex
\(x \cdot y \cdot z\)
```

Similarly, consider we are trying to write a trigonometric function as shown in
below.
```latex
\( sin(x) \)
```
Unfortunately, what we have written is equal to multiplying s, i, n and x.
```latex
\(s \cdot i \cdot n \cdot (x)\)
```

Let's give a few more wrong examples before delving into correct writing style.
```latex
  \begin{gather}
    S=\sum_{i=1}^{8}A_i\\
    S=\sum_{i=1}^{8}B_i\\
    S=\sum_{i=1}^{8}C_i S=\sum_{i=1}^{8}D_i
  \end{gather}
```


