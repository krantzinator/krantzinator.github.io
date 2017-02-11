---
layout: post
comments: true
title: How branches work in git and GitHub
date: 2017-02-11
category: blog
---
At the first meeting of AkronWiT'S current programming book club, we realized that the majority of people had either never used git before or weren't familiar with how to use git/GitHub for version control.

I had personally decided to make heavy use of git branches for this book club. The book we are using, Big Nerd Ranch's Guide to Front End Web Development, has additional challenges at the end of each chapter. The authors warn that you should keep your `challenges` work separate from your main work as the code you write for the challenges may break some of the base functionality of a project. Creating a branch named `challenges` seemed like the obvious choice.

However, when I suggested to the group to make use of branches, we realized we needed to take a step back and explain what branches were and how they work.

Things people did with branches at our book club:

- created a branch for the project we were working on, within the repository that housed the project we were working on

- removed master branch so as to only have branches house specific projects within the repository

Questions people had:

- does the use of branches change greatly when working alone on a project vs working in a team?

I'm going to address all three of these topics in a bit. First, I'm going to attempt a basic explanation on what exactly branches are and why they are useful.

Let's go with the obvious tree analogy.

When you initialize a git repository or create a new repository in GitHub, you are automatically set up with a branch called `master`. Think of `master` as the trunk of the tree. That is the main thing that you want to grow.

Along the way, the main `master` trunk needs help. In tree terms, it needs branches to help it make and store food so it can use it grow. In code terms, the `master` trunk needs side branches to contribute to _its_ growth. The side branches are where new code (food) is created and, when the `master` trunk is ready to absorb it and grow a little bit taller, then you move your code-food from the side branch into the `master` trunk. You do this through a process called _merging_, and we'll talk about how to merge side branches into your `master` in a bit.

For a good visualization of how this works, you can see [CodeSchool's Intro to Git](https://www.codeschool.com/courses/try-git). Go ahead and open that up in a new tab to go through when you finish reading this post.

That's the purpose of branching - to contribute to the growth of your main project, or `master` trunk. (Side note: technically `master` is just another branch. However, I quite like this _trunk_ analogy and am going to refer to it as the main trunk for the rest of this post. I'll explain in a bit why it's called `master` and why it's important.)

Now, why do we add new code in branches? Why not add it directly to `master`? This question has many answers.

**1**
When you are on a team, you are all contributing code to your `master` trunk. However, you're not all working on the same features at once. Think of the tree again. A tree grows multiple branches at a time, because it needs that many opportunities to collect food. In the same way, a project's repository will have multiple branches at once that house different new features to be added to the project. Let's say Chandra is working on Feature A, Stacy is working on Feature B, and Jasmine is working on Feature C. They can work on all these different features simultaneously by _branching off_ of `master`.

When Stacy finishes work on her `feature-b` branch, she can have her coworkers look at her code, run tests against it to make sure it doesn't break any `master` functionality, and then add her code from her `feature-b` branch into `master`. This is what's called a `merge`, and now the `master` trunk has all the code from `feature-b` inside it.

At this point, Stacy would delete `feature-b` branch. When she starts work on a new feature, she will make a new branch to work on that code.

**2**
When you are working alone on a project, branches are still useful. Let's say you have a project that is a live app. That app uses the code in your `master` trunk. Now, you don't want to make changes to your `master` unless you are sure they work, because otherwise your app will break and you'll get lots of angry customer emails. So you write a bunch of new code on your `feature-b` branch. You run your tests against it, and a bunch fail. You broke something! Luckily though, you broke it on a side branch. Since your application only looks at the code in `master` and not any of your side branches, your app is safe and sound.

Once you fix the broken stuff and get your tests to pass, and you're confident your new code won't break your app, you then merge your `feature-b` branch into your `master` trunk and everything is hunky-dory. You can delete `feature-b` and create a new branch to work on your next new feature.

