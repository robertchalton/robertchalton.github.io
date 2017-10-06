---
layout: post
published: true
title: Migrating my blog from Blogger to GitHub Pages with Jekyll
subtitle: My migration from blogger.com to pages.github.com
---
## Jekyll

I chose https://jekyllrb.com/ as the framework mostly becase it seems quite popular and looke easy. The blog posts are in markdown which makes the editing very simple. I ended up using http://deanattali.com/beautiful-jekyll/ which has a nice clear readme on how to get it up and running https://github.com/daattali/beautiful-jekyll#readme which amounts to fork the github project, update the url and go.

After the fork, clone the repo local to add posts locally before posting to github.



## URL

Once the project is forked go into the repo settings down to the GitHub Pages section. Enter the custom domain, in my case krisrice.io

![Screen Shot 2017-10-06 at 9.03.31 AM.png]({{site.baseurl}}/img/Screen Shot 2017-10-06 at 9.03.31 AM.png)

# Gothcha
 The gotcha with the custom domains is there's no option/feature yet to upload a custom SSL cert to accompany the domain.  Github will still host the pages but with their SSL Cert which is browsers will flag as a security issue. So best to no reference your new site with HTTPS. There's a looong list of people asking for this to be built https://gist.github.com/coolaj86/e07d42f5961c68fc1fc8

![Screen Shot 2017-10-06 at 9.03.50 AM.png]({{site.baseurl}}/img/Screen Shot 2017-10-06 at 9.03.50 AM.png)

## Up and running

That's all it takes to get up and running.

## Migrating Blogger

The migration mostly just works. I hit a bug in the migrate code because I had some draft blogs that had / in the title of the blog.  The problem is that blogger escapes those for published posts but not for drafts. The bug is that in Jekyll the postname is the file name so having a / in that didn't work out. I just edited those titles to remove the slashes.  The Jekyll team is adding a fix for it here: https://github.com/jekyll/jekyll-import/issues/321#issuecomment-331204992 

# Step 1 - Export Blogger

Export the Blogger posts. Blogger doesn't make it obvious where/how to export as it's in Settings -> Others


![Screen Shot 2017-10-06 at 9.00.53 AM.png]({{site.baseurl}}/img/Screen Shot 2017-10-06 at 9.00.53 AM.png)


# Step 2 - Migrate

This is just run the migrate code on the Jekyll page. 

![Screen Shot 2017-10-06 at 9.22.50 AM.png]({{site.baseurl}}/img/Screen Shot 2017-10-06 at 9.22.50 AM.png)


## Test and Publishing

Here's gotcha #2 always test the post local before just pushing the markdown file out to github.  I had an edit that failed on github to publish and all you get from github is the equivalent of an ORA-660, something failed. So always build and verify. https://jekyllrb.com/docs/usage/




![Screen Shot 2017-10-06 at 9.09.07 AM.png]({{site.baseurl}}/img/Screen Shot 2017-10-06 at 9.09.07 AM.png)


## New/Editing Posts

There's a number of Markdown desktop tools like [MacDown](https://macdown.uranusjr.com/) which I normally use.  So far I've instead been using [prose.io](http://prose.io/) to edit new posts like this one. It's quite easy to use and posts/commits straight into github.


![Screen Shot 2017-10-06 at 10.00.48 AM.png]({{site.baseurl}}/img/Screen Shot 2017-10-06 at 10.00.48 AM.png)

