I wanted to update my rspec tests to instantiate my class at the top of my test suite.

Before:
```it "does things" do
init = Module::Module::Class.new
expect(init.do_the_thing).to eq("the thing")
end

it "does MORE THINGS" do
init = Module::Module::Class.new
expect(init.do_things).to eq("more things")
end
```

After:
```init = Module::Module::Class.new

it "does things" do
expect(init.do_the_thing).to eq("the thing")
end

it "does MORE THINGS" do
expect(init.do_things).to eq("more things")
end
```

I added `init = Module::Module::Class.new` to the top level, then changed one test to use this global variable and see if it worked. It did! I then did a search-and-replace-all to sub all occurrences of `Module::Module::Class.new` to be `init`.

Run tests!

All tests fail!

Duh. My global variable now read `init = init`.

Happy Tuesday, y'all.
