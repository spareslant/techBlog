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
```
### Prepare contents to be deployed to `GITHUB`
