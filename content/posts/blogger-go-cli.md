---
title: "Blogger Go Cli"
date: 2020-03-14T21:16:00Z
draft: false
tags: ["blogger", "Go", "google API", "blogger API", "Golang"]
---
## Introduction
We will be creating a commandLine utility in `Go` to upload a post in Google blogger.

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
   * Note down blog ID (a string after **blogID=**, excluding #allposts) in above URL. We need this ID to put in `Go` script.

## Go development environment creation
I am using following `Go` version.

```bash
$ go version
go version go1.13.8 darwin/amd64
```

### Setup following `Go` development directory structure.

```bash
mkdir -p BloggerCli/go-cli
touch BloggerCli/go-cli/createAndActivateGoWorkSpace.sh
```

Populate `BloggerCli/go-cli/createAndActivateGoWorkSpace.sh` with following contents

```bash
#! /bin/bash
# current directory is workspace
WORKSPACE=$(pwd)
cd "$WORKSPACE"
mkdir -p src bin pkg
export GOPATH=$WORKSPACE
export GOBIN=$WORKSPACE/bin
```

Create directory structure

```bash
cd BloggerCli/go-cli
source ./createAndActivateGoWorkSpace.sh
mkdir src/blogger
```

### Install required libraries.

```bash
go get -v google.golang.org/api/blogger/v3
go get -v golang.org/x/oauth2/google
```

### Copy credential file.
Copy above downloaded credential json file in `BloggerCli/go-cli/src/blogger` folder and rename it to `OAuth2.0_secret.json`.

**Note:** You can rename it any name

### Create `Go` file `BloggerCli/go-cli/src/blogger/bloggerCli.go` file with following contents.
```go
package main

/*
https://pkg.go.dev/google.golang.org/api/blogger/v3?tab=doc
https://godoc.org/google.golang.org/api/option
https://pkg.go.dev/golang.org/x/oauth2/google?tab=doc
https://pkg.go.dev/golang.org/x/oauth2?tab=doc#Config.AuthCodeURL
*/

import (
	"context"
	"flag"
	"fmt"
	"io/ioutil"
	"log"
	"os"
	"strings"

	"golang.org/x/oauth2"
	"golang.org/x/oauth2/google"

	"google.golang.org/api/blogger/v3"
	"google.golang.org/api/option"
)

func checkErrorAndExit(err error) {
	if err != nil {
		log.Fatal(err)
	}
}

func printNoOfPosts(postsService *blogger.PostsService, BlogID string) {
	postListCall := postsService.List(BlogID).MaxResults(500)
	postList, err := postListCall.Do()
	checkErrorAndExit(err)
	fmt.Printf("Total posts in this blog: %d\n", len(postList.Items))
}

func getBloggerClient() *blogger.Service {
	ctx := context.Background()

	contents, err := ioutil.ReadFile("OAuth2.0_secret.json")
	checkErrorAndExit(err)

	conf, err := google.ConfigFromJSON(contents, blogger.BloggerScope)
	checkErrorAndExit(err)

	url := conf.AuthCodeURL("state")
	fmt.Println("Visit the following URL for the auth dialog:")
	fmt.Printf("\n%v\n", url)

	var code string
	fmt.Printf("\nEnter the code obtained from above here: ")
	fmt.Scanln(&code)

	token, err := conf.Exchange(oauth2.NoContext, code)
	checkErrorAndExit(err)

	bloggerService, err := blogger.NewService(ctx, option.WithTokenSource(conf.TokenSource(ctx, token)))
	checkErrorAndExit(err)

	return bloggerService
}

func checkMandatoryArguments(args []*string) {

	allArgsExists := true
	for _, val := range args {
		if len(*val) == 0 {
			allArgsExists = allArgsExists && false
		} else {
			allArgsExists = allArgsExists && true
		}
	}

	if !allArgsExists {
		flag.PrintDefaults()
		os.Exit(2)
	}
}

func createNewPost(title *string, labels *string, fileToUpload *string) *blogger.Post {

	labelsSlice := strings.Split(*labels, ",")
	contents, err := ioutil.ReadFile(*fileToUpload)
	checkErrorAndExit(err)

	newPost := blogger.Post{}

	if len(*labels) == 0 {
		newPost = blogger.Post{
			Content: string(contents),
			Title:   *title,
		}
	} else {
		newPost = blogger.Post{
			Content: string(contents),
			Labels:  labelsSlice,
			Title:   *title,
		}
	}
	return &newPost
}

func insertAPost(title *string, labels *string, fileToUpload *string, BlogID string, postsService *blogger.PostsService) {
	fmt.Println("============= Inserting a new Post ===============")

	newPost := createNewPost(title, labels, fileToUpload)

	postInsertCall := postsService.Insert(BlogID, newPost)
	post, err := postInsertCall.Do()
	checkErrorAndExit(err)
	fmt.Println("A new post added to Blog with following details")
	fmt.Printf("Title: %s\nPostID: %s\n", post.Title, post.Id)
	fmt.Println("=================================================")
	printNoOfPosts(postsService, BlogID)
}

func updateAPost(title *string, labels *string, fileToUpload *string, BlogID string, postsService *blogger.PostsService, postID string) {
	fmt.Println("============= updating a Post ===============")

	// Search post from Title if postID not specified.
	if len(postID) == 0 {
		postsSearchCall := postsService.Search(BlogID, *title)
		postsList, err := postsSearchCall.Do()
		checkErrorAndExit(err)
		if len(postsList.Items) > 1 {
			fmt.Printf("Following multiple posts found for title = %s\n", *title)
			for _, post := range postsList.Items {
				fmt.Println(post.Title)
			}
			log.Fatal("\n\nERROR: Not updating post. Title must identify a single post only. Try specifying PostID")
		} else {
			postID = postsList.Items[0].Id
		}
	}

	newPost := createNewPost(title, labels, fileToUpload)
	postsPatchCall := postsService.Patch(BlogID, postID, newPost)
	post, err := postsPatchCall.Do()
	checkErrorAndExit(err)
	fmt.Println("Following post has been updated in Blog with following details")
	fmt.Printf("Title: %s\nPostID: %s\n", post.Title, post.Id)
	fmt.Println("=================================================")
	printNoOfPosts(postsService, BlogID)
}

func main() {

	fileToUpload := flag.String("f", "", "html file to upload (mandatory)")
	title := flag.String("t", "", "Post title (mandatory)")
	blogid := flag.String("b", "", "Blod ID (mandatory)")
	labels := flag.String("l", "", "comma separated list of labels (optional)")
	modify := flag.Bool("m", false, "Modify a post contents (optional: modifies post in mentioned in -t )")
	postID := flag.String("p", "", "post ID (optional: To be specified when updating a post)")
	flag.Parse()

	if flag.NFlag() == 0 {
		flag.PrintDefaults()
		os.Exit(1)
	}

	mandatoryArgs := []*string{fileToUpload, title, blogid}
	checkMandatoryArguments(mandatoryArgs)

	BlogID := *blogid
	PostID := *postID

	bloggerService := getBloggerClient()
	postsService := bloggerService.Posts

	printNoOfPosts(postsService, BlogID)

	if !*modify {
		insertAPost(title, labels, fileToUpload, BlogID, postsService)
	} else {
		updateAPost(title, labels, fileToUpload, BlogID, postsService, PostID)
	}
}
```

### Create a test file to be uploaded.
```bash
echo 'This is a first post' > sometext.txt
```
**Note**: You can create an HTML file as well.

## Run the script
### Create a New Post
```bash
$ cd BloggerCli/go-cli/src/blogger

$ go run bloggerCli.go -b 232343232322 -f sometext.txt -t "A new Post" -l "label1,label2"
Visit the URL for the auth dialog: https://accounts.google.com/o/oauth2/auth?client_id=99999999-ccc55566666iiioo999ddd.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fblogger&state=state
Enter the code obtained from above here: 9/rf7hM-sds98JnHgsdK90OiYhsss790NHdsaDs
Total posts in this blog: 51
============= Inserting a new Post ===============
A new post added to Blog with following details
Title: A new Post
PostID: 2344555555555999
=================================================
Total posts in this blog: 52
```

### Update a Post
```bash
$ cd BloggerCli/go-cli/src/blogger

$ go run bloggerCli.go -b 232343232322 -f sometext.txt -t "Multiple Vms Vagrantfile" -m
Visit the URL for the auth dialog: https://accounts.google.com/o/oauth2/auth?client_id=99999999-ccc55566666iiioo999ddd.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fblogger&state=state
Enter the code obtained from above here: 9/rf7hM-sds98JnHgsdK90OiYhsss790NHdsaDs
Total posts in this blog: 51
============= updating a Post ===============
Following post has been updated in Blog with following details
Title: Multiple Vms Vagrantfile
PostID: 334556669992323
=================================================
Total posts in this blog: 51
```
**Note:** You need to open the `URL` mentioned in the above output in a browser and authorize the request. This will return a `code` that need to enter above.


### References used
* https://pkg.go.dev/google.golang.org/api/blogger/v3?tab=doc
* https://godoc.org/google.golang.org/api/option
* https://pkg.go.dev/golang.org/x/oauth2/google?tab=doc
* https://pkg.go.dev/golang.org/x/oauth2?tab=doc#Config.AuthCodeURL
