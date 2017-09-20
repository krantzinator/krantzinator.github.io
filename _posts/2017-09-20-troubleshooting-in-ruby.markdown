---
layout: post
comments: true
title: Troubleshooting (in Ruby)
date: 2017-09-20
category: blog
---
Troubleshooting is often intimidating for people new to code. It's easy to feel lost and have no idea where to start. To that end, I wanted to share a recent troubleshooting attempt of mine at work.

I am upgrading our gems in our test environment to Ruby 2.3.3, and as part of that, I've been working on a particular gem that relies heavily on ActiveRecord. I worked through all the deprecation errors to get our syntax up to current ActiveRecord levels, and then ran `bundle exec rake spec` as the final step before building the gem.

Except they all failed. Wooooo.

**Why did they fail?**</br>
I scroll up my list of failures and see a bunch of `FactoryGirl::DuplicateDefinitionError: Factory already registered`. Most if not all of my tests are failing from this. I do a few searches on this but can't find anything helpful.

**Is this the only failure?**</br>
I scroll all the way to the top and see that one test does not fail due to `FactoryGirl::DuplicateDefinitionError`. The very first test fails due to `ActiveRecord::Base.establish_connection(CONNECTION_NAME) ActiveRecord::AdapterNotSpecified: database configuration does not specify adapter`.

Ah. This would cause problems on downstream tests, if I can't even establish the starting connection to the test database. This is could be fueling the FactoryGirl errors. I don't know anything about FactoryGirl, and the `DuplicateDefinitionError` sounds like it's separate from failing to establish a connection at all to the database, but what they hey. Fixing FactoryGirl definitely won't fix ActiveRecord, and there's an off-chance fixing ActiveRecord will give me a boost with figuring out FactoryGirl.

**What does the internet say?**</br>
I grab the failure line, and find a lot of things along the lines of [this StackOverflow article](https://stackoverflow.com/questions/23336755/activerecordadapternotspecified-database-configuration-does-not-specify-adapte#23338963).
It doesn't seem to be much help, becase my `config.yml` file looks good, and I have sqlite3 installed. So what do I do now?

**What's the top line in the stack trace?**</br>
That points to my `lib/company_name/gem_name.rb` file. I go there.

**Is the way we're calling `establish_connection` different now?**</br>
In this file, I see our setup of `def establish_connection` which includes `ActiveRecord::Base.establish_connection(CONNECTION_NAME)`. OK, cool. This is the exact line from my error message. I've found the culprit.

**What is wrong with this line?**</br>
I assume this is an issue with our syntax again, due to deprecation, and I spend a lot of time searching ActiveRecord docs to try to understand A) what's happening in general since I don't fully understand this thing, and B) see if there have been breaking syntax changes in creating connections since the version we used to use and the most current version.

I find nothing.

**What else could it be?**</br>
The syntax all looks fine from what I've learned about ActiveRecord. We're feeding in options to establish a connection based on the given database. Something must be breaking further up the chain. Back to the stack trace.

**Where does the configuration specify an adapter?**</br>
Since this question is based off of the literal error message, it's where I probably should have started anyway. Well, that's embarrassing.
Looking at my stack trace, I see that the arguments being passed into the `ActiveRecord::Base.establish_connection(CONNECTION_NAME)` come from a `integration_spec.rb` file. However, the error is in the `require` statement on line 1, not with any way that file is calling this function specifically. Huh.

**What is being required that's failing?**</br>
This file is requiring `spec_helper.rb`. That file is also listed in my stack trace. I'm getting closer.
In `spec_helper.rb`, I look for the line where it's calling `establish_connection` - line 29 - as well as the line listed in the stack trace - line 30.

**What argument am I giving this call of `establish_connection`?**
That would be a `dbconfig` variable that I declare a couple of lines up.

**OK, so what is that `dbconfig` variable resolving to?**</br>
`dbconfig` makes use of `StandaloneMigrations::Configurator.load_configurations[:test]`.

**Is the syntax right?**</br>
I am not familiar with this `standalone_migrations` gem. A couple minutes of searching assures me the syntax is not deprecated.

**I do have the `standalone_migrations` gem installed that I'm requiring, right?**</br>
`gem list` shows this gem, with the most current version.

**What is `StandaloneMigrations::Configurator.load_configurations[:test]` returning?**</br>
I throw in a `puts "dbconfig = #{dbconfig}"` and re-run `bundle exec rake spec`. I get `dbconfig = ` in my test output. (As in, my `dbonfig` variable is empty, nil, nada, no-way.)</br>
I open up an IRB, `require 'standalone_migrations'`, and run `StandaloneMigrations::Configurator.load_configurations[:test]`. Predictably, I receive `nil` as the result.</br>
I double check my `config.yml` file, which houses the settings for my `test` environment. Everything checks out. Huh.

