+++
title = "Cloudless Fitness Tracking"
date = 2018-08-24T20:06:21+02:00
draft = false
authors = ["niklas"]
# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["privacy", "review", "sport", "open source", "Android"]
categories = ["Technology"]

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
# Use `caption` to display an image caption.
#   Markdown linking is allowed, e.g. `caption = "[Image credit](http://example.org)"`.
# Set `preview` to `false` to disable the thumbnail in listings.
[header]
image = ""
caption = ""
preview = true

[[gallery_item]]
album = "1"
image = "Gadgetbridge/home.png"

[[gallery_item]]
album = "1"
image = "Gadgetbridge/steps.png"

[[gallery_item]]
album = "1"
image = "Gadgetbridge/alarms.png"

[[gallery_item]]
album = "1"
image = "Gadgetbridge/graph.png"

+++

A couple of years ago I bought a Fitbit Flex, the second generation (?) fitness tracker by Fitbit. It was at a time when I had just graduated from university and started my job, which involved a lot of sitting. (I wasn't paid for the sitting part) I was interested in how close to the 10000 daily steps I got and the Fitbit provided this information. It had a very limited physical display consisting only of 5 LEDs which would consecutively light up as you approached the 10000 steps. For everything else you had to use the Android app. I stopped using it after about a year.

Fast forward to 2018 (or was it the end of 2017?): Snowden had happened (5 years ago, believe it or not) and because of it and other things, I had become more interested in how much of my data ended up on other peoples (and companies) computers. But I was again interested in fitness trackers and researched if there were any devices and/or alternative apps which would not send my data over the internet. After all, I only want to know how many steps a day I take. There's no reason this information needs to leave the device.

Eventually, I discovered [Gadgetbridge](https://gadgetbridge.org/) an open source project aiming to bring support for as many fitness tracker type devices to Android users without the use of the specific vendor's apps. The list of supported devices includes none of the Fitbits, however. As I found out by watching [this](https://media.ccc.de/v/34c3-8908-doping_your_fitbit) great talk from last years Chaos Communication Congress, this is because Fitbit locks out third party apps by design. All data is encrypted on the device. When you want to read it on your phone, the encrypted data packet is first transfered from the Fitbit to your phone via Bluetooth, then sent to Fitbit's servers, where it is decrypted and sent back to the Phone. The server, of course, will only talk to the official app. Naturally, this is only for security reasons or whatever ;) But anyway, Fitbits won't respect your privacy.


But I now had the list of supported devices and eventually chose one of the simpler models, the [Xiaomi Mi Band 2](https://en.wikipedia.org/wiki/Xiaomi_Mi_Band_2). This device is fully supported by Gadgedbridge and only costs about 25 € since it's not the most recent model.

{{% figure src="/img/miband2.jpeg" caption="Xiaomi Mi Band 2"%}}

You can activate the device without even installing the Xiaomi app and therefore never send out any data at all.

The Mi Band is pretty basic, besides the step counter it also shows the time and can measure your heart rate (on demand or periodically) and sleep cycles. It can also display notifications from your phone, but I never used that feature.

Here are some screenshots from the app:

{{< gallery album="1">}}


So yeah, it does the trick. Counts my steps, shows the time and occasionally takes my heart rate. And all that data never leaves my devices. By the way: The battery lasts for about a month.
