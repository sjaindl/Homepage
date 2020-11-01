---
title: ' UIKit Cheatsheet Part 6: Adaptive View Heights'
date: '18:09 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - UIKit
theme: magnet-theme
feed:
    limit: 10
header_id: blog
---

This article is part 6 of my UIKit Cheatsheet series. It deals with adaptive view heights.

===

Often the height of a view - e.g. a label - is not known because it depends on the device dimensions, font size or external data.

![Adaptive height](adaptiveheight.png)

Adaptive heights can be set in storyboard by settings lines to 0, linebreak and autoshrink mode:

![Adaptive height](height_sb.png)