---
title: 'How I Built This Website'
date: 2019-02-24 00:00:00
excerpt: This post details how I setup the website you're looking at! It includes topics like domain registration, DNS, AWS S3 static website hosting, and setting up a CICD pipeline.
categories: [cloud]
tags: [aws, s3, jekyll, cloudfront, cicd]
featured_image: '/images/demo/demo-square.jpg'
---

Up until about a week ago, I had a pretty sweet website that I hosted using [GitHub Pages](https://pages.github.com/). When my student account expired, I decided not to buy the GitHub Pro membership that would let me continue to host my website. Instead, I decided to put my knowledge of AWS to good use and host it there instead. I've set up everything from my own SSL/TLS certificate to a CI/CD pipeline. I got a kick out of building it. I'm looking forward to continuing to experiment and build the website out. Here's how I did it:

## Infrastructure
The first thing you're going to need for your brand spankin' new website is a domain name. I went to [Google Domains](https://domains.google.com) to grab mine. They have a good selection of endings, fair prices, and it's a super easy process. There are lots of other places you can purchase domain names - they will all work with the website infrastucture we're going to set up.

Once you've got your domain name (e.g., *galenballew.fyi*), it's time to get our hands dirty in the cloud. I built my website, CDN, and CICD pipeline all in AWS, but Azure and GCP have the same functionality.

#### S3 Buckets
1. Create a bucket for server access logs.
      * I like to set up a Lifecycle Management Policy to delete logs after 90 days.
2. Create a bucket named _galenballew.fyi_
      * Enable this bucket for static website hosting.
      * Besure to uncheck the box that says "Prevent public policies".
      * Apply a [public bucket policy or ACL](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteAccessPermissionsReqd.html).
      * Configure logging to the logging bucket
3. Create a bucket named _www.galenballew.fyi_
      * Under **Static Website Hosting**, configure the bucket to redirect requests to _galenballew.fyi_ via https.
      * Configure logging to the logging bucket. 

#### Content Distribution Network
A CDN is not necessary for your website, but it does bring some value. Namely, CloudFront will cache objects out on it's edge locations. This will speed your website up for anyone accessing it from another part of the country or world. Further, your CloudFront distribution will allow end users to access it over HTTPS - security first folks. It's worth noting that CloudFront distributions come with _a lot_ of configurablity. I'm only going to cover the settings that were important for my simple website. 

1. Go to AWS Certificate Manger and request a new public certificate. 
    * Provide the root domain as the domain name (i.e., *galenballew.fyi*).
    * Give *www.galenballew.fyi* as an additional name.
    * I opted for DNS validation - it will take care of itself in Route53.
2. Go to CloudFront and create a new distribution (web delivery).
    * Set logs to your logging bucket.
    * Set HTTP to be redirected to HTTPS.
    * Set the Price Class to US, Canada, and Europe (use your best judgement here, I wanted to save a few pennies.)
    * Provide *galenballew.fyi* and *www.galenballew.fyi* as CNAMEs.
    * Set SSL to the certificate you created in step 1. 
    * When setting the origin, be sure to use the actual website endpoint (e.g.,*galenballew.fyi.s3-website-us-east-1.amazonaws.com*) **_not_** the bucket itself. If you're setting this up via the web console, there will be a drop down menu that allows you to select the S3 bucket as the origin. **_This is not the same as setting the origin to the website endpoint!_**
    * Deploy that sucker. It takes a few minutes to deploy. You can use the time to set up DNS. 

#### Domain Name System
1. Go to AWS Route 53 and create a new hosted zone.
    * Name the zone after the root domain.
    * Inside the hosted zone, create an alias record for the root.
      * Select and **A - IPv4** record type and **Alias: Yes**. Provide the CloudFront distribution domain name as the **Alias Target**.
    * Create another record set for _www.galenballew.fyi_ the exact same way. 
2. Copy the 4 name server domain names in the hosted zone. These are the 4 values listed in the **NS** record set for the root domain of your website.
    * Go to Google Domains, or whatever provider you chose, and find the DNS settings for your domain. There should be an option to use custom name servers.
    * Pop the 4 domain names you wrote down into as the name servers for your domain. 
  
---

## Static Site & Content
Alright, we have a website, but there is nothing there! Not even an _index.html_ root document. We need to put some content up for people to see. I looked at a few different static site generators, including [Hugo](https://themes.gohugo.io/) and [Jekyll](https://jekyllrb.com/). Ultimately, I decided on Jekyll after trying out some of the Hugo themes. I even went so far as to pay for the theme I'm using because I liked it so much - [Jekyll Journal Theme](https://jekyllthemes.io/theme/journal-personal-jekyll-theme). 

I will leave it as an exercise to the reader to get Jekyll installed and configured. The docs are a [good place to start](https://jekyllrb.com/docs/). Once you have a directory with some static site content in it, I would **_highly recommend_** storing your website in version control. If you are using Perforce or Mercurial all the power to you, but for everyone else in the world, go with Git. If you have a [GitHub](https://github.com/) account, set up a repository and push your content to it. This is great for managing your content, but it will also become critical when setting up a CI/CD pipeline. 

One other thing I like to do is add the _\_site/_ directory to my _.gitignore_ file. It's not necessary to save in version control since it will be repopulated with each `bundle exec jekyll build`. 

---

## CI/CD 
This last bit is, in my opinion, the best part. AWS makes it super easy to integrate with GitHub for continuous integration and continuous deployment/delivery. Everything that happens in the build process is contained in a YAML configuration file inside of your website directory. It makes publishing and fix-forward updates painless. 


1. Go to AWS CodePipeline and create a new pipeline.
    * I used default values whenever possible.
    * Integrate the **Source** to be your GitHub account (or perhaps CodeCommit if you're fancy!)
    * I set mine to use GitHub webhooks. This mean's the pipeline will execute after any commits are pushed to the master branch. 
2. Select new CodeBuild project.
    * Keep things simple here. I went with the smallest Linux environment available.
    * Select **Use a buildspec file**. I went with the default name and location. 
    * Enable CloudWatch logs so you can monitor the logs for your build in real-time. This can be super useful if you need to debug your buildspec.yml 
    * Enable S3 logs as well if you're into it.
3. Create the pipeline!

Now that the pipeline is up, it will execute every time the GitHub webhooks fire off a `POST`. But what exactly will execute? Whatever is in your _buildspec.yml_. You'll want to place this file at the root of your website directory. Here is my *buildspec.yml*:

```yaml
version: 0.2
phases:
 install:
   commands:
     - echo "install step"
     - gem update --system --quiet
     - gem install jekyll jekyll-paginate jekyll-sitemap jekyll-gist
     - echo "jekyll installed"
     - gem uninstall bundler
     - gem install bundler
     - bundle install
     - echo "bundle installed"
 build:
   commands:
     - echo "building step"
     - bundle exec jekyll build --verbose
     - echo "building complete"
 post_build:
   commands:
     - echo "post_build step"
     - aws s3 sync --delete _site/ s3://galenballew.fyi/
     - echo "files transferred successfully"
     - aws cloudfront create-invalidation --distribution-id E1BBD9XD38EPQC --paths "/*"
     - echo "CDN cache invalidated"
```
Just to highlight the major events:
1. Jekyll and bundler are installed
2. The website is built
3. The _\_site/_ directory is synced to the root S3 bucket. `--delete` ensures that anything in the bucket that is _not_ in _\_site/_ will be deleted from the bucket.
4. Everything cached in CloudFront is invalidated. 

This is a simple and heavy handed buildspec, but it works great! Tinker with it as needed. 

---
## Conclusion

This was a really fun project to set up. My only regret is that I did not do it via CloudFormation so that I could share it with all of you lovely people, or easily dupliate my infrastructure in new regions. It's probably going to cost me a few more dollars to host my website in AWS than to pay $7/month for GitHub Pages, but I think it's worth it. At least my website won't go down if I make it to the front page of Hacker News :smile:

---

## Resources

* https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html
* https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteAccessPermissionsReqd.html
* https://jekyllrb.com/docs/