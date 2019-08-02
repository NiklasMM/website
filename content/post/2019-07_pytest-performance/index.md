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

Working on any sort of performance issue, like I was here, is a very specific kind of problem which can become frustrating very quickly, especially if you spring into action to quickly. There are two very important rules:

1) Measure before you do anything. Be absolutely sure that the part of the code you'll be working on is the one that's slowing things down.

2) Optimization anywhere else but at the bottleneck(s) is irrelevant. It may be tempting to shave another couple of milliseconds off that sorting algorithm, but it's not gonna win you anything if that's not the main problem of your application.

If we're transfering this to the slow test suite, we must first find out why the tests are slow and what we can do about it. Of course any improvement will help the overall runtime, but it would surely be most beneficial to find issues affection all or at least many tests.

So here are some things I found helpful when analyzing the problem:

* Pytest comes with the pretty handy option [`--durations`](https://docs.pytest.org/en/latest/usage.html#profiling-test-execution-duration) to list the n slowest tests after a run. This is a pretty low barrier step to get a first idea of which group of tests are causing poblems.

* The [pytest-profiling](https://pypi.org/project/pytest-profiling/) plugin can be used to run the test suite with a profiler. This can give you very good insights but reading profile data can be a bit tricky and may need some getting used to.

* To find out how long the collection phase is, use `--collect-only` and time that.

## Fixing stuff

After profiling and analyzing the problem a bit, I found out, that I had three core problems: The collection phase takes very long. Tests which use a database are somewhat slow (that's about 80 % of all tests) and some tests unneccessarily produce a PDF on the side by making an external call to pdflatex. So let's see how I went about fixing these issues.

### Collection time

The collection phase of the test suite took about 25 seconds. Now that's not too long for a complete run, but keep in mind that pytests needs to collect all tests even if you only want to run a small subset, so this was very annoying during development.

To do anything about that it's important to understand how pytest discovers test: For each test run you give pytest at least one file or directory to find tests in. This can be done via the command line or a pytest.ini config file. Pytest will then import all Python modules from these targets and look for functions and classes defining tests (usually functions with a name starting with `test_` but this behavior can be customized). In our case, although we have all tests placed in a `tests` directory, we had pytest pointed at the root directory of our code base. That way it did discover and run a couple of legacy [doctests](https://docs.pytest.org/en/latest/doctest.html) we had lying around from before the pytest days.
But it means that before each run, pytest had to import (and possibly compile) the entire code base.

Pointing pytest directly at the `tests` directory greatly improved the collection time, but of course the doctests were no longer found. Left with the decision of either rewriting the doc tests as proper tests in the `tests` directory or giving the 4 or 5 files containing the doctests as additional test files to pytest, we opted for the latter one and decided to not add any more doctests in the future.

### Avoiding external calls

Finding out that a lot of PDFs were generated on the side for each test run was... interesting. I wanted to fix this globally without having to go into each single test and mock it out.

@pytest.fixture(scope="function", autouse=True)
def pdftools_popen_mock(request):
    """
        Mocks away the Popen calls in tools.pdf_exports. There are two external processes which
        are called from that module:

        * pandoc: The mock just returns an empty string
        * pdflatex: A copy of the testpage pdf is placed in the requested output dir, so a valid
                    Pdf is available.

        The mock of pdflatex can be disabled by marking the test with
        @pytest.mark.dont_mock_pdflatex
    """

    pdflatex = mock.Mock(returncode=0)
    pandoc = mock.Mock(communicate=mock.Mock(return_value=[b""]))

    def filtered_Popen(*args, **kwargs):
        if args[0][0] == "pdflatex" and "dont_mock_pdflatex" not in request.keywords:
            output_dir = args[0][2]
            shutil.copy(TESTPAGE_PATH, os.path.join(output_dir, "export.pdf"))
            return pdflatex
        elif args[0][0] == "pandoc":
            return pandoc
        return Popen(*args, **kwargs)

    with mock.patch("chemocompile.tools.pdf_export.Popen", side_effect=filtered_Popen) as m:
        yield m

So I took to [Twitter](https://twitter.com/NiklasMM/status/1143142052940718080) to see if anyone had tips on this and I got many resonses helping me to get closer to my goal.


