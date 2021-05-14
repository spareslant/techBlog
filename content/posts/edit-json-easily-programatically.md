---
title: "Edit Json Easily Programatically"
date: 2021-05-14T06:58:28+01:00
draft: false
tags: ["python", "json", "yaml"]
---

## Introduction
Often I had to manipulate the JSON/YAML files. I created a utility that can help make changes in JSON/YAML easily.

### Get the code.
```bash
git clone https://github.com/spareslant/json_manipulation.git
cd json_manipulation
```
### Python version used
```bash
$ python --version
Python 3.9.0
```

### How to Run:
#### `input_data_1.json` contents
```json
{
    "data": [
        {
            "actions": [
                {
                    "link": "http://www.facebook.com/X999/posts/Y999",
                    "name": "Comment"
                },
                {
                    "link": "http://www.facebook.com/X999/posts/Y999",
                    "name": "Like"
                }
            ],
            "created_time": "2010-08-02T21:27:44+0000",
            "from": {
                "id": "X12",
                "name": "Tom Brady"
            },
            "id": "X999_Y999",
            "message": "Looking forward to 2010!",
            "type": "status",
            "updated_time": "2010-08-02T21:27:44+0000"
        },
        {
            "actions": [
                {
                    "link": "http://www.facebook.com/X998/posts/Y998",
                    "name": "Comment"
                },
                {
                    "link": "http://www.facebook.com/X998/posts/Y998",
                    "name": "Like"
                }
            ],
            "created_time": "2010-08-02T21:27:44+0000",
            "from": {
                "id": "X18",
                "name": "Peyton Manning"
            },
            "id": "X998_Y998",
            "message": "Where's my contract?",
            "type": "status",
            "updated_time": "2010-08-02T21:27:44+0000"
        }
    ],
    "job": {
        "Fulltimejob": "Lawyer",
        "hobbyjob": [
            "Gardening",
            "InteriorDesign"
        ]
    },
    "name": "Anonymous"
}
```

### Generate Flat JSON structure
```bash
./generate_flat_json.py input_data_1.json > input_data_1_flat.json
```

##### `input_data_1_flat.json` file contents

```json
{
  "data/0/actions/0/link": "http://www.facebook.com/X999/posts/Y999",
  "data/0/actions/0/name": "Comment",
  "data/0/actions/1/link": "http://www.facebook.com/X999/posts/Y999",
  "data/0/actions/1/name": "Like",
  "data/0/created_time": "2010-08-02T21:27:44+0000",
  "data/0/from/id": "X12",
  "data/0/from/name": "Tom Brady",
  "data/0/id": "X999_Y999",
  "data/0/message": "Looking forward to 2010!",
  "data/0/type": "status",
  "data/0/updated_time": "2010-08-02T21:27:44+0000",
  "data/1/actions/0/link": "http://www.facebook.com/X998/posts/Y998",
  "data/1/actions/0/name": "Comment",
  "data/1/actions/1/link": "http://www.facebook.com/X998/posts/Y998",
  "data/1/actions/1/name": "Like",
  "data/1/created_time": "2010-08-02T21:27:44+0000",
  "data/1/from/id": "X18",
  "data/1/from/name": "Peyton Manning",
  "data/1/id": "X998_Y998",
  "data/1/message": "Where's my contract?",
  "data/1/type": "status",
  "data/1/updated_time": "2010-08-02T21:27:44+0000",
  "job/Fulltimejob": "Lawyer",
  "job/hobbyjob/0": "Gardening",
  "job/hobbyjob/1": "InteriorDesign",
  "name": "Anonymous"
}
```

