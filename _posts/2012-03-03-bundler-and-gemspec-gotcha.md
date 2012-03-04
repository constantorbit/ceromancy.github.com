---
layout: post
title: "Bundler and gemspec gotcha"
category: Technical
tags: [Bundler, Gems, Gemspec]
---
{% include JB/setup %}

I recently fixed some issues with [Michael Dvorkin's](https://github.com/michaeldv) awesome [awesome_print](https://github.com/michaeldv/awesome_print), but
 as soon as I forked it and tried to use the fork directly in my rails project
(e.g. `, :git =>` in my Gemfile, not loading from rubygems) it stopped working.

Very strangely, just silently stopped working.  Bundler still happily installed it, `require 'awesome_print'` didn't complain, but when I tried to use it I got something like:

    [0, 1].ai
    NoMethodError: undefined method `ai' for [0, 1]:Array

I got hung up for a while thinking the gem wasn't getting built properly, as michaeldv did have some atypical code in there (using `Rake::FileList`, for example, to get lists of files instead of `git ls-files`
that seems to be more typical)

Nothing was helping, though, so I dove into what was happening when the gem was being loaded.

The main entry point which `require`'d all the subsidiary files was wrapped in:

    unless defined?(AwesomePrint)
    [...]
       require File.dirname(__FILE__) + "/awesome_print/inspector"
    [...]
    end

which [an earlier commit](https://github.com/michaeldv/awesome_print/commit/24b6192f57d2e3b7483057c8c4e7024d0ff6d3cc) made clear
was to avoid loading the gem multiple times.  Made sense.  But what became clear is that the `AwesomePrint` constant was always already defined when this code ran.  Huh?

It turns out the culprit was `awesome_print.gemspec`.  It has:

    require File.dirname(__FILE__) + "/lib/awesome_print/version"

in order to supply a version to `Gem::Specification`.  However, this was instantiating the `AwesomePrint` module.

This wasn't an issue when the gem has been installed from rubygems.org, but when bundler loads the version it got from github it must be executing the gemspec every time.

I've fixed this in [my fork](https://github.com/ceromancy/awesome_print) and have [filed a bug](https://github.com/michaeldv/awesome_print/issues/79).  I'll need to clean up my
fork (since I have other changes) before I submit a pull request for this.