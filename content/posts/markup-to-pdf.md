---
title: "Markup to Pdf/HTML"
date: 2019-07-27T20:39:13+01:00
draft: false
tags: ["pdf generation", "html generation"]
---

<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
--->
<style>
.hl {color: #f155f1;}
.hlb {color: #f155f1; font-weight: bold;}
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {color:#f20101; background: #f0f0f0; padding: 0.2em;    
</style>

## Introduction
This documents shows the various ways to generate PDF and self contained HTML files. 

## Using VSCode extension.
Install VSCode and install extension `Markdown PDF`. Write a new markup page in VSCode. When done, right click on the editor and you will see the options to export to PDF and HTML.

* **pros**
    1. Easy installation and generation of PDF
    2. Line wraps in code blocks.


* **cons**
    1. Not very eye pleasing.
    2. Headings like h1, h2 are not consistent in rendered pdf.

## Using pandoc/latex/Tex
Install `pandoc`, `macTex` and `TeX Live`. Create a workspace are and clone following two git repos in that workspace area.
```bash
mkdir $HOME/panDocs
cd $HOME/panDocs
git clone https://github.com/Wandmalfarbe/pandoc-latex-template.git
git clone https://github.com/tajmone/pandoc-goodies.git
```
* `pandoc-goodies` is to generate `html` files.
* `pandox-latex-template` is to generate `pdf` files.

Create an markup file in `$HOME/panDocs` say `input.md`.

### Run following command to generate `PDF`
_PDF generation_
```bash
pandoc input.md -f markdown --template pandoc-latex-template/eisvogel.tex --listings -o output.pdf --highlight-style pygments  -V lang=en-GB -V listings-disable-line-numbers=true --toc --toc-depth 6
```
Above command will generate an pdf file of name `output.pdf`

### Run following command to generate `HTML`:
_Self contained HTML_
```bash
pandoc input.md -f markdown --template pandoc-goodies/templates/html5/github/GitHub.html5 --self-contained --toc --toc-depth=6 -o output.html
```
Above command will generate a self contained `HTML` file of name `output.html`