### Make changes to above file/conents. like delete unwanted lines or change or add new ones. Following is the diff
```diff
diff -b -u flat_input_data_1.json flat_input_data_1.json.new                                                                                     1 ↵
--- flat_input_data_1.json      2021-05-14 06:33:56.000000000 +0100
+++ flat_input_data_1.json.new  2021-05-14 06:32:50.000000000 +0100
@@ -4,16 +4,18 @@
   "data/0/actions/1/link": "http://www.facebook.com/X999/posts/Y999",
   "data/0/actions/1/name": "Like",
   "data/0/created_time": "2010-08-02T21:27:44+0000",
-  "data/0/from/id": "X12",
+  "data/0/from/id": "Y9999",
   "data/0/from/name": "Tom Brady",
   "data/0/id": "X999_Y999",
   "data/0/message": "Looking forward to 2010!",
+  "data/0/urgentmessage": "Not at the moment",
   "data/0/type": "status",
   "data/0/updated_time": "2010-08-02T21:27:44+0000",
   "data/1/actions/0/link": "http://www.facebook.com/X998/posts/Y998",
   "data/1/actions/0/name": "Comment",
   "data/1/actions/1/link": "http://www.facebook.com/X998/posts/Y998",
   "data/1/actions/1/name": "Like",
+  "data/1/actions/2/link": "bogus.com",
   "data/1/created_time": "2010-08-02T21:27:44+0000",
   "data/1/from/id": "X18",
   "data/1/from/name": "Peyton Manning",
@@ -22,7 +24,9 @@
   "data/1/type": "status",
   "data/1/updated_time": "2010-08-02T21:27:44+0000",
   "job/Fulltimejob": "Lawyer",
+  "job/anotherJob": "consultant",
   "job/hobbyjob/0": "Gardening",
   "job/hobbyjob/1": "InteriorDesign",
-  "name": "Anonymous"
+  "name": "Anonymous",
+  "function": "Programming"
```

### Re-generate JSON from flat JSON
```bash
./recreate_json_from_flat.py flat_input_data_1.json.new | python -m json.tool --sort-keys > recreated_data_1.json
```
**Note**: Input file was also sorted with above `-m json.tool`. Sorting is not required at all. We are sorting above json so that diff can be viewed comfortably.

#### `recreated_data_1.json` file contents
```json
{
    "data": [
        {
            "actions": [
                {
                    "link": "http://www.facebook.com/X999/posts/Y999",
                    "name": "Comment"
                },
                {
                    "link": "http://www.facebook.com/X999/posts/Y999",
                    "name": "Like"
                }
            ],
            "created_time": "2010-08-02T21:27:44+0000",
            "from": {
                "id": "Y9999",
                "name": "Tom Brady"
            },
            "id": "X999_Y999",
            "message": "Looking forward to 2010!",
            "type": "status",
            "updated_time": "2010-08-02T21:27:44+0000",
            "urgentmessage": "Not at the moment"
        },
        {
            "actions": [
                {
                    "link": "http://www.facebook.com/X998/posts/Y998",
                    "name": "Comment"
                },
                {
                    "link": "http://www.facebook.com/X998/posts/Y998",
                    "name": "Like"
                },
                {
                    "link": "bogus.com"
                }
            ],
            "created_time": "2010-08-02T21:27:44+0000",
            "from": {
                "id": "X18",
                "name": "Peyton Manning"
            },
            "id": "X998_Y998",
            "message": "Where's my contract?",
            "type": "status",
            "updated_time": "2010-08-02T21:27:44+0000"
        }
    ],
    "function": "Programming",
    "job": {
        "Fulltimejob": "Lawyer",
        "anotherJob": "consultant",
        "hobbyjob": [
            "Gardening",
            "InteriorDesign"
        ]
    },
    "name": "Anonymous"
}
```

#### Following is the diff between original input json file and newly created file
```diff
diff -b -u input_data_1.json recreated_data_1.json
--- input_data_1.json   2021-05-14 06:26:45.000000000 +0100
+++ recreated_data_1.json       2021-05-14 06:36:45.000000000 +0100
@@ -13,13 +13,14 @@
             ],
             "created_time": "2010-08-02T21:27:44+0000",
             "from": {
-                "id": "X12",
+                "id": "Y9999",
                 "name": "Tom Brady"
             },
             "id": "X999_Y999",
             "message": "Looking forward to 2010!",
             "type": "status",
-            "updated_time": "2010-08-02T21:27:44+0000"
+            "updated_time": "2010-08-02T21:27:44+0000",
+            "urgentmessage": "Not at the moment"
         },
         {
             "actions": [
@@ -30,6 +31,9 @@
                 {
                     "link": "http://www.facebook.com/X998/posts/Y998",
                     "name": "Like"
+                },
+                {
+                    "link": "bogus.com"
                 }
             ],
             "created_time": "2010-08-02T21:27:44+0000",
@@ -43,8 +47,10 @@
             "updated_time": "2010-08-02T21:27:44+0000"
         }
     ],
+    "function": "Programming",
     "job": {
         "Fulltimejob": "Lawyer",
+        "anotherJob": "consultant",
         "hobbyjob": [
             "Gardening",
             "InteriorDesign"
```

#### Recreated json file may have some `null` elements in it, depending on what changes were introduced in flat json. `remove_null_from_json.py` removes those `null` values if not required.


