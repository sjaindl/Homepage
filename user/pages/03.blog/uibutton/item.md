---
title: ' UIKit Cheatsheet Part 3: UIButton'
date: '17:29 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - UIKit
theme: magnet-theme
feed:
    limit: 10
continue_link: true
link: 'http://daringfireball.net'
header_id: blog
badge_class: badge-color-blog
---

This article is part 3 of my UIKit Cheatsheet series. It deals with tricks for UIButton.

===

One tricky issue with UIButton that consists of text as well as an image, when using highlighting animation on button press, is that the image gets greyed too much, but shouldn't be. Animations for images embedded in buttons can be disabled as follows:

```
myButton.adjustsImageWhenHighlighted = false
myButton.adjustsImageWhenDisabled = false
```
