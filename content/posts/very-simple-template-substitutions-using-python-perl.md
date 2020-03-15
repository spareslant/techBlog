---
title: "Very Simple Template Substitutions Using Python Perl"
date: 2020-03-15T20:50:05Z
draft: false
---
## Introduction
It is quite common to treat text files as templates which contains place holders that will be replaced by some real values. Python jinja or Ruby erb template engines are there to resolve for these kind of issues. They are quite feature rich. Following is a very simple Template substitution program in Python and Perl that does similar kind of job. Its very basic but works for simple scenarios.

## Using Python
Consider following text file `test_file.txt` with place holders in it.
```
This is a %TEST1% . Another %TEST1%  %TEST1%
Now this is a %TEST2%.
```

Following is the file `var.py` containing values of above place holder.
```ini
TEST1="test_string1"
TEST2="test_string2"
```

Following is the python program that will replace above values in `test_file.txt` from `var.py` file.
```python
#! /usr/bin/env python

import re
import fileinput
import sys
from var import *

def print_data_vars(matchObj):
    return globals()[matchObj.group(1)]

for line in fileinput.input('test_file', inplace=True):
    line = re.sub(r'%(.+?)%', print_data_vars, line)
    sys.stdout.write(line)
```
when above program is run , it will replace placeholders with values in existing `test_file.txt` file.

## Using Shell and Perl
Assuming `var.py` and `test_file.txt` contents are same as above, same effect can also be achieved using `shell` and `perl` combination as follows.
```bash
#! /usr/bin/env bash
set -a
source ./var.py
set +a
perl -i -wpl -e 's!%(.+?)%!$ENV{$1}!g' test_file
```


