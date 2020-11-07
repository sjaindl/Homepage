---
title: 'Feature Flag Architecture'
date: '20:38 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - 'design patterns'
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

I have once implemented a complete feature flag architecture for an iOS app. Such an architecture is important to switch features on or off, even remotely so that no new build is required. This blog post describes how the architecture was implemented.

===

First a general Feature protocol and an implementation of it is provided:

```
protocol Feature {
    var key: String { get }
}


struct FeatureFlag: Feature {
    let key: String
    
    init(key: FeatureKey) {
        self.key = key.rawValue
    }
}
```

Then, we define an enum FeatureKey, which represent the unique features within the app:

```
enum FeatureKey: String, CaseIterable {
    case firstFeature
    case superFeature
    case uniqueFeature
...
}
```

The FeatureFlagProviderType enum represents the different sources where feature flags come from. 
In this case it is debug, release, remote (for switching  features on/off remotely) and test.

```
enum FeatureFlagProviderType {
    case debug, release, remote, test
}
```

Next, the FeaturePriority struct defines feature orders, when more than one FeatureFlagProviderType is applicable to a feature:

```
struct FeaturePriority {
    static let minPriority = 1
    static let mediumPriority = 5
    static let maxPriority = 10
    static let testPriority = 15
}
```

Then the protocol FeatureFlagProvider requires that priority as well as the type. Implementers must provide methods that determine whether it defines that feature, and if so, whether it is enabled:

```
protocol FeatureFlagProvider: class {
    var priority: Int { get }
    var type: FeatureFlagProviderType { get }
    
    func isFeatureEnabled(_ featureKey: FeatureKey) -> Bool
    func hasFeature(_ featureKey: FeatureKey) -> Bool
}
```

The feature flag providers should be instantiated via a factory:

```
final class FeatureFlagProviderFactory {
    static func createProvider(type: FeatureFlagProviderType, with featurePairs: [FeaturePair] = [], userDefaults: UserDefaults? = nil, remoteSync: Remote? = nil) -> FeatureFlagProvider {
        
        switch type {
            case .debug: return DebugFeatureFlagProvider(userDefaults: userDefaults ?? .standard)
            case .release: return ReleaseFeatureFlagProvider()
            case .remote: return ContentfulFeatureFlagProvider(featurePairs: featurePairs, remoteSync: remoteSync)
            case .test: return TestFeatureFlagProvider()
        }
    }
}
```

We will look in detail at the debug and release providers. The ReleaseFeatureFlagProvider has min priority, so that it can be overruled by remote flags. In this implementation it provides valus for all features, whether they are generally enabled or not:

```
//Used in release version
final class ReleaseFeatureFlagProvider: FeatureFlagProvider {
    
    internal let priority = FeaturePriority.minPriority
    internal let type = FeatureFlagProviderType.release
    
    func isFeatureEnabled(_ featureKey: FeatureKey) -> Bool {
        switch featureKey {
         case firstFeature:
         	return true
         case superFeature:
         	return false
         case uniqueFeature
         	return true
        }
    }
    
    func hasFeature(_ featureKey: FeatureKey) -> Bool {
        switch featureKey {
        case firstFeature:
         	return true
         case superFeature:
         	return true
         case uniqueFeature
         	return true
        }
    }
}
```

The DebugFeatureFlagProvider should only be added for debug/dev builds. Features can be switched on or off by UserDefaults settings. It may be handy to provide a UI for doing so:

```
final class DebugFeatureFlagProvider: MutatingFeatureFlagProvider {
    
    private let newFeaturesInitiallyEnabled = true
    private let userDefaults: UserDefaults
    internal let priority = FeaturePriority.mediumPriority
    internal let type = FeatureFlagProviderType.debug
    
    init(userDefaults: UserDefaults) {
        self.userDefaults = userDefaults
    }
    
    func isFeatureEnabled(_ featureKey: FeatureKey) -> Bool {
        switch featureKey {
        case .routing: return userDefaults.bool(forKey: featureKey.rawValue, defaultValue: false)
        default: return userDefaults.bool(forKey: featureKey.rawValue, defaultValue: newFeaturesInitiallyEnabled)
        }
    }
    
    func hasFeature(_ featureKey: FeatureKey) -> Bool {
        return true
    }
    
    func setFeatureEnabled(feature: Feature, enabled: Bool) {
        userDefaults.set(enabled, forKey: feature.key)
    }
}
```