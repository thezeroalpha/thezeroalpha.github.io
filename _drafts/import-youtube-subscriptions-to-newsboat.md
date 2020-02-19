---
layout: post
title: "Import Youtube subscriptions to newsboat"
author: zeroalpha
categories: Guide
---

Go to youtube -> subscriptions -> manage subscriptions. Scroll all the way down so everything loads. Then open web inspector. In "elements" tab, right click on the root "html" element and click copy -> copy outerHTML.
Save this somewhere as "source.html". Go to that directory in the terminal, run the ruby script. Channels and URLs will be stored in newsboat format, with URL and name, as well as 'yt-video' tag.
