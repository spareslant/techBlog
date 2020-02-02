---
title: "Deploy HugoSite to Github"
date: 2019-08-03T19:01:43+01:00
draft: false
tags: ["hugo", "github pages deployment", "blog", "blog publishing"]
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

### Run HTTP server to serve file locally to check the contents
```bash
$ hugo server -D
```
### Do following change in md files that need to be published.
<pre>
change <span class="hlbr">draft: true</span> to <span class="hlb">draft: false</span>
</pre>

### Run HTTP server in prod mode and make sure all required files are being served.
```bash
$ hugo server
```
### `Git` commit the changes
```bash
$ git add <files>
$ git commit
$ git push origin master
```
### Prepare contents to be deployed to `GITHUB`
```bash
$ cd MyTechnialBlog/

$ ls -og
total 8
drwxr-xr-x  3     96 17 Jul 21:25 archetypes
-rw-r--r--  1   2067 17 Jul 21:40 config.toml
drwxr-xr-x  3     96 27 Jul 21:29 content
drwxr-xr-x  2     64 17 Jul 21:25 data
drwxr-xr-x  2     64 17 Jul 21:25 layouts
drwxr-xr-x  3     96 17 Jul 21:35 resources
drwxr-xr-x  2     64 17 Jul 21:25 static
drwxr-xr-x  3     96 17 Jul 21:27 themes

```
### clone remote `spareslant.github.io` repo.
```bash
cd ../spareslant.github.io
git clone https://github.com/spareslant/spareslant.github.io.git
```
### Build site to be deployed.
```bash
$ cd MyTechnialBlog/
$ hugo -d ../spareslant.github.io/
```
### commit new changes in `../spareslant.github.io`.
```bash
$ cd ../spareslant.github.io/
$ git add .
$ git commit
```

### Publish site by pushing the contents to repo.
```bash
$ git push origin master
```
### check published changes at following link
[https://spareslant.github.io/](https://spareslant.github.io/)