#3#
The final use case I'm going to address is the use case I am encouraging people to apply in the book club. This is where you use branches to maintain a different version of your repository _alongside of_ your `master` trunk. Maybe think of this as those weird looking trees that have two trunks. Or really thick branches that are so big they have their own branches.

Here's how it plays out. We have our repository (or "repo" for short) "front-end-web-dev." By the end of the book, this repo is going to have four different directories, or folders, in it that each house a separate project.

This is where a lot of people got tripped up. Branches are _not_ for separate projects. You use **repos** and/or **directories** to house separate projects. Branches are for _code changes_ in your projects, not for the projects themselves. Your Ottergram project code goes in either a repository named "ottergram" or a folder named "ottergram" inside of your "front-end-web-dev" repository.

As we go along, we have the option of completing extra challenges at the end of each chapter. The book authors warned us that this challenge code can sometimes break our main project code, so we will want to keep them separate. Enter branches!

The main way this use case differs from the above two is we are not going to ever merge our branches into our `master` trunk, and therefore we also will not be deleting our branches. We want to keep them as alternate copies of our `master` code. Maybe the first branch we make is `ottergram-challenges`. When we start the next project, maybe we'll make a branch called `coffeerun-challenges`. I say "maybe" because, as many things are in writing code, naming of branches is a personal choice. Maybe you want one singular `challenges` branch for all the code for all the challenges. Maybe you want to create a new branch for each challenge, such as `ottergram-challenge-1`, `ottergram-challenge-2`, etc. Maybe you want to try a few different strategies before deciding on one -- which would be fantastic because this is all about learning.

This concept is also used in a team setting or in a workplace. For example, at my job we often push out new releases of software. When we push out version 8.8, we still need to keep the 8.7 code around because we still have customers using 8.7 and they will still need support. So we create a branch `8.7` to be a copy of the code exactly as it looked for the 8.7 software, and then we can add new code for the 8.8 version to `master` and know that our 8.7 code is safe and sound in its own branch.

___

OK! Those explanations got longer and longer as I went on. As you may have realized by now, git is very complex and powerful. But the basics are simple once you get them down, I promise!

I've addressed two of the topics I listed at the beginning. As for renaming your `master` branch to something other than "master", don't do that. As a beginner, you don't want to get into that maze. The short answer to why the main branch (or trunk!) is called "master" is because git has quite a commands that default to whatever branch is called `master`, and, I assume, because it's nice to have consistency -- you can jump into any codebase anywhere on GitHub or in a codebase that uses git and know that the `master` branch is the main one.

If you _really_ __really__ want to rename your `master` branch, [here's some explanation on how.](http://stackoverflow.com/questions/12759615/how-to-set-develop-branch-as-default-in-github-instead-of-master/12764517#12764517) I don't recommend it at this stage of your learning journey into git, but I'm neither am I going to keep you from experimenting.

I think that covers the broad concepts behind branching and the main use cases for branching. For more info on the specifics and some hands-on practice, check out that CodeSchool link above. I also recently found this ["Getting Comfortable with Git and the Command Line"](https://www.vikingcodeschool.com/web-development-basics/getting-comfortable-with-git-and-the-command-line) from Viking Code School. I haven't gone through it personally, but I trust VCS enough to recommend it here.

I want to keep going and explain how GitHub and git are actually not the same thing, but I won't. I need to work on the next chapters for book club now, for one thing. For another, git has many, many rabbit holes I could go down in explaining its nuances. (Kind of like real trees! Ahhh.)

In closing, I'll leave you with this: if you decide to have one main branch to house your work on the challenges (such as `challenges` or `ottergram-challenges`), you are going to run into the question how to merge your updated project code _from_ `master` _into_ `challenges`.

But wait, that's the opposite of how branches are supposed to be used! I'm supposed to merge code from branches into `master`, not the other way 'round!

That, my friend, is another topic. We'll probably have to get into it this coming week, so I may write another post for it. To that end - let me know if you have more questions on branching I can answer.
