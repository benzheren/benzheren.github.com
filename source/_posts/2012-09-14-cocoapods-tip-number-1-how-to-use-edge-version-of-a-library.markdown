---
layout: post
title: "CocoaPods Tip #1: How To Use Edge Version of A Library?"
date: 2012-09-14 10:55
comments: true
categories: [iOS, Software]
tags: [iOS CocoaPods Github]
---

I am using [CocoaPods](http://cocoapods.org/) to manage my iOS project. Because a lot of iOS libraries are open source, sometimes you have to use the edge version of the library to get the latest pathes to make things work. If you read through articles on the [project wiki](https://github.com/CocoaPods/CocoaPods/wiki/Dependency-declaration-options), and it seems to be pretty easy and just like how developers do in Gemfile: 
    
    pod 'LibName', :git => 'https://url/of/the/github/repo'
 
If you try to enter ``pod install``, it probably won't work the way you expect: you will still use the old library. So the easiest way to solve this problem is to **delete your Pods director** and do ``pod install``. And this is still the way to make it work for CocoaPods 0.14.0. 
