---
title: "Convert markup to blogger compatible html"
date: 2020-03-10T11:12:00Z
draft: false
tags: ["blogger HTML", "html generation", "html", "blogger"]
---

## Introduction
We will be converting a `markup` file to a `blogger` compatible `html`.

## Install `pandoc` and `GitHUB html5` Template
Install `pandoc` using URL *https://pandoc.org/installing.html* . Create a workspace and clone following git repos in that workspace area.
```bash
mkdir $HOME/panDocs
cd $HOME/panDocs
git clone https://github.com/tajmone/pandoc-goodies.git
cd pandoc-goodies/templates/html5/github/src
```

### Modify above `GitHUB html5 template` source code for blogger.
Make the `padding` and `max-width` changes in `_github-markdown.scss` file as follows:

```diff
$ git diff templates/html5/github/src/_github-markdown.scss
diff --git a/templates/html5/github/src/_github-markdown.scss b/templates/html5/github/src/_github-markdown.scss
index 479b5fc..54c9abb 100644
--- a/templates/html5/github/src/_github-markdown.scss
+++ b/templates/html5/github/src/_github-markdown.scss
@@ -48,9 +48,9 @@
   word-wrap: break-word;
   box-sizing: border-box;
   min-width: 200px;
-  max-width: 980px;
+  max-width: 100%;
   margin: 0 auto;
-  padding: 45px;
+  padding: 45px 5px;

   a {
     color: #0366d6;
```

Change the css source file name in `src/css-injector.js` as follows:
```diff
$ git diff css-injector.js
diff --git a/templates/html5/github/src/css-injector.js b/templates/html5/github/src/css-injector.js
index 861494d..14b5597 100644
--- a/templates/html5/github/src/css-injector.js
+++ b/templates/html5/github/src/css-injector.js
@@ -16,7 +16,7 @@ This script will:
 */

 templateFile  = "GitHub_source.html5" // Template with CSS-injection placeholder
-cssFile       = "GitHub.min.css"      // CSS source to inject into placeholder
+cssFile       = "GitHub.css"      // CSS source to inject into placeholder
 outFile       = "../GitHub.html5"     // Final template output file
 placeHolder   = "{{CSS-INJECT}}"      // Placeholder string for CSS-injection
```

### Compile scss file.
Note: You need to install `node` on your system in order to compile.
```bash
cd pandoc-goodies/templates/html5/github/src
npm init -y
npm install node-sass
./node_modules/node-sass/bin/node-sass --source-map true --output-style=expanded --indent-type space GitHub.scss > GitHub.css
node css-injector.js
```

**Note**: You can clone pre-compiled code from `https://github.com/spareslant/markup_to_GitHUB_HTML5_and_PDF.git` as well.

* `pandoc-goodies` is to generate `html` files.

## Create a sample markup file.
Create a markup file in `$HOME/panDocs` say `markupContents.md` with following contents.

```
    # A top Level Heading
    This is a heading.

    ## Second top level heading
    Another heading

    ### Following is a Bash code.
    ```bash
    $ ls -l $HOME
    ```
    ### Following is just a text file.
    ```
    This is a plain text
    or it can be any kind of text.
    ```

    ### Some highligts in `heading`.
    ```
    This Text is colorized.
    This Text is not colorized.
    ```
```

## Create a conversion script.
Create a shell script say `generate_html_for_blogger.sh` with following contents in `$HOME/panDocs` folder.
```bash
#! /bin/bash

[[ $1 == "" ]] && echo "pass a html filename as first argument" && exit 2

#output_file="output.html"
input_file="$1"
output_file="$(basename $input_file .md).html"
output_file_code_tag_removed="$(basename $input_file .md).code_tag_removed.html"

pandoc "$input_file" -f markdown --template pandoc-goodies/templates/html5/github/GitHub.html5 --self-contained --toc --toc-depth=6 -o $output_file

cat "$output_file" | perl -wpl -e 's!\<code\>(.+?)\</code\>!\<span class=\"code"\>$1\</span\>!g' > "$output_file_code_tag_removed"
echo "file geneated: $output_file_code_tag_removed"
```
### Run the script.
```bash
$ ./generate_html_for_blogger.sh markupContent.md
[WARNING] This document format requires a nonempty <title> element.
  Defaulting to 'markupContent' as the title.
  To specify a title, use 'title' in metadata or --metadata title="...".
file geneated: markupContent.code_tag_removed.html
```
html content for blogger is generated in `markupContent.code_tag_removed.html` file. You can copy/paste the contents in blogger post without any modification.

**Note**: Blogger does **NOT** recognize `<code>` tag. Therefore we had to replace `<code>` tag with `<span>`. This also gives us the flexiblity to add custom colors in final HTML. You cannot colorize text in markup though.

## Colorizing text for blogger in Markup.
In order to colorize the texts in blogger, add  `CSS styles` as following in the markup file. We also added `<pre>` tag to colorize the text we want. Following style will also colorize text in single backquotes in markup.

```
    <style>
    /* To highlight text in Green in pre tag */
    .hl {color: #008A00;}
    /* To highlight text in Bold Green in pre tag */
    .hlb {color: #008A00; font-weight: bold;}
    /* To highlight text in Bold Red in pre tag */
    .hlbr {color:#e90001; font-weight: bold;}
    /* <code> tag does not work in blogger. Use following class with span tag */
    .code {
        color:#7e168d; 
        background: #f0f0f0; 
        padding: 0.1em 0.4em;
        font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
    }
    </style> 
    # A top Level Heading
    This is a heading.

    ## Second top level heading
    Another heading

    ### Following is a Bash code.
    ```bash
    $ ls -l $HOME
    ```
    ### Following is just a text file.
    ```
    This is a plain text
    or it can be any kind of text.
    ```

    ### Some highligts in `heading`.
    <pre>
    This Text is not colorized.
    <span class="hlb">This Text is colorized.</span>
    </pre>
```

### Run the script.
```bash
$ ./generate_html_for_blogger.sh markupContent.md
[WARNING] This document format requires a nonempty <title> element.
  Defaulting to 'markupContent' as the title.
  To specify a title, use 'title' in metadata or --metadata title="...".
file geneated: markupContent.code_tag_removed.html
```
`markupContent.code_tag_removed.html` will have contents with CSS styles applied.