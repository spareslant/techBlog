---
title: "Learn Vim the Hard Way"
date: 2021-04-19T22:52:37+01:00
draft: true
---

## Start vim without anything
* `vim -N -u NONE -i NONE` => disables everything.
* `MYVIMRC="~/LearnVIM/vimrc" HOME="~/LearnVIM" vim -N -u ./vimrc <file_to_edit>` => ignore existing vim setup and launch vanila vim with all defaults supplied by system.
