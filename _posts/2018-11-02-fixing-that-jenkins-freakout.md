---
layout: post
comments: true
title: Fixing that Jenkins freakout
date: 2018-11-02
category: blog
---

Below you will find the transcript of a story I shared with an old co-worker at the time that it happened. If you've ever wrestled with Jenkins and/or Jenkins ghosts, you may commiserate a bit with me as you read it.

~

well the short of it is, [redacted] was mad that there were a ton of regression tests backed up, so he forced jenkins to restart in order to cancel all the jobs, not knowing that you need to cancel all the jobs before stopping/restarting jenkins.

then when it came back online, a bunch of plugins and probably other things were stuck in a not good, Very Bad state

and we didn't notice until someone came by my desk to ask how we use boolean tags in jenkins

and i went to show them, but 😱 the Conditional Steps option was gone. and it was gone from _every_ job

after a bit of sleuthing and tying this problem to [redacted]'s restart of jenkins -- verified by seeing that NONE of the jobs starting at ~9:30am onward made use of any conditional steps -- i had to reset jenkins to the last pushed up branch

and then i thought it was all fixed

but then no jobs would kick off

so then i read the jenkins log and saw that the error "can't start job 34556 it already exists" were happening

_(dear reader: we did keep our Jenkins config backed up in git, and I thought that would save me here; you may be able to guess it did not.)_

i learned that when you revert to a previous branch of jenkins, that's great except all the build information is kept in the dungeons of jenkins' memory, so once i restarted jenkins with last night's backup branch, it auto-populated records of all the jobs we had run today.
except those builds weren't there anymore.
weeeeeeeeee.
i noticed -- by `cd`'ing into a `builds` subfolder (i.e. `/var/lib/jenkins/jobs/DeploySingleServer_clean/builds`) that all the jobs whose ghost records from today were still around had a create date of 16:31
so i created a `timestamp` file with the date of today and time of 16:30
i then ran this handy little script in the `/var/lib/jenkins/jobs` directory as the jenkins user to delete _only stuff from a specific build number's folder_ (important so you don't lose config data)
`find . -regextype sed -regex ".*/builds/[0-9]\+" -newer /tmp/timestamp -exec rm -r {} \;`

and that was my whole afternoon 🎉
