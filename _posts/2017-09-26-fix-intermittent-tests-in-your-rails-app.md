---
layout: post
title: Fix intermittent tests in your Rails App
date: September 26, 2017
permalink: fix-intermittent-tests-in-your-rails-app.html
---

Earlier today I had an intermittent test that when I run the whole test suite,
the test sometimes passes, sometimes not. But when I run the single file which
causes the error, it passes by itself. What's happening?

This was my first time having this problem and I was just gonna dump it to fix
it later bin until I found minitest_bisect. So what is [minitest_bisect][1] asd
you ask?

>
  **minitest-bisect** helps you isolate and debug random test failures.
  If your tests only fail randomly, you can reproduce the error consistently
  by using `–seed <num>`, but what then? How do you figure out which
  combination of tests out of hundreds are responsible for the failure?
  You know which test is failing, but what others are causing it to fail
  or were helping it succeed in a different order? That’s what minitest-bisect
  does best.


It is quite easy to install, just add `gem minitest-bisect` to your `Gemfile` and you’re good to go. After that run your test again and when it fails, note the seed number where it fails.

```
project_name master % bin/rails test
Running via Spring preloader in process 19316
Run options: --seed 30692
# Running:
........F
```

Then after seeing your test fails, run it again with `minitest-bisect`.

```
bundle exec minitest_bisect --seed 9808 -Itest test/
reproducing... in 3.08 sec
Reproduction run passed? Aborting.
Try running with MTB_VERBOSE=2 to verify.
```

You may see some problems at first when running it like this, so we can just follow and add the `MTB_VERBOSE=2` to our command.

```
MTB_VERBOSE=2 bundle exec minitest_bisect --seed 9808 -Itest test/
lib/rails/test_unit/minitest_plugin.rb:62:in `plugin_rails_options': invalid option: --server (OptionParser::InvalidOption)
```

Now, if you see an error with `OptionParser::InvalidOptionlike` above, it means that you are running Rails version ≤5.1.2. There’s [pending issue][2] that can be solved by upgrading to at least 5.1.3. Once you do that, run again the command. Don’t be surprised if it looks like it’s looping (running the test again and again). Wait for it.

```
bundle exec minitest_bisect --seed 9808 -Itest test/
...
...
...
Final reproduction:
Run options: --seed 9808 -n "/^(?:test_new_employees_sorts_specified_time))$/"
```

You’ll see this `Final reproduction:` block with the `Run options` take note of this whole command. This is the command you’re supposed to run every time to fix the intermittent error you are currently getting.

```
bin/rails test test/ --seed 9808 -n "/^(?:test_new_employees_sorts_specified_time))$/"
```

For my problem, I need to change how I deal with time and my sample data is off so I had to fix that too.

In case you have a problem dealing with time, this [article][3] by [Nicklas Ramhöj][4]
is a good read.


[1]: https://github.com/seattlerb/minitest-bisect
[2]: https://github.com/seattlerb/minitest-bisect/issues/16
[3]: https://www.varvet.com/blog/working-with-time-zones-in-ruby-on-rails/
[4]: https://twitter.com/ramhoj
[5]: https://www.pexels.com/photo/man-in-white-shirt-using-macbook-pro-52608/
