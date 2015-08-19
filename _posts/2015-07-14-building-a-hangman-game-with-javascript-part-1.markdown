---
layout: post
category: learn to code, javascript
title: Building a Hangman Game with JavaScript - Part 1
date: 2015-07-14
---
Have you heard of CodeNewbie? I’ll rave about them pretty regularly on here. One fateful night, after joining their Slack community, I impulsively jumped on the Hangout for that night’s JavaScript Tuesday.

Lesson 1: follow your impulses!

(Don’t quote me on that.)

That night was the first in planning and building a project as a team. Let me tell you, team-based project work is a whole different world than working solo on projects. And it’s radically different than any online tutorial you’ve ever taken.

I’ve learned more in the last few weeks of working on this project than in the last year of studying concepts, reading documentation, and doing tutorials. Get your hands dirty, people!

We decided to build a Hangman game, and talked through all the different pieces we’d need to create. The JS-Tuesday host, Dan Berger, created issues of each item on the GitHub repo for tracking development progress. We’ve since moved everything to Trello, but you can still see the closed issues if you’re interested.

That process itself was crazy interesting to me. I tend to jump into projects head first and try to barrel through. That is not good practice for programming! More on how that can mess up a development process in my next post, where I talk about how many days it took me to build a couple of working JavaScript functions. (And they’re not pretty functions either!)

I’m still wrapping my head around how you can have several different people working on several different pieces, many of them with interdependencies, and have it all come out as one cohesive project.

Things I’ve learned so far:

- Limit your global functions; or establish what global functions and associated dependencies you will create as a team, before starting to build things
  - Side note: Dependencies generally means any other functions or elements someone has created that call, or use, your functions or elements
  - Additional side note: Calling a function means using a function; ex:
        {% highlight html %}
        for (var i = 0; i < 10; i++) {  myFunction();  }
        {% endhighlight %}
  - I just called a function!
- Using libraries like jQuery can increase the load time of your page, which is something to consider when building for performance. But of course, using libraries can also make your code more efficient, which decreases the load time of your page. Both. Do both.
- jQuery doesn’t have to invoke the $(document).ready() every time! This is the format I learned at Code Academy, and while I’m also learning you should invoke this statement prior to some functions for different reasons (clear as mud, I know), it’s not imperative. Depends on the timing of when things load on the page and when you want your function to run.
- jQuery can be interwoven into JavaScript! Say whaaaaat. It’s a library, so it’s a natural part of JavaScript, not a separate thing! I’m still not 100% clear on the differences and similarities of libraries as compared to frameworks, but FWIW. This function works:
  {% highlight html %}
  easyPuzzle = document.getElementById("easy-puzzle").addEventListener("click", function() {
   wordSelect(easyArray);
   for (i = 0; i < lettersToGuess.length; i++) {
   $('.letter-space').empty();
   $('.letter-space').append(lettersToGuess[i]);
   };
  });
  {% endhighlight %}

There is so much more to learn! I keep seeing these things called Grunt and Gulp (and Redis now) floating around, and I have a vague understanding of their existence, but that’s about it. And that’s OK for now.
There are also tools that will do the “build” of an app for you. We used Yeoman, which builds out the backend of the aforementioned Grunt and Gulp and importing different libraries and all sorts of fancy things. I don’t fully understand its importance yet, but I know I’ll learn more once we finish the application and talk about the hosting side of things.

I’m avoiding feeling overwhelmed by focusing on JavaScript, and throwing in little bits of things as they come up. I’m not going to worry about Gulp right now. I’ll get there, once I understand more of JavaScript’s depth and what it does. Then I’ll be able to more fully understand how Gulp interacts with it.

That’s what I’ve got so far! Still learning, still building. My next post, I want to show you what I’ve built so far, and admit to how long it took me to create. I’ll also let you know if my first pull request of my life is accepted into the project!
