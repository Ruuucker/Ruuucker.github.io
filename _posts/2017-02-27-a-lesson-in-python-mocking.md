---
layout: post
title:  "A Lesson in Python Mocking"
date:   2017-02-27 19:34:13 -0500
categories: python mocking
---
Recently while writing some tests to check that methods were called/not called in a Python 2.7 application, I noticed that [assert_called()](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_called) and [assert_not_called()](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_not_called) were always passing when they shouldn’t have been. Some googling lead me [here](http://engineroom.trackmaven.com/blog/mocking-mistakes/), where I read that those methods don’t exist as of 12/1/15. That begged the question: if in fact these assertions don't exist, why would they pass?

After some more fiddling, I managed to produce the error I expected: Setting `autospec=True` caused: `AttributeError: 'function' object has no attribute 'assert_called_once'`.

Okay, so it looks like the assertions really *don't* exist.

To confirm: `pip list mock` showed I had Mock version [1.0.1](https://github.com/testing-cabal/mock/tree/1.0.1) — and a quick perusal of the source verified that sure enough, they really didn't exist!

That was a valuable lesson in mocking learned. Without [autospeccing](https://docs.python.org/3/library/unittest.mock.html#auto-speccing), I may have gone one to write a number of additional false-positive tests!
