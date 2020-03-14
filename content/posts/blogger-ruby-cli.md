---
title: "Blogger ruby cli"
date: 2020-03-10T00:34:42Z
draft: false
tags: ["blogger", "python", "google API", "blogger API"]
---

## Introduction
We will be creating a commandLine utility in `ruby` to upload a post in Google blogger.

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
   * Type in `Name` to `CLI Utility` (it can be any string) and click create.
   * You will see an entry with name `CLI Utility` under `OAuth 2.0 Client IDs`.
   * Download the credential files by clicking on a *down arrow* button.
   * This will be a `json` file and we need it to authenticate it with google `OAuth2.0` services.

* Login to blogger site *https://www.blogger.com*
   * Sign-in to blogger.
   * Enter a display name you like.
   * Create a `NEW BLOG` and give it a name you like in `Ttile`
   * Type in address. It has to be a unique e.g somethingrandom23455.blogspot.com and click `create blog`
   * On next screen, look at the address bar of browser, it will be like *https://www.blogger.com/blogger.g?blogID=3342324243435#allposts*
   * Note down blog ID (a string after **blogID=**, excluding #allposts) in above URL. We need this ID to put in ruby script.

## Ruby environment creation
I am using following ruby version.

```bash
$ ruby --version
ruby 2.7.0p0 (2019-12-25 revision 647ee6f091) [x86_64-darwin19]
```

We will be creating a python virtual environment.
```bash
mkdir BloggerCli
cd BloggerCli
mkdir ruby-cli
```

### Install required libraries.
Create a file `ruby-cli/Gemfile` with following contents.

```bash
source 'https://rubygems.org'
gem 'google-api-client'
gem 'googleauth'
gem 'launchy'
gem 'slop'
```


Run following commands to install above gems.
```bash
cd ruby-cli
bundler
```

### Copy credential script.
Copy above downloaded credential json file in `ruby-cli` folder and rename it to `OAuth2.0_secret.json`.

**Note:** You can rename it any name.

### Create Ruby script `bloggerCli.rb`

Create following ruby script in `ruby-cli` folder.
```ruby
#! /usr/bin/ruby

# https://github.com/googleapis/google-api-ruby-client
# https://github.com/googleapis/google-api-ruby-client/blob/master/docs/oauth-server.md
# https://github.com/googleapis/google-api-ruby-client/blob/master/docs/oauth-installed.md
# https://github.com/googleapis/google-auth-library-ruby
# https://googleapis.dev/ruby/google-api-client/latest/Google/Apis/BloggerV3/BloggerService.html
# https://developers.google.com/identity/protocols/googlescopes
# https://console.developers.google.com/apis/credentials
# https://www.rubydoc.info/gems/slop/4.8.0

# Note: This script does NOT check the duplicate title of posts while inserting the post in Blog.

require 'googleauth'
require 'google/apis/blogger_v3'
require 'googleauth/stores/file_token_store'
require 'launchy'
require 'slop'

opts = Slop::Options.new
opts.banner = "usage: #{File.basename(__FILE__)} -f <html file> -t <post title> -l <label_string1> -l <label_string2> -l .. -l .."
opts.string '-f', '--uploadfile', 'html file', required: true
opts.string '-t', '--title', 'Post Title', required: true
opts.string '-e', '--email', 'Account email id', required: true
opts.string '-b', '--blogid', 'Blog id', required: true
opts.array '-l', '--labels', 'comma separated labels', delimiter: ','

parser = Slop::Parser.new(opts)
begin
    result = parser.parse(ARGV)
rescue Slop::MissingRequiredOption, Slop::MissingRequiredOption
    puts opts
    raise
end

BLOG_ID = result[:blogid]
ACCOUNT_EMAIL = result[:email]
FILE_TO_POST = result[:uploadfile]
POST_TITLE = result[:title]
LABELS_FOR_POST = result[:labels]

SCOPE = 'https://www.googleapis.com/auth/blogger'
OOB_URI = 'urn:ietf:wg:oauth:2.0:oob'
SERVICE_ACCOUNT_FILE = 'blogger-api-credentials.json'
OAUTH2_ACCOUNT_FILE = 'OAuth2.0_secret.json'


# create an API client from service account
def create_client_from_serviceAccount(scope, service_account_cred)
    authorizer = Google::Auth::ServiceAccountCredentials.make_creds(
                    json_key_io: File.open(service_account_cred), scope: scope)

    authorizer.fetch_access_token!
    blogger = Google::Apis::BloggerV3::BloggerService.new
    blogger.authorization = authorizer
    return blogger
end

# create an API client from an AOUTH2.0 account.
# Following function will create a new file called tokens.yaml
def create_client_from_outhAccount(scope, oob_uri, user_id, oauth_cred_file)
    #oob_uri = 'urn:ietf:wg:oauth:2.0:oob'
    #user_id = 'eyemole@gmail.com'
    client_id = Google::Auth::ClientId.from_file(oauth_cred_file)
    token_store = Google::Auth::Stores::FileTokenStore.new(:file => 'tokens.yaml')
    authorizer = Google::Auth::UserAuthorizer.new(client_id, scope, token_store)
    credentials = authorizer.get_credentials(user_id)
    if credentials.nil?
        url = authorizer.get_authorization_url(base_url: oob_uri )
        #Launchy.open(url)
        puts "Open this URL in Browser and enter the code you got from browser below"
        puts "URL: #{url}"
        print "enter the code you got from browser here and press Enter: "
        # code = gets
        code = STDIN.gets.chomp
        credentials = authorizer.get_and_store_credentials_from_code(user_id: user_id, code: code, base_url: oob_uri)
    end
    blogger = Google::Apis::BloggerV3::BloggerService.new
    blogger.authorization = credentials
    return blogger
end

# From a service account you cannot create a new post in blogger. It might need some special permissions that I am not aware of.
# However, Service account can read posts and fetch them.
# blogger = create_client_from_serviceAccount(scope, SERVICE_ACCOUNT_FILE)

# Below will open a browser window to authenticate yourself.
blogger = create_client_from_outhAccount(SCOPE, OOB_URI, ACCOUNT_EMAIL, OAUTH2_ACCOUNT_FILE)

blog = blogger.get_blog(BLOG_ID)
# puts "blog URL = #{blog.url}"
puts "Total posts in this blog = #{blog.posts.total_items}"

puts "========== Inserting a new Post ========"

contents = File.read(FILE_TO_POST)

new_post = Google::Apis::BloggerV3::Post.new
new_post.kind = "blogger#post"
new_post.title = POST_TITLE
new_post.content = contents
new_post.labels = LABELS_FOR_POST
blogger.insert_post(BLOG_ID, new_post)

blog = blogger.get_blog(BLOG_ID)
puts "Total posts in this blog = #{blog.posts.total_items}"

#posts.items.each do |post|
#    puts post.to_json
#end
```

### Create a test file to be uploaded.
```bash
echo 'This is a first post' > sometext.txt
```
**Note**: You can create an HTML file as well.

### Run the script.
```bash
$ ruby bloggerCli.rb -f sometext.txt -t "My First Post" -e "<userAccount>@gmail.com" -b "3434533535534" -l label1,label2
enter the code you got from browser here and press Enter: 6/xQEBsJRhTZfthVP9ZSNsasaa7Y78HSDj2sd7aG
Total posts in this blog = 0
========== Inserting a new Post ========
Total posts in this blog = 1
```
**Note-1**: While running above script, this will open a browser and wil ask you to login to your google account.

**Note-2**: Above run will create a `tokens.yaml` file. On subsequent runs, browser window will not open due to presence of tokens.yaml file. If you remove this file, then you will have to authenticate again in the browser.

### References used:
* https://github.com/googleapis/google-api-ruby-client
* https://github.com/googleapis/google-api-ruby-client/blob/master/docs/oauth-server.md
* https://github.com/googleapis/google-api-ruby-client/blob/master/docs/oauth-installed.md
* https://github.com/googleapis/google-auth-library-ruby
* https://googleapis.dev/ruby/google-api-client/latest/Google/Apis/BloggerV3/BloggerService.html
* https://developers.google.com/identity/protocols/googlescopes
* https://console.developers.google.com/apis/credentials
* https://www.rubydoc.info/gems/slop/4.8.0

