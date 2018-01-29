+++
title = "When Django is too bloated - Specialized Web-Applications with Werkzeug"
date = 2018-01-12T22:52:01+01:00  # Schedule page publish date.
draft = false

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
time_start = 2017-07-13T16:20:01+01:00
#time_end = 2018-01-12T22:52:01+01:00

# Abstract and optional shortened version.
abstract = """
Did you ever think, Django and all the other “batteries included” frameworks are not flexible enough for your needs? Do you feel like they limit you in your creativity and design? Then this talk is for you!

Werkzeug is a very lightweight HTTP/WSGI utility for Python. You might have actually used it before, since the popular framework Flask is based on it. Werkzeug handles the WSGI communication with the web server and parsing of HTTP packets for you, after that, you are left to do whatever you want. No pre-defined ORM, no request dispatching or template rendering. As a developer you are supported with a live debugger that runs in the browser and a great variety of testing tools making it easy to write fine grained unit tests for your application.

As a developer at MPS - Medical Systems, I work with Werkzeug on a daily basis. One of our products is ChemoCompile, a chemo therapy planning, management and documentation tool used in hospitals in various European countries. It is a single-page web application written in Python (backend) and AngularJS (frontend). When we created it, we first prototyped it using Django, but soon realized, that we did not need most of the functionality that Django provides and many of our needs, like interfacing with hospital information systems, are too much out of the scope of a regular web applications. I will talk about, how we then discovered Werkzeug and built our own very customized stack on top of it and how you can do it too!
"""
abstract_short = ""

# Name of event and optional event URL.
event = "EuroPython 2017"
event_url = "https://ep2017.europython.eu/en/"

# Location of event.
location = "Rimini, Italy"

# Is this a selected talk? (true/false)
selected = false

# Projects (optional).
#   Associate this talk with one or more of your projects.
#   Simply enter the filename (excluding '.md') of your project file in `content/project/`.
#   E.g. `projects = ["deep-learning"]` references `content/project/deep-learning.md`.
projects = []

# Tags (optional).
#   Set `tags = []` for no tags, or use the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["Python", "Web Development"]

# Links (optional).
url_pdf = ""
url_slides = "https://ep2017.europython.eu/media/conference/slides/when-django-is-too-bloated-specialized-web-applications-with-werkzeug.pdf"
url_video = "https://www.youtube.com/watch?v=mXpBuELtpro"
url_code = ""

# Does the content use math formatting?
math = false

# Does the content use source code highlighting?
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
[header]
image = ""
caption = ""

+++
