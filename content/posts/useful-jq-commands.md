+++
title = "jq examples"
date = "2019-08-11"
aliases = ["Post","post","blog-page"]
tags = ["jq","cheatsheet", "linux admin"]
disqusShortname = "http-applegreengrape-com"
+++

jq is a command-line JSON processor to parse json format data. You can find the detailed documentation [here](https://stedolan.github.io/jq). And you can try to play it online at [jqplay.org](https://jqplay.org/). I am listing out few jq command line examples that I found quite useful for day-to-day work 😃.

Let's try jq with [AWS resources api](https://pricing.us-east-1.amazonaws.com). 

```bash
// to parse and extract json data 
$ curl -s  \
https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/index.json \
| jq .offers.AmazonEC2
{
  "offerCode": "AmazonEC2",
  "versionIndexUrl": "/offers/v1.0/aws/AmazonEC2/index.json",
  "currentVersionUrl": "/offers/v1.0/aws/AmazonEC2/current/index.json",
  "currentRegionIndexUrl": "/offers/v1.0/aws/AmazonEC2/current/region_index.json"
}
```
To capture the field of "currentVersionUrl"
```bash
curl -s  \
https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/index.json \
| jq .offers.AmazonEC2 \
| jq .currentVersionUrl
"/offers/v1.0/aws/AmazonEC2/current/index.json"
```
`--raw-output / -r` is the flag used to capture the json string without quotes. It is actually quite useful if you want to use jq in a bash script
 ```bash
curl -s  \
https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/index.json \
| jq .offers.AmazonEC2 \
| jq -r .currentVersionUrl 
/offers/v1.0/aws/AmazonEC2/current/index.json

// using jq in a script
for i in $(curl -s  \
https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/index.json \
| jq -r .offers.AmazonEC2.versionIndexUrl ); \
do echo https://pricing.us-east-1.amazonaws.com$i ; \
done
https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonEC2/index.json


for i in $(curl -s  \
https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/index.json \
| jq .offers.AmazonEC2.versionIndexUrl ); \
do echo https://pricing.us-east-1.amazonaws.com$i ; \
done
https://pricing.us-east-1.amazonaws.com"/offers/v1.0/aws/AmazonEC2/index.json"  
// see sometimes it is better to parse and capture the data without quotes
```

To parse the json data with `arrary[]`.
```bash
curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq . 
// by running jq, you can get a formatted json output which has already been parsed
{
  "success": {
    "total": 1
  },
  "contents": {
    "quotes": [
      {
        "quote": "Do not wait to strike till the iron is hot; but make it hot by striking....",
        "length": "75",
        "author": "William B. Sprague",
        "tags": [
          "action",
          "inspire",
          "tod"
        ],
        "category": "inspire",
        "date": "2019-08-13",
        "permalink": "https://theysaidso.com/quote/william-b-sprague-do-not-wait-to-strike-till-the-iron-is-hot-but-make-it-hot-by",
        "title": "Inspiring Quote of the day",
        "background": "https://theysaidso.com/img/bgs/man_on_the_mountain.jpg",
        "id": "UNe_Znlalg5mFEBaAWzoCweF"
      }
    ],
    "copyright": "2017-19 theysaidso.com"
  }
}

// To capture the json data within the array[]
curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq '[.quote, .tags[0], .tags[1] ]'
[
  "Do not wait to strike till the iron is hot; but make it hot by striking....",
  "action",
  "inspire"
]
```

To filter json data:
```bash
curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq 'select(.tags[] == null)'
// no data returned since everything has been filtered out

curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq 'select(.tags[] = "td") | [.quote, .tags[1]]'
[
  "Do not wait to strike till the iron is hot; but make it hot by striking....",
  "inspire"
]

curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq 'select(.tags[] == "tod") | [.quote, .tags[1]]'
[
  "Do not wait to strike till the iron is hot; but make it hot by striking....",
  "inspire"
]
// "=" can work as contains instead of equals
```

To reformat json data:
```bash
curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq '{tag_1: .tags[0], tag_2: .tags[1], tag_3: .tags[2]}' 
{
  "tag_1": "action",
  "tag_2": "inspire",
  "tag_3": "tod"
}

curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq '{tag_1: .tags[0], tag_2: .tags[1], tag_3: .tags[2]}' \
| jq .tag_1
"action"
```

Also there is some interesing array[] indexing jq is following. It works similar to iterating over slices in golang. Something like this:
```bash
curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq 'select(.tags[] != "tod")| .quote'
"Do not wait to strike till the iron is hot; but make it hot by striking...."
"Do not wait to strike till the iron is hot; but make it hot by striking...."

curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq 'select(.tags[] != "td")| .quote'
"Do not wait to strike till the iron is hot; but make it hot by striking...."
"Do not wait to strike till the iron is hot; but make it hot by striking...."
"Do not wait to strike till the iron is hot; but make it hot by striking...."
```

To escape null, you can use `//` as the alternative operator:
```bash
curl -s \
--request GET \
--url 'http://quotes.rest/qod.json' \
| jq .contents.quotes[] \
| jq '{tag_1: .tags[0], tag_2: .tags[1], tag_3: .tags[2]}' \
| jq '[.tag_1, .tag_2, .tag_4//"no that tag yet"]'
[
  "action",
  "inspire",
  "no that tag yet"
]
```

Oh well, there are much more jq commands you can use, like a MYSQL style of `join()`. Documentaion is quite good and you can find it [here](https://stedolan.github.io/jq/manual/#ConditionalsandComparisons). Leave a comment if you have more jq questions 😃
