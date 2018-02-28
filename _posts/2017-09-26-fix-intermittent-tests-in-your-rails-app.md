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

<script
src="https://gist.github.com/iamarmanjon/fc3edbbdc9b9039170f2ebc814e14f97.js"></script>

Then after seeing your test fails, run it again with `minitest-bisect`.

<script
src="https://gist.github.com/iamarmanjon/0ab4fea5765edd753acf9426fd8472b9.js"></script>

You may see some problems at first when running it like this, so we can just follow and add the `MTB_VERBOSE=2` to our command.

<script
src="https://gist.github.com/iamarmanjon/4cd25c4f7646a9eb6076eeb65618eb44.js"></script>

Now, if you see an error with `OptionParser::InvalidOptionlike` above, it means that you are running Rails version ≤5.1.2. There’s [pending issue][2] that can be solved by upgrading to at least 5.1.3. Once you do that, run again the command. Don’t be surprised if it looks like it’s looping (running the test again and again). Wait for it.

<script
src="https://gist.github.com/iamarmanjon/e4c89d99a01ecc8f9f85a78182640147.js"></script>

You’ll see this `Final reproduction:` block with the `Run options` take note of this whole command. This is the command you’re supposed to run every time to fix the intermittent error you are currently getting.

<script
src="https://gist.github.com/iamarmanjon/3e7e028db77543129486b6835858cc18.js"></script>

For my problem, I need to change how I deal with time and my sample data is off so I had to fix that too.

In case you have a problem dealing with time, this [article][3] by [Nicklas Ramhöj][4]
is a good read.


[1]: https://github.com/seattlerb/minitest-bisect
[2]: https://github.com/seattlerb/minitest-bisect/issues/16
[3]: https://www.varvet.com/blog/working-with-time-zones-in-ruby-on-rails/
[4]: https://twitter.com/ramhoj
[5]: https://www.pexels.com/photo/man-in-white-shirt-using-macbook-pro-52608/
