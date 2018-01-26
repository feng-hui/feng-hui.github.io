---
layout: post
title: "如何搭建jekyll博客"
date: 2017-12-18
description: 如何搭建jekyll博客
tags: [jekyll,ruby]
---
> 主要讲述如何在windows系统上安装并运行jekyll，同时部署到github pages上，配合购买好的域名，来访问我们搭建的博客。

windows 搭建jekyll博客

参考链接：[http://www.jianshu.com/p/27de87d4447e](http://www.jianshu.com/p/27de87d4447e)

## 1、下载rubyinstaller并安装；

## 2、更改默认的source源；

首先查看当前已经添加的源

{% highlight ruby %}gem source -l;{% endhighlight %}


然后移除官方源和淘宝源

{% highlight ruby %}gem sources -r https://rubygems.org/ {% endhighlight %}


{% highlight ruby %}gem sources -r https://ruby.taobao.org/ {% endhighlight %}


添加ruby china源

{% highlight ruby %}gem source -a http://gems.ruby-china.org {% endhighlight %}

## 3、安装jekyll

安装依赖包：

{% highlight ruby %}gem install bundler {% endhighlight %}


{% highlight ruby %}gem install jekyll{% endhighlight %}


## 4、运行博客

{% highlight ruby %}jekyll serve{% endhighlight %}