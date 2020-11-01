---
title: ' UIKit Cheatsheet Part 7: Loading'
date: '18:29 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - UIKit
theme: magnet-theme
feed:
    limit: 10
hero_classes: 'text-light title-h1h2 overlay-dark-gradient hero-large parallax'
hero_image: unsplash-sidney-perry.jpg
show_sidebar: true
header_id: blog
---

This article is part 7 of my UIKit Cheatsheet series. It deals with loading indicators.

===

Loading indicators are important to show to users that something is happening (e.g. data is fetched from network).
In this blog post I describe 2 methods of doing so.

1. Show loading indicator on a specific view, e.g. a button.

Therefore a UIActivityIndicatorView needs to be connected from storyboard (or created in code) and the loading state needs to be set before a network call and reset after it finished:

```
@IBOutlet var activityIndicator: UIActivityIndicatorView!

activityIndicator.isLoading = true
            imageView.kf.setImage(with: url.imageWith(preset: .content), placeholder: nil, options: nil, progressBlock: nil) { [weak self] result in

                self?.activityIndicator.isLoading = false
```

I'm using the following handy extension to set the loading indicator:

```
extension UIActivityIndicatorView {
    var isLoading: Bool {
        get {
            return isAnimating
        }

        set {
            if newValue {
                isHidden = false
                startAnimating()
            } else {
                stopAnimating()
            }
        }
    }
}
```

2. Displaying loading animation as an entire view:

```
private lazy var loadingView = CustomLoadingView.Builder()
        .with(backgroundColor: themeColors.defaultBackground.fillColor)
        .with(title: L10n.loadingTicketInformation)
        .withLoadingSpinner()
        .build()

...

if viewModel.initialLoading {
      loadingView.translatesAutoresizingMaskIntoConstraints = false
      view.addSubview(loadingView)
      loadingView.bindToSuperView(edgeInsets: .zero)} 
else {
      loadingView.removeFromSuperview()
}
 
```
In my example I'm using a CustomLoadingView. You can actually use any view you like, you just need to pin it to the superview, so that it takes up the entire screen.