---
date: '2010-09-08 17:08:50'
layout: post
slug: improved-deploycleanup-for-capistrano
status: publish
title: Improved deploy:cleanup for capistrano
wordpress_id: '478'
categories:
- Development
- Infrastructure
- Ruby
- Ruby on Rails
tags:
- capistrano
- rails
- ruby
- Ruby on Rails
---

We ran into a problem today where capistrano wasn't correctly cleaning up old releases on a 15-minute multi-host deploy.  It seems like the default deploy:cleanup task wasn't written with multiple hosts in mind.

Essentially what it does is list the contents of your releases_path for the first host in the list of hosts and assumes that all those individual release directories will be present on all other hosts in the current deploy.  This is a poor assumption.  What if one host isn't deployed to as frequently (perhaps a QA environment?).  These edge cases cause trouble with the default code.

What it should do is, for each host you're deploying to, check the releases_path for that host with keep_releases and delete only those old directories on just that host.  I re-wrote deploy:cleanup to do just that using some of the features of run() to execute commands only on specific hosts...

```ruby
  task :cleanup, :except => { :no_release => true } do
    count = fetch(:keep_releases, 5).to_i
    run "hostname" do |c, s, hostname|
      local_releases = capture("ls -xt #{releases_path}", :hosts => [hostname]).split.reverse
      if count >= local_releases.length
        logger.important "no old releases to clean up on #{hostname}"
      else
        logger.info "keeping #{count} of #{local_releases.length} deployed releases on #{hostname}"

        (local_releases - local_releases.last(count)).each { |release|
          run "#{sudo} rm -rf #{File.join(releases_path, release)}", :hosts => [hostname]
        }
      end
    end
  end
``` 
