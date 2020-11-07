---
title: ' UIKit Cheatsheet Part 4: UIView Roundings'
date: '17:30 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - UIKit
theme: magnet-theme
feed:
    limit: 10
hero_classes: 'text-light title-h1h2 overlay-dark hero-large parallax'
hero_image: unsplash-focus.jpg
show_sidebar: true
header_id: blog
badge_class: badge-color-blog
---

This article is part 4 of my  UIKit Cheatsheet series. It deals with roundings for all kinds of UIViews.

===

Roundings on buttons, labels etc. are looking quite good. That's why we want to have it for our views in iOS. 
There are generally 2 preferrable ways of doing these roundings:

1. CALayer

```
extension CALayer {
    func roundCorners(_ radius: CGFloat, cornerMask: CACornerMask? = nil) {
        cornerRadius = radius
        if let cornerMask = cornerMask {
            maskedCorners = cornerMask
        }
        
        masksToBounds = true
    }
}
```

Using CALayer existing properties is an easy (and likely the most efficient) solution for simple corner masking. It's animatable, too, and the masked corners can be set (e.g. round all corners, only top corners etc.).

2. CALayerMask

```
public func round(corners: UIRectCorner, radius: CGFloat) {
        let maskPath = UIBezierPath(roundedRect: bounds, byRoundingCorners: corners, cornerRadii: CGSize(width: radius, height: radius))
        let maskLayer = CAShapeLayer()
        maskLayer.frame = bounds
        maskLayer.path = maskPath.cgPath
        layer.mask = maskLayer
    }
```

CAShapeLayer masks are nice approach if the corner masking isn't simple corner rounding but some arbitrary path. You have to be cautious to make sure to update this mask if the frame changes (e.g. in layoutSubviews of view or in viewDidLayoutSubviews of controller). Animating requires more work.

I'd suggest option 1 for simple cases, and option 2 if you need more control than option 1 can offer.