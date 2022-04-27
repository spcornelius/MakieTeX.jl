# MakieTeX

MakieTeX allows you to draw and visualize arbitrary TeX documents in Makie!  You can insert anything from a single line of math to a large and complex TikZ diagram.


MakieTeX works by compiling a latex document and transforming it to a renderable
svg for CairoMakie or raster image for GLMakie.


```julia
using GLMakie, Makie, MakieTeX
fig = Figure(resolution = (400, 300));
ax = Axis(fig[1, 1]);
lines!(rand(10), color = 1:10);
tex = LTeX(fig[2, 1], raw"\int \mathbf E \cdot d\mathbf a = \frac{Q_{encl}}{4\pi\epsilon_0}", scale=2);
fig
```
![ltex](https://user-images.githubusercontent.com/10947937/110216157-d1d87d00-7ead-11eb-8507-62ddcff2a841.png)

```julia
using GLMakie, Makie, MakieTeX, LaTeXStrings
fig, ax, p = teximg(L"\hat {f}(\xi )=\int _{-\infty }^{\infty }f(x)\ e^{-2\pi ix\xi }~ dx", scale=10)
# Don't stretch the text
ax.aspect = DataAspect()
fig
```
![teximg](https://user-images.githubusercontent.com/10947937/110216144-c5542480-7ead-11eb-9753-7ff215e36056.png)

There is a way to integrate LTeX into a legend, but it's pretty hacky now.  Ask on `#makie` in the JuliaLang Slack if you want to know.
![legendtex](https://user-images.githubusercontent.com/32143268/79641479-6adaa880-81b5-11ea-8138-4d6054ccfa6d.png)

## Including full LaTeX documents

With the `TeXDocument` struct, you can feed in a String which contains a full LaTeX document, and show it at real scale in Makie!

This example is from [Texample.net](https://texample.net/tikz/examples/title-graphics/)
```julia
using MakieTeX, CairoMakie, Makie
td = TeXDocument(raw"""
\documentclass{standalone}
\usepackage{tikz}
\usetikzlibrary{trees,snakes}
\begin{document}
\pagestyle{empty}
\tikzstyle{level 1}=[sibling angle=120]
\tikzstyle{level 2}=[sibling angle=60]
\tikzstyle{level 3}=[sibling angle=30]
\tikzstyle{every node}=[fill]
\tikzstyle{edge from parent}=[snake=expanding waves,segment length=1mm,
                              segment angle=10,draw]
\begin{tikzpicture}[grow cyclic,shape=circle,very thick,level distance=13mm,
                    cap=round]
\node {} child [color=\A] foreach \A in {red,green,blue}
    { node {} child [color=\A!50!\B] foreach \B in {red,green,blue}
        { node {} child [color=\A!50!\B!50!\C] foreach \C in {black,gray,white}
            { node {} }
        }
    };
\end{tikzpicture}
\end{document}
""")
fig = Figure()
lt = LTeX(fig[1, 1], td; tellheight=false)
ax = Axis(fig[1, 2])
lines!(ax, rand(10); color = 1:10)
fig
```
![makietex](https://user-images.githubusercontent.com/32143268/165130481-53ee0fe1-4c70-4453-b430-7a2ad37082f8.png)

If you are


## Installation

In order to run the latest code, you should check out the master branch of this repo and the master branch of Makie.  You can do this by:

```
]add Makie#master MakieTeX#master
```


## The rendering pipeline

The standard rendering pipeline works as follows: the string is converted in to a TeXDocument, which is compiled to pdf (by default, using system lualatex).  The pdf file is then cropped using the `pdfcrop` Perl script (this requires Perl_jll and Ghostscript_jll).

This cropped PDF is converted to svg via `pdftocairo` (provided by Poppler_jll).  Then, the svg is processed by librsvg.  From here, if CairoMakie is the backend, we can render directly to the surface.  For any other backend, we render an ARGB image and then plot that.

Thus, you only need a TeX engine, preferably LuaTeX, installed on your system.
