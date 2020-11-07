---
title: 'App Deep Linking'
date: '20:38 10/31/2020'
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
badge_class: badge-color-blog
---

This blog post illustrates how deep linking into an iOS app works.

===

DeepLinking means that you can enter e.g. the following link in Safari or any other browser on iPhone or iPad:

`app://mycustomcontroller/1234?showIt=true`

This should open a defined part of the app. Therefore first the following function in AppDelegate must be implemented:

```
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
        logVerbose("application:openUrl:", tag: "openUrl")
        return env.deepLinkHandler.handleUrl(url, animated: false)
    }
```

and that deeplink must be handled:

```
public func handleUrl(_ url: URL, animated: Bool) -> Bool {
  let deepLink = DeepLink(url: url)
}

public struct DeepLink {

public init?(string: String) {
        guard let url = URL(string: string) else { return nil }
		
     	guard let components = URLComponents(url: url, resolvingAgainstBaseURL: true) else { return nil }        
…
        if components.hasFeature("mycustomcontroller") {

guard let queryItems = components.queryItems else {
            return nil
        }


queryItems.forEach { item in

…
}
            route = .mycustomcontroller(url.lastPathComponent)
            return
        }
…
    }
}

...

```