**What's an obvious thing I can try?**</br>
You know, I've been reading a lot lately about how you should use `["test"]` syntax instead of `[:test]` because the `:` syntax is kinda unique to Ruby and would confuse a new person.

**What would happen if I change `[:test]` to `["test"]` in my IRB call?**</br>
I try it, and get back `{"adapter"=>"sqlite3", "database"=>"path/to/db/test.sqlite3", "pool"=>5, "timeout"=>5000}`</br>
WOOHOOO Y'ALL.</br>
Time to update my `spec_helper.rb`.</br>
My elation is short-lived, as I still get the same failure.
However, my `puts` statement is now returning `dbconfig = {"adapter"=>"sqlite3", "database"=>"path/to/db/test.sqlite3", "pool"=>5, "timeout"=>5000}`, so that's something.</br>
Huh.</br>
The stack trace is the same. So the error was before that `dbconfig` line...which makes sense since the `dbconfig` stuff was on lines 26-29, and my stack trace clearly points to line 30 as the culprit. However, my efforts were not in vain because line 30 uses the variable `connection` which is set on line 29 like so: `connection = Vht::ConfigAdapter.establish_connection(dbconfig)`.</br>
So I believe I am just a step ahead in my troubleshooting. Now to take a step back for the problem in front of my face.

**What does line 30 say?**</br>
`database_cleaner = DatabaseCleaner[:active_record, {:connection => connection}]`

**Is this another problem of `:` syntax versus `""`?**</br>
To the IRB, Robin!</br>
`require 'database_cleaner'`</br>
`DatabaseCleaner[:active_record, {:connection => connection}]`</br>
`NameError: undefined local variable or method 'connection' for main:Object`</br>
Oh, duhhh.
I try again, running the code defining `dbconfig` so I can feed it into the `connection` variable definition.</br>
`connection = CompanyName::GemName.establish_connection(dbconfig)`</br>
`NameError: uninitialized constant CompanyName`</br>
DUH again. A glance up at all the `require`s in this `spec_helper.rb` and -</br>
`require 'company_gem_name'`</br>
`connection = CompanyName::GemName.establish_connection(dbconfig)`</br>
`ActiveRecord::AdapterNotSpecified: database configuration does not specify adapter`</br>
But it does. I is confused.

**Why isn't my `sqlite3` adapter coming through?**</br>
I double-check my `adapter` setting to make sure. Yep, it's right there, in the output of setting my `dbconfig` variable: `"adapter" => "sqlite3"`.</br>
Options as I see them:
- sqlite3 isn't installed
- something weird with syntax `:` vs `""` making it unable to read the adapter setting that is clearly there

**What's the status of sqlite3?**</br>
`gem list sqlite3` shows `sqlite3 1.3.13 x64-mingw32`.</br>
Ugh. I hope this isn't a DevKit problem. Blahhhhh. I recently learned that on Windows (which I am on), DevKits for different Ruby versions behave differently, and the `x64-mingw32` tells me this gem was compiled by DevKit, so maybe I have some funny business going on there.</br>
Some searching turns up that if it couldn't find sqlite3 at all, I would get an error like `sqlite3 is not part of the bundle`

And guess where I am. Back to searching for that error phrase. I don't feel like this time was wasted though, because I've learned more about how ActiveRecord works, and I have a more full picture in my mind of how these pieces are working together. I _am_ however, at a bit of a loss.

**Ask for help**</br>
It's time.

**How I ask for help**</br>
I don't go through _all_ of that above work. I explain where my mind is:
- I'm getting this error message `ActiveRecord::AdapterNotSpecified: database configuration does not specify adapter`
- I am listing an `adapter` of `sqlite3` in my `config.yml` and it's being pulled in correctly, as verified by my code runs in IRB
- `sqlite3` is in my `Gemfile.lock` as well as in my `gem list`, so I believe it's _there_
- One thing I'm curious about: in my `.gemspec` file that pulls in dependencies, there are `s.add_dependency` and `s.add_development_dependency`. Internet searching shows me these are the two options, and there is no `.add_test_dependency` option. Are the `development_dependency` gems (of which `sqlite3` is one) being used in my test setup?
- Halp.

-------------

