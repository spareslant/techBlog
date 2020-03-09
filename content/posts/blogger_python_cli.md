---
title: "Blogger python cli"
date: 2020-03-09T18:46:41Z
tags: ["blogger", "python", "google API", "blogger API"]
draft: false
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
We will be creating a commandLine utility in `python` to upload a post in Google blogger.

## Account Preparation
* You need to have a `google` account (gmail) (Paid account NOT required).
* Once you have a google account, Login to *https://console.developers.google.com/apis/credentials* and create a new `Project` here.
* Click on `Dashboard` on left hand column, and click on `+ ENABLE APIS AND SERVICES`.
   * Type in `blogger` in search bar, and select `Blogger API v3` and then click `ENABLE`.`
* Click on `OAuth consent screen` on left hand column, and select `User Type` to `External` and click `CREATE`.
   * On next screen type in `Application Name` as `Blogger CLI` (It can be any string).
   * Click on `Add Scope` and select both the `Blogger API v3` options and click `ADD` and click `Save`.
* Click on `Credentials` on the left hand side and you will be presented with following two options of creating credentials for Blogger API.
   * API Keys
   * OAuth 2.0 Client IDs
   * Service Accounts
   * Click on `+ CREATE CREDENTAILS` on the top and select `OAuth Client ID` from drop down menu.
      * Select `Application Type` to `other`.
      * Type in `Name` to `Python Blogger CLI` (it can be any string) and click create.
      * You will see an entry with name `Python Blogger CLI` under `OAuth 2.0 Client IDs`.
      * Download the credential files by clicking on a *down arrow* button.
      * This will be a `json` file and we need it to authenticate it with google `OAuth2.0` services.
* Login to blogger site *https://www.blogger.com*
   * Sign-in to blogger.
   * Enter a display name you like.
   * Create a `NEW BLOG` and give it a name you like in `Ttile`
   * Type in address. It has to be a unique e.g somethingrandom23455.blogspot.com and click `create blog`
   * On next screen, look at the address bar of browser, it will be like *https://www.blogger.com/blogger.g?blogID=3342324243435#allposts*
   * Note down blog ID (a string after **blogID=**, excluding #allposts) in above URL. We need this ID to put in python script.

## Python environment creation
I am using following python version.

```bash
$ python --version
Python 3.7.0
```

We will be creating a python virtual environment.
```bash
mkdir BloggerCli
cd BloggerCli
python -m venv python-2.7.0
source python-2.7.0/bin/activate
```

### Install required libraries.
```bash
pip install google-api-python-client
pip install --upgrade google-auth-oauthlib
```

### Copy credential script.
```bash
mkdir python-cli
```
Copy above downloaded credential json file in `python-cli` folder and rename it to `OAuth2.0_secret.json`.

**Note:** You can rename it any name.

### Create Python script `bloggerCli.py`

```bash
cd python-cli
```

Create following python script in `python-cli` folder.
```python
#! /Users/<username>/BloggerCli/python-2.7.0/bin/python

# https://github.com/googleapis/google-api-python-client/blob/master/docs/README.md
# https://github.com/googleapis/google-api-python-client/blob/master/docs/oauth-server.md
# https://github.com/googleapis/google-api-python-client/blob/master/docs/oauth-installed.md
# http://googleapis.github.io/google-api-python-client/docs/dyn/blogger_v3.html
# http://googleapis.github.io/google-api-python-client/docs/dyn/blogger_v3.posts.html
# https://developers.google.com/identity/protocols/googlescopes
# https://console.developers.google.com/apis/credentials
# pip install --upgrade google-auth-oauthlib

# Note: This script does NOT check the duplicate title of posts while inserting the post in Blog.

from google.oauth2 import service_account
import googleapiclient.discovery
from google_auth_oauthlib.flow import InstalledAppFlow
import os
import sys
import argparse


usage = f"{os.path.basename(__file__)} -f <html file> -t <post title> -l <label_string1> -l <label_string2> -l .. -l .."

parser = argparse.ArgumentParser(prog=os.path.basename(__file__), usage=usage, description='Upload a post to Blogger')
parser.add_argument('-f', '--uploadfile', action='store', dest='fileToPost', help='html file to post', required=True)
parser.add_argument('-l', '--labels', action='append', dest='labels', default=[], help='-l <label1> -l <label2>')
parser.add_argument('-t', '--title', action='store', dest='title', help='-t <Post Title string>', required=True)
parser.add_argument('-b', '--blogid', action='store', dest='blogid', help='-b <blogid string>', required=True)
arguments = parser.parse_args()

