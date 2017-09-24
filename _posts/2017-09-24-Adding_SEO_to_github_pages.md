---
title: Adding SEO to github pages
date:   2017-09-24 
layout: post
comments: true
---
After completing my first post on this blog I thought it would be a good idea to get some basic SEO (search engine optimisation) in there.  It turns out this is dead easy, Github and their pages machinery with jekyll have it all set up and there's some very simple instructions you can follow here [Installing Jekyll SEO tag](https://github.com/jekyll/jekyll-seo-tag/blob/master/docs/installation.md).

There's really just one issue, the instructions are wrong.  My first attempt was met with two separate problems.  When attempting to serve my site (locally) I got these errors
```
Deprecation: The 'plugins' configuration option has been renamed to 'plugins_dir'. Please update your config file accordingly.
```
and
```
Liquid Exception: Liquid syntax error (line 8): Unknown tag 'seo' in /_layouts/post.html
```

Being somewhat conversant in tech, it was clear to me that the instructions were out of date.  All I had to do was go to my `_config.yml` file and update `plugins` to be `plugins_dir`.  It didn't make a lot of sense since the property was an array of plugins, not a directory, but the error message is clear.  Sure enough, making this change got me from two errors to just one:

```
Liquid Exception: Liquid syntax error (line 8): Unknown tag 'seo' in /_layouts/post.html
```

Well, that's an improvement but it's evident that changing the property name to `plugins_dir` simply made it a valid, but *wrong* property in `_config.yml`.

Long story short it turns out there have been a couple of changes to property names in `_config.yml` and I was being misled by an older one.  Some time ago, there used to be a `plugins` property that pointed at a directory containing plugins to be loaded.  It was renamed to `plugins_dir` and the old property deprecated.  Then, more recently another property that used to be called `gems` was renamed to `plugins` (see where this is going yet?).  All the instructions think this change has happened and that you should load the `jekyll-seo-tag` gem using the `plugins` property.  However the github pages setup I have locally (which was only set up a couple of weeks ago) does not have this change in place.  The fix, then, is simple: in your `_config.yml` you need to put
```
gems:
  -  jekyll-seo-tag
```

And that is it - now my pages have simple SEO information in the headers.  It should have been a five minute job, it took a little longer, but I guess you can't complain really.  It's all free after all...

Afterword
=========
Looking at the docs page for the jekyll-seo-tag gem, it's clear that the change from `gems` to `plugins` was only made a month or so ago but it seems to be ahead of the curve since my local jekyll setup doesn't seem `gems` as being deprecated yet.  Maybe there's another problem where the local setup instructions don't get you the same version of jekyll as the one github pages are using internally.

The instructions on github tell you to keep up to date with jekyll using
```
bundle update
```
I have run this but still `gems` seems the right property name for now, not `plugins`.