---
layout: post
title:  "The Structure of This Blog"
date:   2017-03-15 21:44:52 +0800
categories: Productivity
tags: Git
---
This is the first post of this blog, and it's about how I manage this blog.

People have secrets. What if one wants to keep some of his posts out of certain people's sights? The most straightforward way may be maintain a series of blogs, and keep different sets of blogs in them. However there may be some posts appear nearly in all the blogs, and it's a pity to copy and paste them all the time.

My solution is making use of git's branch facilities. Basically I sort my posts into different categories and put them into separate branches. To publish a blog of certain "degree of secrets", just merge the branches you want people to see into a specified branch, like `gh-pages` for GitHub pages.

This method can also be used to create different themes for different blogs if one keeps theme files in certain branches as well.

The big picture:

![Blog Structure]({{ site.url }}/assets/2017-03-15-the-structure-of-this-blog/blog-structure.png)
