---
title: "Json Full Paths From Root Node to Leaf"
date: 2020-03-16T22:09:29Z
draft: true
tags: ["json", "python", "recursion"]
---

<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
Note: draft: true above. This prevents publishing it to GitHUB.
--->
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

## Introduction:
Often there is a requirement to create full paths from root node to leaf node in JSON file. Following python program will generate these paths.

Sample JSON file `sample_json_file.json` is below:
```json
{
   "data": [
      {
         "id": "X999_Y999",
         "from": {
            "name": "Tom Brady", "id": "X12"
         },
         "message": "Looking forward to 2010!",
         "actions": [
            {
               "name": "Comment",
               "link": "http://www.facebook.com/X999/posts/Y999"
            },
            {
               "name": "Like",
               "link": "http://www.facebook.com/X999/posts/Y999"
            }
         ],
         "type": "status",
         "created_time": "2010-08-02T21:27:44+0000",
         "updated_time": "2010-08-02T21:27:44+0000"
      },
      {
         "id": "X998_Y998",
         "from": {
            "name": "Peyton Manning", "id": "X18"
         },
         "message": "Where's my contract?",
         "actions": [
            {
               "name": "Comment",
               "link": "http://www.facebook.com/X998/posts/Y998"
            },
            {
               "name": "Like",
               "link": "http://www.facebook.com/X998/posts/Y998"
            }
         ],
         "type": "status",
         "created_time": "2010-08-02T21:27:44+0000",
         "updated_time": "2010-08-02T21:27:44+0000"
      }
   ],
  "name": "Anonymous",
  "job": { "hobbyjob": [ "Gardening", "InteriorDesign"], "Fulltimejob": "Lawyer" }
}
```

Python program to create full paths from above JSON file
```python
#! /usr/bin/env python

# This programm will create paths from root nodes to leafnodes along with values from any json file or structure.

import json
import pprint


json_data = open('sample_json_file.json', 'r').read()
json_dict = json.loads(json_data)

stack = []
final_dict = {}

def do_walk(datadict):

    if isinstance(datadict, dict):
        for key, value in datadict.items():
            stack.append(key)
            #print("/".join(stack))
            if isinstance(value, dict) and len(value.keys()) == 0:
                final_dict["/".join(stack)] = "EMPTY_DICT"
            if isinstance(value, list) and len(value) == 0:
                final_dict["/".join(stack)] = 'EMPTY_LIST'
            if isinstance(value, dict):
                do_walk(value)
            if isinstance(value, list):
                do_walk(value)
            if isinstance(value, unicode):
                final_dict["/".join(stack)] = value
            stack.pop()

    if isinstance(datadict, list):
        n = 0
        for key in datadict:
            stack.append(str(n))
            n = n + 1
            if isinstance(key, dict):
                do_walk(key)
            if isinstance(key, list):
                do_walk(key)
            if isinstance(key, unicode):
                final_dict["/".join(stack)] = key
            stack.pop()


do_walk(json_dict)
pprint.pprint(final_dict)
```
Below is the result:
```bash
# python create_json_paths.py
{u'data/0/actions/0/link': u'http://www.facebook.com/X999/posts/Y999',
 u'data/0/actions/0/name': u'Comment',
 u'data/0/actions/1/link': u'http://www.facebook.com/X999/posts/Y999',
 u'data/0/actions/1/name': u'Like',
 u'data/0/created_time': u'2010-08-02T21:27:44+0000',
 u'data/0/from/id': u'X12',
 u'data/0/from/name': u'Tom Brady',
 u'data/0/id': u'X999_Y999',
 u'data/0/message': u'Looking forward to 2010!',
 u'data/0/type': u'status',
 u'data/0/updated_time': u'2010-08-02T21:27:44+0000',
 u'data/1/actions/0/link': u'http://www.facebook.com/X998/posts/Y998',
 u'data/1/actions/0/name': u'Comment',
 u'data/1/actions/1/link': u'http://www.facebook.com/X998/posts/Y998',
 u'data/1/actions/1/name': u'Like',
 u'data/1/created_time': u'2010-08-02T21:27:44+0000',
 u'data/1/from/id': u'X18',
 u'data/1/from/name': u'Peyton Manning',
 u'data/1/id': u'X998_Y998',
 u'data/1/message': u"Where's my contract?",
 u'data/1/type': u'status',
 u'data/1/updated_time': u'2010-08-02T21:27:44+0000',
 u'job/Fulltimejob': u'Lawyer',
 u'job/hobbyjob/0': u'Gardening',
 u'job/hobbyjob/1': u'InteriorDesign',
 u'name': u'Anonymous'}
```


