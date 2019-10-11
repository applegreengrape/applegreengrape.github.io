+++
title = "amazon s3"
date = "2019-08-12"
aliases = ["Post","post","blog-page"]
tags = ["bash", "AWS", "amazon-s3", "cloud admin"]
disqusShortname = "http-applegreengrape-com"
+++

aws s3 cli is great! You can easily move your local files to your aws s3 buckets. However, sometimes it is not that easy to do simple tasks - like copy files where there is a whitespace in the file name, delete all versions of all files in a versioned s3 bucket and the difference between `aws s3 sync`  and `aws s3 cp --recursive `. 

### escape the whitespace in your file name

When you have a long list of files that you need to upload to s3 bucket, it will be easy for you to loop it through if you have nice filenames that there are no whitespace. Something like this:
```bash
.
├── test1.txt
├── test2.txt
├── test3.txt
└── test4.txt

$ for i in $(ls); do echo $1 && aws s3 cp ./$i s3://your_s3bucket_name/$i; done
```
However, sometimes it is slightly more complicated when it comes to filenames with whitespace. There is also one command handy - `basename`. Something like this:
```bash
$ tree .
.
├── s3.sh
├── test\ 1.txt
├── test\ 2.txt
└── test3.txt

0 directories, 4 files
$ basename "$(ls)"
s3.sh
test 1.txt
test 2.txt
test3.txt
```
So if we put it into a bash script loop, it will be something like this:
```bash
#!/bin/sh

find .  -type file -name "*" -print0 | while IFS= read -r -d '' file; do
#    echo $file
    KEY=$(basename "$file")
    echo $KEY
    echo aws s3api put-object --bucket your_s3_bucket --key '"'$KEY'"' --body $file
    aws s3api put-object --bucket your_s3_bucket --key '"'$KEY'"' --body $file
done
```
### delete all versions of all files in a versioned s3 bucket
When it comes to delete a versioned s3 bucket, you will need to delete all versions of s3 bucket objects before you can delete the s3 bucket. So let's wrap a simple script to do it. You will use `aws s3` command and `jq`. If you don't know anything about jq. You can check my last post on [jq](/posts/useful-jq-commands).
```bash
#!/bin/bash

bucket=$1

set -e

echo "Removing all versions from $bucket"

versions=`aws s3api list-object-versions --bucket $bucket |jq '.Versions'`
markers=`aws s3api list-object-versions --bucket $bucket |jq '.DeleteMarkers'`
let count=`echo $versions |jq 'length'`-1

if [ $count -gt -1 ]; then
        echo "removing files"
        for i in $(seq 0 $count); do
                key=`echo $versions | jq .[$i].Key |sed -e 's/\"//g'`
                versionId=`echo $versions | jq .[$i].VersionId |sed -e 's/\"//g'`
                cmd="aws s3api delete-object --bucket $bucket --key $key --version-id $versionId"
                echo $cmd
                $cmd
        done
fi

let count=`echo $markers |jq 'length'`-1

if [ $count -gt -1 ]; then
        echo "removing delete markers"

        for i in $(seq 0 $count); do
                key=`echo $markers | jq .[$i].Key |sed -e 's/\"//g'`
                versionId=`echo $markers | jq .[$i].VersionId |sed -e 's/\"//g'`
                cmd="aws s3api delete-object --bucket $bucket --key $key --version-id $versionId"
                echo $cmd
                $cmd
        done
fi
```
