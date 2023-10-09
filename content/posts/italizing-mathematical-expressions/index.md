---
title: "Italicizing and Uprighting Mathematical Expressions"
date: 2023-02-09T22:49:14+03:00
authors: ["Gokberk Gunes", ]
tags: [latex, typography]
draft: false
---

When we write down mathematical expressions we commonly face the dilemma of
italicizing the character or not. This article will guide us when to use
upright characters and when to not.


## Mathematical Text
### Common Issues While Writing Mathematical Expressions
If we are writing an equation, an expression, we probably use italic letters on
a computerized environment. This style commonly means that if there is no
symbol between the letters multiplication symbol is assumed to be there.

For example, consider below expression. It's very likely that everyone would
consider right-hand-side of the equation is as the right-hand side. This is
very natural way to type in mathematical context, right?
```latex
2xyz = 2 \cdot x \cdot y \cdot z
```
Now consider we are trying to write a trigonometric function as shown
in below. Unfortunately, what we have written on the left hand side could be
interpreted as the multiplication of all the letters.
```latex
sin(x) = s \cdot i \cdot n \cdot \left(x\right)
```

Few more **wrong examples** are below.
```latex
\begin{align}
Re &= R \cdot e \\
St &= S \cdot t \\
mL &= m \cdot L \\
dx &= d \cdot x \\
\partial x &= \partial \cdot x \\
\mathbf{u}_{max} &= u_{m \cdot a \cdot x}
\end{align}
```
### Neatly Written Mathematical Expressions
As demonstrated earlier, if we are writing objects with specific meanings, in
italic type they will not carry the correct meanings. Also, for those single
letter constants `i`, `j` as in complex units, we will avoid confusion with
their counterparts, running indices. Therefore, we better write them in upright
mode, and we should **be consistent** about the way we write them.
```latex
\begin{align}
\text{Euler's number: }&\mathrm{e} &\text{\\mathrm\{e\}} \\
\text{Imaginary indices: }&\mathrm{i\ j\ k} &\text{\\mathrm\{i\\ j\\ k\}}\\
\text{Sine function: }&\sin(x) &\text{\\sin(x)}\\
\text{Reynolds' number: }&\mathrm{Re} &\text{\\mathrm\{Re\}}\\
\text{Maximum velocity: }& \mathbf{u}_\mathrm{max} &\text{\\mathbf\{u\}\_\\mathrm\{max\}}\\
\text{Differential operator: }& \mathrm{d}x &\text{\\mathrm\{d\}x} \\
\text{Shear strength tensor: }& \mathbf{\tau}_{ij} &\text{\\mathbf\{\\tau\}\_\{ij\}} \\
\text{Mililiters: }& \mathrm{mL} &\text{\\mathrm\{mL\}} \\

\end{align}
```
**NOTE:** We should keep running indices, such as i, j, k, in italic form as
they are individual variables. If these indices are constant, it's better to keep them upright.

**NOTE:** It's common to write tensors in boldface.

**NOTE:** Prefer using `siunitx` package when writing units.

### Upright Greek Letters and Upright Special Letters
#### XeLaTex and LuaLatex
Note that Greek letters and special letters like partial operator is not
as straightforward as shown above. In order to get them right, we should follow
this answer on Stack Exchange[^1].
```text
\usepackage{unicode-math}

\symup{\pi} r^2  = Area of circle
\symup{\partial}x = Partial operator on x
\symup{\delta}_{ij} = Kronecker's delta
```
#### Pdftex
The upright Greek and special letters for `pdftex` gets hacky. There could me
many solutions given the flexibility of Latex. You may find two solutions:
1. [More flexible, but little strange looking.](https://tex.stackexchange.com/a/170567)
2. [Better looking one.](https://tex.stackexchange.com/a/230220)



## Further Reading and References

1. [Johannes Küster's presentation](https://www.youtube.com/watch?v=UkCjcMfV3EE)
2. [Upright Greek font fitting to Computer Modern](https://tex.stackexchange.com/questions/145926/upright-greek-font-fitting-to-computer-modern)
3. [Should subscripts in math mode be upright?](https://tex.stackexchange.com/questions/33120/should-subscripts-in-math-mode-be-upright)
4. [NIST: Typefaces for Symbols in Scientific Manuscripts](https://physics.nist.gov/cuu/pdf/typefaces.pdf)

[^1]: [Unicode-Math Suggestion on Stack Exchange](https://tex.stackexchange.com/questions/145926/upright-greek-font-fitting-to-computer-modern/431804#431804)
