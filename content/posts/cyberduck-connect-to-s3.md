+++
title = "Cyberduck <- to -> S3"
date = "2022-12-07T14:39:44+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["aws"]
keywords = ["aws", "cyberduck", "s3"]
description = "Using Cyberduck to simplify moving data in and out of S3"
showFullContent = false
readingTime = false
hideComments = false
color = "orange" #color from the theme settings
+++

Here's an easy way to move data in and out of Amazon S3

[Cyberduck](https://cyberduck.io/) is an FTP app for MacOS and Windows that also connects to S3 object storage. It provides a nice graphicall way to handle data in S3




Click on `Open Connection` to create a new connection

![image alt text](/posts/cyberduck-connect-to-s3/cyberduck-s3-1.png)


Select S3, then enter your Access Key and Secret Access Key for your IAM user

![image alt text](/posts/cyberduck-connect-to-s3/cyberduck-s3-2.png)

Once done you can see a list of all the S3 buckets in that account

*Note: you can only open the ones the IAM user has access to.*

![image alt text](/posts/cyberduck-connect-to-s3/cyberduck-s3-3.png)


## Bonus! S3 storage classes

To see the S3 storage class, right click on the red underlined section and ensure `Storage Class` is ticked

![image alt text](/posts/cyberduck-connect-to-s3/cyberduck-s3-4.png)

Then there will be a column showing the storage class of each object

![image alt text](/posts/cyberduck-connect-to-s3/cyberduck-s3-5.png)

To move an object to a different storage class, right click on an object and select `info`

![image alt text](/posts/cyberduck-connect-to-s3/cyberduck-s3-6.png)


Then in the info window select S3

![image alt text](/posts/cyberduck-connect-to-s3/cyberduck-s3-7.png)

There is also a section to create basic lifecycle rules to transition objects into Glacier or delete them after a set number of days