OK, I'm back. I have learned things!</br>
The way this is set up, it is theoretically pointing to a `test` version of a `sqlite3` database. That's what it's missing.</br>
Theoretically.</br>
HOWEVER.</br>
Under my `db` folder, in the same folder as my `config.yml` no less, I see, plain as day, the `test.sqlite3` database. In fact, I earlier created this as part of my `db:migrate` and `db:seed` commands. And this matches the path that is spit out when I initialize that `dbconfig` variable. So that is what I am sending to `establish_connection`.

**So. Why is ActiveRecord not seeing this database?**</br>
For the hell of it, since all my configurations have the same setup, and to get ideas out of my head, I'm going to run those IRB commands with "development" instead of "test". Be back in a sec.

--------------

OK same problem. I just had to make sure because I actually created my `test.sqlite3` by copy/pasting my `development.sqlite3` and I don't know, maybe the copy/paste didn't work right. Whatever.</br>
Back to Square 1. Well, I'm at least at Square 2 now. Insofar as I have eliminated options.</br>
BLAH.

I'm going to take a break here. I have some other things I need to do, and then I'm going out for lunch with some co-workers. I'll be back.

------------
------------
------------

The error messaging in IRB has an ActiveRecord-specific stack trace. The top of that stack points me to line 170 of ActiveRecord's `connection_adapters/connection_specification.rb`. Going there, I see this error message is raised `unless spec.key?(:adapter)`, with `spec` being set as `spec = resolve(config).symbolize_keys`. `config` is an argument fed into the function which is also, and totally not confusingly, called `spec`.</br>
Here is the portion of ActiveRecord's `establish_connection` that I am hitting. My error is happening on the last line of this, as `resolver.spec` takes us to the aforementioned stuff where `.symbolize_keys` happens and then stuff fails.
```
def establish_connection(spec = nil)
  raise RuntimeError, "Anonymous class is not allowed." unless name

  spec     ||= DEFAULT_ENV.call.to_sym
  resolver =   ConnectionAdapters::ConnectionSpecification::Resolver.new configurations
  # TODO: uses name on establish_connection, for backwards compatibility
  spec     =   resolver.spec(spec, self == Base ? "primary" : name)
  ```

But back to that `unless spec.key?(:adapter)` line. I do, quite clearly, have a key of `"adapter"`, when I set my `dbconfig` variable to my test configuration. Let me try manually symbolizing the keys of that output and see if I get back something wonky.</br>
Nope, sure don't. I get back `{:adapter=>"sqlite3", :database=>"path/to/db/development.sqlite3", :pool=>5, :timeout=>5000}`.
And when I run in IRB `spec = dbconfig.symbolize_keys` and `spec.key?(:adapter)` it all checks out.</br>
So WHY IS THIS HAPPENING.</br>
Well. ActiveRecord doesn't do a strict `spec = dbconfig.symbolize_keys`. It is actually running `spec = resolve(dbconfig).symbolize_keys`. Time to dig into this `resolve` business.

-------------

OK back from lunch.

**What resource haven't I used yet?**</br>
The _Issues_ on GitHub!
I found [this issue](https://github.com/rails/rails/issues/11609) which linked to [this gist](https://gist.github.com/drogus/6087979) which had a comment by this wonderful [jehughes](https://gist.github.com/jehughes) which said:
>>>"I'm using Ruby 2.3.1 and found I needed to change the env variable to a symbol when making the connection:
`ActiveRecord::Base.establish_connection DatabaseTasks.env.to_sym`
Otherwise I was getting a missing adapter error."

Ahhhhhh.</br>
After a minor flub with forgetting that parentheses aren't required in Ruby and that the `DatabaseTasks.env.to_sym` was an argument and not a separate method, I implemented this lovely fix, which involved changing `ActiveRecord::Base.establish_connection(CONNECTION_NAME)` to `ActiveRecord::Base.establish_connection(CONNECTION_NAME.to_sym)`.

Tested in IRB. Got back a whole object rather than an error message about missing a database adapter.

Changed my code. Ran `bundle exec rake spec`.

And look at that! Now I have a whole _slew_ of failing tests! A quick glance through the ~100 failing tests shows lots of syntax deprecation. Here's hoping my full update of this gem is Ruby 2.3.3 is in sight.

*knock on wood*

Oh I forgot to mention -- the deprecation errors I worked through at the start of this? Yeah those were just from getting the database to seed correctly. Now the _real_ deprecation upgrade work begins.

_As a final note, my troubleshooting is still a work-in-progress. I keep reminding myself to **check the Issues first!** These are at least as helpful as StackOverflow, and rarely show up as well in blanket internet searches, whether using Google or DuckDuckGo or otherwise. CHECK THE ISSUES, SELF. CHECK THE ISSUES._
