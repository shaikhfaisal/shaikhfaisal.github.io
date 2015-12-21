---
title: Installing dredd
layout: default
excerpt: <p>dredd is a tool for validating your api against your api documentation. While installing it...there was trouble.</p>
 
---
### {{page.title}}

<p>Dredd is a useful sounding tool that one of my colleagues mentioned to me at work. I thought I would play around with a little to better understand where it lies in the software development toolkit.</p>

The first problem I faced was in installing dredd on Ubuntu 14.04. It turned out to be the classic node vs nodejs naming conflict. 

The documentation suggested I attempt to install dredd via the command below 

{% highlight bash %}
sudo npm install -g dredd
{% endhighlight %}

That resulted in a lot of verbose and silly level output. Eventually npm spat out the following:

{% highlight bash %}
npm ERR!
npm ERR! Additional logging details can be found in:
npm ERR!     /vagrant/code/npm-debug.log
npm ERR! not ok code 0
{% endhighlight %}

I also noticed a lot of lines similiar to the following:

{% highlight bash %}
npm Error: ENOENT, lstat '/usr/local/lib/node_modules/dredd/node_modules/html/img/copyashtml'
{% endhighlight %}


A few google searches later I came across this issue on github titled <a href='https://github.com/apiaryio/dredd/issues/175'> 'Installation troubles'</a>. A comment about node reminded me to try 

{% highlight bash %}
sudo ln -s /usr/bin/nodejs /usr/local/bin/node
{% endhighlight %}

The installation now completed succesfully:

{% highlight bash %}
$ dredd --version
dredd v1.0.2
{% endhighlight %}

d:o)
