---
title: "Deploy HugoSite to Github"
date: 2019-08-03T19:01:43+01:00
draft: true
---
<style type="text/css">
.hl {color: #f155f1;}
.hlb {color: #f155f1; font-weight: bold;}
.hlbr {color:#e90001; font-weight: bold;}
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
#### Remove old contents (just to start from fresh)
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

$ rm -rf ../spareslant.github.io
```
#### clone remote `spareslant.github.io` repo.
```bash
cd ../spareslant.github.io
git clone git clone https://github.com/spareslant/spareslant.github.io.git
```
#### Build site to be deployed.
```bash
$ cd MyTechnialBlog/
$ hugo -d ../spareslant.github.io/
```
#### commit new changes in `../spareslant.github.io`.
```bash
$ cd ../spareslant.github.io/
$ git add .
$ git commit
```

#### Publish site by pushing the contents to repo.
```bash
$ git push origin master
```


