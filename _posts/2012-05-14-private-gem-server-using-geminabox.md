---
layout: post
title: Private Gem Server Using Geminabox
---

The Problem
-----------

At [HealPay](http://www.healpay.com), we were starting to develop a lot of internal gems to handle the business logic of our applications separate from our web applications. We were also creating some re-usable code that we didn't necessarily want to open source (at least yet). We needed something that could sustain our use of rubygems without us having to create a private rubygems server for gem distribution.

The Solution
------------

Enter [Gem in a Box](https://github.com/cwninja/geminabox) - Really simple rubygem hosting. It lets you host your own gems, and push new gems to it just like with [rubygems.org](http://rubygems.org).

[Gem in a Box](https://github.com/cwninja/geminabox) includes a simple web app and command line client. The clean web interface lets you view your published gems and the command line comes bolted on when you install the gem. Pushing new changes for a gem are as easy as running this command:

{% highlight bash %}
gem inabox pkg/mygem-0.0.1.gem --host myprivategemserver.com
{% endhighlight %}

To automate the task of pushing to my private gem server, I created a Rake task that has all of the standard gem building tasks but with the **inabox** command twist.

{% highlight ruby %}
require 'rubygems'
require 'rubygems/package_task'
require 'bundler/setup'

def gemspec
  $mygem_gemspec ||= Gem::Specification.load("mygem.gemspec")
end

task :gem => :gemspec

desc %{Validate the gemspec file.}
task :gemspec do
  gemspec.validate
end

desc "Build but don't install gem"
task :build => :gemspec do
  sh "mkdir -p pkg"
  sh "gem build mygem.gemspec"
  sh "mv mygem-#{MyGem::VERSION}.gem pkg"
end

desc "Release new version of gem to HP Gem Server"
task :release => :build do
  sh "gem inabox pkg/mygem-#{MyGem::VERSION}.gem --host #{ENV['HPGEMS_URL']}" # See below for a description of the HPGEMS_URL 
end

desc "Install the gem"
task :install => :build do
  sh "gem install pkg/mygem-#{MyGem::VERSION}.gem"
end
{% endhighlight %}

The **HPGEMS_URL** environment variable needs to be set on your servers (usually in your bashrc) to the url where you installed the geminabox server. Then you can simply add this to the **Gemfile** if your gems:

{% highlight ruby %}
source :rubygems
source ENV["HPGEMS_URL"] if ENV["HPGEMS_URL"]
gemspec
{% endhighlight %}

And example of the .bashrc file might look like this:

{% highlight bash %}
export HPGEMS_URL=myprivategemserver.com
{% endhighlight %}

When you run bundle install on your gems, it will look for dependencies at [rubygems.org](http://rubygems.org) and your private gem server.

[HN Discussion](http://news.ycombinator.com/item?id=3975365)