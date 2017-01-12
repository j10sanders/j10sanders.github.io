---
layout: post
title:  My top two resources for intermediate-advanced Python programmers
category: Python, Testing
description: Read 'The Hacker's Guide to Python' and watch 'Getting Started Testing'
---


<!--description-->


# My top two resources for intermediate - advanced Python programmers

I'll start off by saying that the two best resources I've come across for becoming a better Python programmer take far larger time commitments than the two I'm writing about here.  Those are: attending [Recurse Center](https://www.recurse.com/) (although not Python specific), and doing Thinkful's ["Programming in Python"](https://www.thinkful.com/courses/learn-python-online/) course.  I plan on writing more about RC at some point, and if you are new to building web applications, I highly recommend Thinkful's course.  For now, I figured I would share the two less time-intensive resources I've found to be most helpful to becoming a better Python programmer.



## The Hacker's Guide to Python


A fellow RC'er, Pieter, let me borrow his copy of [The Hacker's Guide to Python](https://thehackerguidetopython.com/) (I recently bought my own copy).  I figured it would be nice to skim over - but I'd already been programming in Python for  a couple years, so I didn't expect to find it very insightful.  I was wrong.  Sure, there are chapters I skimmed over, but there is a remarkable amount of material that will help even veteran Python programmers up their game.

As an example, Danjou covers **"Managing API changes"**, a topic most Python books wouldn't touch, with depth and clarity.  

> "You can’t rely on developers to read your change log or documentation when they upgrade to a newer version of  your Python package. Python provides an interesting module called 'warnings' that can help in this regard.  This module allows your code to issue various kinds of warnings, such as `PendingDeprecationWarning` and `DeprecationWarning`.  These warnings can be used to inform the developer that a function they’re calling is either deprecated or going to be deprecated. This way, developers will be able to see that they’re using an old interface and should do something about it." 

He gives code examples and shows the warnings developers would relieve after calling a function with a warning.  __And that's just one of many practice he discusses for managing API changes.__

A few other topics that I found particularly insightful (italicized words are mine):

**Sharing your Work with the World** - *about best practices for publishing to PyPI*

**Entry Points** - *have you ever used `entry_point_inspector`?*

**Named Tuples and Slots** - *how to use objects in a way that uses half the default behavior (with the `__slots__` attribute or `namedtuple` class from the `collection` module).*

**Using six** - *how to use six - a module that implements compatibility for Python2 and 3*

This is just a taste of the topics that Danjou covers.  He even devotes a lot of time discussing functional programming approaches that can be used in Python.  I know [some people](http://www.effectivepython.com/) prefer using list comprehensions to filter or map, but Danjou makes a good case to not only use these, but to learn LISP while you are at it (and I agree)!

Lastly, he peppers interviews with accomplished Python programmers throughout the book, and these are well thought out, enlightening conversations that add a lot of value and help keep a nice rhythm to the overall text.  The only downside I found is that Danjou's English is at times a bit shaky.  It's always clear what he is trying to say, but sometimes the grammar can be a bit jarring.


## Getting Started Testing

The second resource is Ned Batchelder's (the guy who wrote Coverage) ["Getting Started Testing"](http://nedbatchelder.com/text/test0.html).  You should probably just watch it now.  If you think you already write great tests and don't need to watch this video which is an introduction, I’d say to watch it at [2x speed](https://chrome.google.com/webstore/detail/video-speed-controller/nffaoalbilbmmfgbnbgppjihopabppdk?hl=en).  The value of this video isn't in showing how tests are written - it's how Ned builds up the case for best practices.  He starts off by showing how you can test code without actual test cases at all.  He quickly find disadvantages to this approach (not repeatable and labor intensive), so he make a separate file to hold test code... and finds issues with this approach (tests don't compare results to expected results), and continues in this manner over and over again, until he hits modern best practices for tests, including automated testing, correct and careful usage of Mocks, and even writing tests for your test helpers!

The way he builds up the case for current best practices is elegant, convincing, and helped me become a more thoughtful tester.  Ned is a force in the Python community, so I recommend watching and reading everything he has put out, but I especially recommend this lecture as a start.
