---
title: Theming
date: '20:28 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - Theming
theme: magnet-theme
header_image: '0'
feed:
    limit: 10
hero_classes: 'text-dark title-h1h2 overlay-light hero-large parallax'
hero_image: unsplash-luca-bravo.jpg
blog_url: /blog
show_sidebar: false
show_breadcrumbs: true
show_pagination: true
subtitle: 'finding beauty in structure'
header_id: blog
---

An important topic for an iOS app may be to use a consistent theming across the whole UI.
This blog post shows a few options how to do app theming.

===

Set the text color of DescriptiveLabel (subclass of UILabel) in a CustomCell:

`DescriptiveLabel.appearance(whenContainedInInstancesOf: [CustomCell.self]).textColor = textDescriptiveColor`

Setting state (normal, highlighted, disabled,..) coloring of a button:

`UIButton.appearance(whenContainedInInstancesOf: [CustomViewController.self]).setStateColors(themeColor: primary)`

The following UIButton extension is pretty handy for that:
```
extension UIButton {

func setStateColors(colorStates: [ColorState]) {
        colorStates.forEach { state in
            state.titleColor.flatMap {
                setTitleColor($0, for: state.buttonState)
                tintColor = .white
                backgroundColor = .white
            }
            state.backgroundColor.flatMap {
                setBackgroundColor($0, for: state.buttonState)
            }
        }
    }

    func setStateColors(themeColor: ThemeColor) {
        let states: [ColorState] = [
            .normal(titleColor: themeColor.tintColor, backgroundColor: themeColor.fillColor),
            .highlighted(titleColor: themeColor.tintHighlightedColor, backgroundColor: themeColor.fillHighlightedColor),
            .selected(titleColor: themeColor.tintSelectedColor, backgroundColor: themeColor.fillSelectedColor)
        ]
        setStateColors(colorStates: states)
    }
}
```

Customize navigation appearance attributes:

```
       let titleAttributes: [NSAttributedString.Key: Any] = [
            .foregroundColor: navigationBar.textColor
        ]

        let buttonAttributes: [NSAttributedString.Key: Any] = [
            .foregroundColor: navigationBar.tintColor
        ]

        let navBar = UINavigationBar.appearance()
    
        let standardAppearance = UINavigationBarAppearance()
        standardAppearance.configureWithOpaqueBackground()
        standardAppearance.backgroundColor = navigationBar.backgroundColor
        standardAppearance.titleTextAttributes = titleAttributes
        standardAppearance.largeTitleTextAttributes = titleAttributes
        standardAppearance.shadowColor = seperatorColor
        standardAppearance.applyTitleAttributesForAllButtonAppearances(buttonAttributes)
        navBar.standardAppearance = standardAppearance

        let scrollEdgeAppearance = UINavigationBarAppearance()
        scrollEdgeAppearance.configureWithTransparentBackground()
        scrollEdgeAppearance.titleTextAttributes = titleAttributes
        scrollEdgeAppearance.largeTitleTextAttributes = titleAttributes
        scrollEdgeAppearance.applyTitleAttributesForAllButtonAppearances(buttonAttributes)
        navBar.scrollEdgeAppearance = scrollEdgeAppearance

    }
```