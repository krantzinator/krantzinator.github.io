---
layout: post
title: Learnings on Mac and the Default Python
date: 2016-09-05
category: blog
---
Today I learned to not take the first answer you find on StackOverflow at face value. Combine it with a bit of critical thinking to make sure it's what you need.

The sequence of events went something like:

**Started my mentorship through Recurse Center's [RCStart](https://www.recurse.com/blog/99-free-one-on-one-mentorship-for-new-programmers) program**

We jumped right into web data scraping basics, spinning up a virtualenv and installing numpy, pandas, and matplotlib. (Aside: I knew both my mentor and I had recently returned from travels, and as an intro I asked where she had been spending her time. She responded, "Glacier National Park," and I immediately knew this mentorship is going to go swimmingly. That's where I had just returned from as well!)

My mentor had a hard stop at 5:50pm, so she left me with the following error to figure out, which I hit when importing matplotlib into my virtualenv's REPL:

`RuntimeError: Python is not installed as a framework. The Mac OS X backend will not be able to function correctly if Python is not installed as a framework. See the Python documentation for more information on installing Python as a framework on Mac OS X. Please either reinstall Python as a framework, or try one of the other backends. If you are Working with Matplotlib in a virtual enviroment see 'Working with Matplotlib in Virtual environments' in the Matplotlib FAQ`

**Learned that matplotlib requires Python framework access, which isn't available in a virtualenv**

The above error message was fantastic, because it pointed me to exactly where I needed to start fixing this problem. I followed [the included instructions](http://matplotlib.org/faq/virtualenv_faq.html) for creating a PYTHONHOME script to include in your virtualenv's `bin`.

**Hit errors saying Python2.7 wasn't in my `/usr/local/bin` folder**

I didn't save these errors (maybe I should add that to lessons learned? Save all errors until the problem is solved? Is that a thing people do?), but they essentially said there was not python2.7 in my `/usr/local/bin` folder, which the above-created PYTHONHOME script pointed to.

**Accessed both `/usr/local/bin` and `/Library/Frameworks/Python.framework` - no Python2.7**

This is when I freaked out. I saw all the Python 3s I have downloaded over the past months in `/usr/local/bin`, and I saw 3.4 and 3.5 in `Python.framework/Versions`. But where the heck was my Python2.7??

**Got confused and worried**

I *know* Mac needs Python2.7 to run its apps, and that you should never, ever remove the default Mac Python unless you *really* know what you're doing. I'd never even considered doing this. My next thought was, *Did installing additional versions of Python somehow remove 2.7?*

This was followed up by wondering why A) all my Mac apps functioned normally, and B) why when I open a REPL it still, with no problems, defaults to Python 2.7.

I figured 2.7 was *somewhere*, but why wasn't it showing up in the above mentioned directories? Had I removed links somewhere but not the root program, allowing my computer to still function but I couldn't link to the python install, as I was trying to do in the PYTHONHOME script?

**Incessant Googling**

Herein lies the problem with Googling: sometimes you Google the wrong things.

My searches centered around asking why python2.7 was missing from `/usr/local/bin` and how to restore it to that location. I found a lot of answers about how removing it was the dumbest thing you could do (I *know*! I didn't, I swear!), and also suggestions on reinstalling your most recent OS X to restore system defaults, including system Python defaults.

Embarrassingly, I was going to go through with a [reinstall of my OS](https://support.apple.com/en-us/HT204904). Mac makes this easy to do, and you theoretically don't lose any of your files or data. Thankfully, I hit a snag that made me put off doing this. Details unimportant there, but I am SO GLAD that snag happened.

*How close I came to taking an involved action that wouldn't have fixed my problem has made me much more cautious of the "Blindly copy-paste from StackOverflow!" mentality programmers joke about so much. Again, you don't know if you're asking the right question.*

**Finally Googled the right thing**

I continued searching for a solution.

The magic Google was: "restore python2.7 to frameworks mac."

The first result was an answer to the question: [How to restore python on OS X Yosemite after I've deleted something?](http://stackoverflow.com/questions/26917765/how-to-restore-python-on-os-x-yosemite-after-ive-deleted-something).

This question was asking similar questions to what I had been wondering, as the questioner specifically referenced `/Library/Frameworks` and `/usr/local/bin`, which were the two paths I had been most concerned with.

Luckily for both of us, someone answered our unasked question as well, which should have been, "Is there somewhere else I should look for the default Mac installs of Python?"

Thank you, [abarnert](http://stackoverflow.com/users/908494/abarnert). I created a StackOverflow account so I could upvote your answer. (I know, I should have created one a while ago. This experience fixed so many problems!)

The most pivotal piece of their answer was:

>Let's start off with this:

>`/Library/Frameworks/Python/2.7` is neither the Apple Python nor the Homebrew Python. You apparently installed a third Python, maybe the one from the official python.org binary installers. Removing that one won't affect the Homebrew one.

>`/usr/local/bin/python` is not the Apple Python either. It may be a symlink to your third Python or to the Homebrew Python, but it's not from Apple.

>Here's where each Python goes:

>Apple's Python is in `/System/Library/Frameworks/Python/2.7`. It also includes various wrapper executables in `/usr/bin`, including `/usr/bin/python`, that point at the `/System` framework. Any extra stuff you install with that Python (e.g., via `easy_install` or `pip`) that includes executables or scripts will go into `/usr/local/bin`, not `/usr/bin`, but Apple's pre-installed stuff never does.
Most third-party binary installers install into `/Library/Frameworks/Python/2.7`. Different versions can optionally add the framework's bin directory to your path, or symlink the binaries into `/usr/local/bin`.
Homebrew installs to somewhere like `/usr/local/Cellar/python/2.7.8`, then symlinks various executables and scripts into `/usr/local/bin`.
So, the fact that you're trying to get back to the Apple Python by making sure `/usr/local/bin` is on your PATH is already heading in the wrong direction.

**Queue Hallelujah chorus**

I now knew my Mac default Python was safe and sound, *and* I knew how to fix my matplotlib problem.

I updated my path in my PYTHONHOME executable from `/usr/local/bin` to `/usr/bin`. Ta-da! I can now run a REPL in my virtualenv with matplotlib working in all its glory.

Well, once I figure out how matplotlib works.

To the docs, Robin! We've still got work to do.
