+++
title = "Web Scraper"
date = "2019-07-09"
aliases = ["Post","post","blog-page"]
tags = ["Golang", "Web Scraper", "text mining"]
disqusShortname = "http-applegreengrape-com"
+++
So you are interested in news and events and you want to track the latest news and headlines. And I guess you are familiar with the concept of RSS [Rich Site Summary](https://en.wikipedia.org/wiki/RSS). Okay,😃Let's start to build a simple Golang application to fetch the latest news and headlines with in a second. 

```go
package main

import (
	"fmt"
	"github.com/mmcdole/gofeed"
)

func feed(){

	fp := gofeed.NewParser()
	
	feed, err := fp.ParseURL("http://feeds.reuters.com/reuters/UKTopNews")
	
	if err != nil {
		panic(err)
	}

	for _, item := range feed.Items {

		fmt.Println(item.Title, item.PublishedParsed)

	}

}

func main(){
     feed()
}
```