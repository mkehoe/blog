---
date: "2020-09-11"
title: "Qt Problems"
slug: "qt-problems"
tags: [
    "c++",
    "qt"
]
categories: [
    "qt"
]
disable_comments: true
---

Recently I have been working a bunch with Qt and while every framework has is pro's and con's, this week's dealings with it have been particularly frustrating. Qt's strength is that it offers more features than most C++ GUI frameworks, but sometimes it seems that details are overlooked with their implementations. After wasting a ton of time on one of their minor "quirks" this week, just needed vent on some of their shortcomings.

Qt has two ways of constructing a UI for an application: Qt Widgets and Qt Quick (Qml). I inherited a legacy project that is based on the widgets framework which appears to be the older, original framework. These days, Qt seems to give all the love to the qml framework. I don't want to make it seem like the widgets aren't maintained, but qml has more to offer. For example, material theme is only available for qml. A [third party option](https://github.com/laserpants/qt-material-widgets) exists but it has been idle for 2 years.

Another issue I have is that the default build is missing a key component for the web kit so I need to cut a custom build. With out the proprietary codecs switch on the build, the web kit is not able to handle standard H264 videos. As of now H264 is the most common option for web video, though it is very patent hindered so I am guessing their reason for leaving it out is legal.

Building a custom version isn't the worst thing, but I ended up wasting several hours on one oddity. The build procedure is fairly well [documented](https://doc.qt.io/qt-5/build-sources.html) and I chose to build it from [git](https://wiki.qt.io/Building_Qt_5_from_Git). Qt's code is all stored on code.qt.io however they also have repositories on github that mostly mirror the qt.io site, and I say mostly because building from github is broken. This issue is related to this [infrastructure bug]{(https://bugreports.qt.io/browse/QTQAINFRA-3384) though github was never addressed in the fix. I suppose the real question is why bother maintaining two separate repositories?

I do think Qt has a lot of positives, but the negatives such as the above mentioned quirks and the recent push away from open source would definitely cause me to look at other options in the future.