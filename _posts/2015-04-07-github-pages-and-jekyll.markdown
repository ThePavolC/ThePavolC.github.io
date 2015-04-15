---
layout: post
title:  GitHub Pages and Jekyll
categories: jekyll github-pages how-to
description: Quick guide on how to make GitHub page and start using Jekyll.
published: True
---

By some accident, I run into [GitHub Pages website][github-pages] and I decided to give it a try. It seems like really quick and easy way to get yourself a website with full control over it.

To install Jekyll on Ubuntu 14.04, my home computer, I first installed ruby and ruby development packages.

{% highlight bash %}
sudo apt-get install ruby ruby-dev make gcc nodejs
{% endhighlight %}

Then I installed Jekyll using `gem` command, which is shipped with Ruby.

{% highlight bash %}
sudo gem install jekyll
{% endhighlight %}

After that I was all set and ready to go. You can verify that Jekyll was successfully installed by checking it's installed version.

{% highlight bash %}
jekyll --version
{% endhighlight %}

If you run into problems, there are thousands of posts to help you out. [I pretty much just followed this post.][jekyll-installation]

# Creating a website

To get your page on your GitHub account, just create repository with name `username.github.io` with your user name. Then all you need to do is:

* Download the repository.

~~~~
> clone https://github.com/<username>/<username>.github.io.git
~~~~

* Create the jekyll website and maybe have a look at it on your computer.

~~~~
> jekyll new <username>.github.io
> cd <username>.github.io
> jekyll serve
~~~~

* Push it to server.

~~~~
> git add --all
> git commit -m "Initial commit"
> git push -u origin master
~~~~

I changed the look of website, based on my old static website I had on my computer.

![Old website](/assets/website.png)


So far, I really like Jekyll. It's fairly simple, has loads of add-ons and features. And it just works. You are pretty much able to have whole website running within a minute and everyone who is little bit familiar with HTML & CSS can create and update content.

[github-pages]: https://pages.github.com/
[jekyll-installation]: http://michaelchelen.net/81fa/install-jekyll-2-ubuntu-14-04/
