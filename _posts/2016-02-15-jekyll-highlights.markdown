---
title:  "Jekyll Highlights"
date:   2016-02-15 10:38:04 -0200
categories: jekyll update
---

## Cool stuff about Jekyll

Hello World!

[Jekyll](http://jekyllrb.com/) is a static site generator. What does that mean? It means that it doesn't have a backend coupled to it and all the tasks of generating pages based on templates, the URL structure and so on is made before the deploy in a build step.

It is ideal then for generate websites where content is the main *feature*. Everything you need is just manage a folder with files, there's no need of dealing with programming languages - unless you want create plugins for it - or databases. I don't know if this is a common expression in English, but in my language we commonly say 'Do not use a cannon to kill an ant' - basically: do not do overengineering.

Jekyll's documentation is great. It is easy to start with and it goes pretty deep on more advanced things. This post is more about what I think it is nice on this static site generator and this is also a cheatsheet - I love cheatsheets.

### Easy start

    $ gem install jekyll
    $ jekyll new my-site && cd my-site
    $ jekyll serve

### Easy configuration

Jekyll provides a `_config.yml` file, where you can change default behaviors or add custom configuration. There are plenty of configuration ([read docs][jekyll-docs]). A example that I like is `permalink`, that you can change the way your *URLs* are defined.

[jekyll-docs]: http://jekyllrb.com/docs/configuration/ "Jekyll configuration docs"

### Built-in categories and tags

On Jekyll, each post you create you can add a configuration part called `YAML Front Matter` or just `Fron Matter`, that's basically a YAML piece that you put on the very top of your file. With `Front Matter` you can override configuration written in the `_config.yaml` file and also add your custom ones for that specific page. In the `Front Matter` you can add categories and tags to that post, so it's easier to organise and create searches in your site or blog.

### Code highlighting

A very nice thing Jekyll brings out of the box as well is code highlighting. As this is a tech blog and I basically will write about software development - languages, libs/frameworks and tools -, it is very good not being worried about this technicality.

{% highlight ruby linenos %}
class RubySayer
  def say(name)
    puts "RUBY, #{name}, RUBY!"
  end
end

RubySayer.new.say('ARTHUR')
#=> 'RUBY, ARTHUR, RUBY!'
{% endhighlight %}

### Templates

The templating language available on Jekyll is called `Liquid`. In `Liquid` you don't have access to a programming language so you do whatever want. You have directives that you can use to include template partials, display content, evaluate conditional expressions, iterate over lists etc. It's pretty restrictive. That might sound bad but in fact it is a good thing on my point of view. First is that the complexity to deal with these directives, once you learn how to they work, is much smaller than maintaining a project written with a fully capable language. Other advantage is that is more secure just running a few set of directives than run a whole language on your servers. Because of being pretty secure, Github gives you the possibility of hosting websites/blogs in their own infrastructure using Jekyll. Those below are include directives, very useful for keep html partials separate with their own responsabilities:

<img src="/images/2016-02-15-jekyll-highlights/jekyll-template-structure.png" alt="">

### Community

Lots of people use Jekyll and that makes the tool even better. Lots of plugin contributions were and continue being made. Jekyll's website has a page dedicated to Plugins with a long list full of them. Check it [here][jekyll-plugins]

[jekyll-plugins]: http://jekyllrb.com/docs/plugins/ "Jekyll Plugins"

So this blog is powered by Jekyll and write about it was a good way of breaking the ice and start writing a technical blog.
