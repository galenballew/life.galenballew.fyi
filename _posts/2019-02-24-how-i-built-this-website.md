---
title: 'How I Built This Website'
date: 2019-02-24 00:00:00
featured_image: '/images/demo/demo-square.jpg'
excerpt: This post details how I setup the website you're looking at! It includes topics like domain registration, DNS, AWS S3 static website hosting, and setting up a CICD pipeline.
---



Infrastructure
* google domains
    * buy a domain
* set up cloudfront
    * make a certificate
    * go back and change alias records to point at the cloudfront distribution
    * make sure origin is set to s3 website url and not just the bucket
* make buckets
    * root, www, logging
        * configure a lifecycle management policy for logging
    * need to make sure the prevent public policies box is not checked
    * apply public policy
* configure root for website hosting
* configure www for redirect
* go to route53
    * create hosted zone for the root
        * create an alias record for the root
        * create an alias for www that points to the other alias record
* copy the NS records from your hosted zone
    * go to google domains and use these as custom IP
---
Static Site + Content
* jekyll
    * i looked at hugo, but jekyll had better themes
    * i chose to buy one, but there are lots of free options
* install ruby + jekyl
    * link to how to
    * install bundler
* jekyll build
* **optional** set up codecommit or github
    * .gitignore _site
---
CICD
* create codebuild project
    * enable cloudwatch logs (and s3 if you want)
    * this lets you see the build logs in realtime
        * useful to debug buildspec
* create pipeline
* make buildspec.yml
    * include file