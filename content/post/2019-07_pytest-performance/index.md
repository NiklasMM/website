+++
title = "Profiling and improving the runtime of a large pytest test suite"
subtitle = ""

# Add a summary to display on homepage (optional).
summary = "In this post I summarize the things I learned when trying to improve the runtime of the testsuite for one of our projects at work."

date = 2019-06-30T13:06:04+02:00
draft = true

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["niklas"]

# Is this a featured post? (true/false)
featured = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = []
categories = []

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
[image]
  # Caption (optional)
  caption = ""
  preview_only = true
  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""
+++

At work we have a large code base for one of our main products. This code base has been growing for about 5 years now and there is no end in sight.
Naturally, the test suite required to test all that code has been growing with it. It started out using the `unittest` module from Python's standard library and was migrated to [pytest](https://www.pytest.org) as the testing framework and test runner a couple of years ago. Pytest is the de-facto standard for automated testing in Python today. Not only is it very powerful and extendible, but it's also a great example for a thriving open source project, adding new features and spitting out releases faster than many commercial products.

The test suite now contains about 1500 tests, many of which go beyond the traditional concept of a unit test. We use the web application library [Werkzeug](https://werkzeug.palletsprojects.com) for our project, which provides a nice test client to simulate HTTP-requests during tests. Such tests are probably better described as integration tests, though the lines between the different categories of tests are rather blurry.

Naturally, with the increasing number of tests the execution time also went up. In this article I want to discuss if this is even a problem, what we can do about it.

## The scenario

As mentioned above the test suite in question is a bit of a mixed bag of "true" unit tests and integration test which simulate a lot of the real application. This includes http requests and database access. We use a sqlite in-memory database for the tests, while the production db is usually managed by postgreSQL. Using different SQL dialects in tests and production is arguably problematic, but that's a topic for another time.

We make heavy use of pytest's [fixtures](https://docs.pytest.org/en/latest/fixture.html) to provide tests with streamlined access to the application under test. For example the `client` fixture creates a db engine, initializes a session and creates all tables, before it is connected to a Werkzeug test client.

When I started diving into this topic the test suite had a runtime of about 5 minutes on a reasonably powerful PC without the use of pytest-xdist (i.e. one one machine and core only)

## Why even bother?

First of all, I'd like to discuss the motivation behind this project. Why even care if the test suite is slow, just let your continuous integration server handle it? Can a runtime of 5 minutes even be considered slow? When discussing these questions with coworkers and on twitter, it became clear that it's very much a matter of perspective. For some people it's fine if the test suite takes long others are annoyed by it. Some say 5 minutes is a great runtime others find it way too slow. So here are my reasons for trying to improve the test suites runtime:

1) A slow test suite slows me down during development and bugfixing on my local machine. While it is true that I rarely run the whole test suite while actively working on a piece of code, I do like to practise test-driven-development and therefore run the test very frequently. Therefore even small improvements in the performance can be beneficial.

2) Waiting for CI sucks. When everything goes smoothly, it's no problem waiting for your continuous integration server to complete the tests and then see the green ticks appear, but when you're out hunting bugs or trying to work out why-oh-why that one test sometimes fails on CI but never on your machine, it can be so annoying to wait every time.

3) A slow test suite could be a sign that the application itself is slow.

## Problem analysis