if len(sys.argv) == 1:
    parser.print_help()
    parser.error("You must specify command line flags as mentioned above.")


FILE_TO_POST = arguments.fileToPost
LABELS_FOR_POST = arguments.labels
POST_TITLE = arguments.title
BLOG_ID = arguments.blogid

SCOPES = ['https://www.googleapis.com/auth/blogger']
SERVICE_ACCOUNT_FILE = 'blogger-api-credentials.json'
OAUTH2_ACCOUNT_FILE = 'OAuth2.0_secret.json'
FLOW_SERVER_PORT = 9090
API_SERVICE_NAME = 'blogger'
API_SERVICE_VERSION = 'v3'

# create a API client from service account
def create_client_from_serviceAccount():
    credentials = service_account.Credentials.from_service_account_file( SERVICE_ACCOUNT_FILE, scopes=SCOPES)

    blogger = googleapiclient.discovery.build(API_SERVICE_NAME, API_SERVICE_VERSION, credentials=credentials)
    return blogger

# create a client from an AOUTH2.0 account.
def create_client_from_outhAccount():
    flow = InstalledAppFlow.from_client_secrets_file(OAUTH2_ACCOUNT_FILE, scopes=SCOPES)
    credentials = flow.run_local_server(host='localhost', port=FLOW_SERVER_PORT, authorization_prompt_message='Please visit this URL: {url}', 
                        success_message='The auth flow is complete; you may close this window.', open_browser=True)
    blogger = googleapiclient.discovery.build(API_SERVICE_NAME, API_SERVICE_VERSION, credentials=credentials)
    return blogger

def print_number_of_posts(postList):
    if not 'items' in postList:
        print("No of posts in blog = 0")
    else:    
        print("No of posts in blog = {}".format(len(postList['items'])))

# From a service account you cannot create a new post in blogger. It might need some special permissions that I am not aware of.
# However, Service account can read posts and fetch them.
# blogger = create_client_from_serviceAccount()

# Below will open a browser window to authenticate yourself.
blogger = create_client_from_outhAccount()

postList = blogger.posts().list(blogId=BLOG_ID).execute()
print_number_of_posts(postList)

print("====== Inserting a new post =======")

with open(FILE_TO_POST) as f:
    contents = f.read()

blogBody = {
    "content": contents,
    "kind": "blogger#post",
    "title": POST_TITLE,
    "labels": LABELS_FOR_POST,
        }

result = blogger.posts().insert(blogId=BLOG_ID, body=blogBody).execute()
postList = blogger.posts().list(blogId=BLOG_ID).execute()
print_number_of_posts(postList)
```

### Create a test file to be uploaded.
```bash
echo 'This is a first post' > sometext.txt
```
**Note**: You can create an HTML file as well.

### Run the script.
```bash
$ ./bloggerCli.py -f sometext.txt -t "My First Post" -b 34456769666334 -l label1 -l label2 -l label3
Please visit this URL: https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=26652434353-e9u77isb2sdsd4343sdsd343434.apps.googleusercontent.com&redirect_uri=http%3A%2F%2Flocalhost%3A9090%2F&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fblogger&state=BkaXVyiW15sdsdsdADH7sadsH8hS&access_type=offline
No of posts in blog = 0
====== Inserting a new post =======
No of posts in blog = 1
```
**Note-1**: While running above script, this will open a browser and wil ask you to login to your google account.

**Note-2**: You may need to change the *FLOW_SERVER_PORT* in above script.

### References used:
* https://github.com/googleapis/google-api-python-client/blob/master/docs/README.md
* https://github.com/googleapis/google-api-python-client/blob/master/docs/oauth-server.md
* https://github.com/googleapis/google-api-python-client/blob/master/docs/oauth-installed.md
* http://googleapis.github.io/google-api-python-client/docs/dyn/blogger_v3.html
* http://googleapis.github.io/google-api-python-client/docs/dyn/blogger_v3.posts.html
* https://developers.google.com/identity/protocols/googlescopes
* https://console.developers.google.com/apis/credentials
