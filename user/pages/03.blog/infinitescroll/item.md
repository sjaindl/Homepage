---
title: 'Overlay with Infinite Scrolling Behaviour'
media_order: uinavigation.png
date: '18:49 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - UIKit
theme: magnet-theme
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

I have worked once on a ViewController that should support drag & drop of a child ViewController with a certain amount of fixed sheet states.
I want to share how such a feature can be implemented.

===

<video controls  height="500" src="infinitescroll/scroll.mov" style="display: block; margin: auto;">
Your browser does not support the video tag.
</video>

There are actually three ViewControllers involved:
* The outer ContainerViewController
* The inner ChildViewController
* Another ViewController that is embedded in the ChildViewController - let's call it EmbeddedViewController

Let's deal with the ContainerViewController first.
The ContainerViewController deals with the main logic. It adds and talks with the Child and Embedded ViewControllers with the help of pan gesture recognizers and delegates to get informed about the scrolling position of the content and adjusts the overlay accordingly. Adjustments happens on a top constraint from the top safe area to the ChildViewController.
It also handles bouncing and keyboard show/hide events.

Here is the code for the ContainerViewController:

```
final class ContainerViewController: UIViewController {

    private var gesture: UIPanGestureRecognizer?
    private var currentState: SheetState = .sheetDefault
    private var currentTranslationOffset: CGFloat = 0.0
    private var keyboardHeight: CGFloat = 0.0
    private var initialOverlayAdjustmentDone = false

override func viewDidLoad() {
        super.viewDidLoad()

        setupUI()
        addBottomSheet()
        setupGesture()
        addObservers()
    }

func setupGesture() {
        gesture = UIPanGestureRecognizer(target: self, action: #selector(panGesture))

        if let gesture = gesture {
            stackView.addGestureRecognizer(gesture)
        }
    }


@objc func panGesture(recognizer: UIPanGestureRecognizer) {
        let dragVerticalLocation = recognizer.location(in: panelDragContainer).y

        if currentTranslationOffset == 0.0 {
            currentTranslationOffset = dragVerticalLocation
        }

        let adjustmentPosition = currentTranslationOffset - dragVerticalLocation
        handleInfiniteScroll(to: panelTopConstraint.constant - adjustmentPosition, animated: false, velocity: recognizer.velocity(in: panelDragContainer).y)

        if recognizer.state == .ended || recognizer.state == .cancelled {
            let state = viewModel.nearestState(for: panelTopConstraint.constant, currentState: currentState)
            currentState = state
            adjustOverlay(to: verticalPosition(for: state), animated: true)
            currentTranslationOffset = 0.0
            containerOverlayDelegate?.didDragOverlay()
        }
    }


private func addBottomSheet() {
       
        let embeddedPage = EmbeddedPageViewController.create( .. 
            viewConfig: ViewConfig(customModalPresentationStyle: .overCurrentContext, ..) .. )

        let childViewController =ChildViewController.create(rootViewController: embeddedPage, ..)
        childViewController.containerDelegate = self
        containerOverlayDelegate = childViewController
        embeddedPage.containerNavigationDelegate = childViewController
        
        let navigationController = childViewController.embedInNavigationController().withModalPresentationStyle(.fullScreen)

        embedd(controller: navigationController)
    }
}

extension ContainerViewController {

    private func handleInfiniteScroll(to y: CGFloat?, animated: Bool, velocity: CGFloat = 0.0) {
        let scrollDirection = SheetScrollDirection.directionForVelocity(velocity: velocity)

        //If we did not scroll at all (scrollDirection == .unchanged), we can stop immediately
        guard let y = y, scrollDirection != .unchanged, let containerOverlayDelegate = containerOverlayDelegate, let gesture = gesture else {
            return
        }

        //.. Otherwise we have to handle the overlay, and the content in the EmbeddedViewController

        if overlayIsAtTop() {
            //if the overlay is at its maximum sheet state position..

            //Drag content: This handles movement of embedded content
            let contentIsAtTop = containerOverlayDelegate.dragContent(velocity: velocity * SheetStateConfig.panVelocityToTranslationFactor)

            if contentIsAtTop {
                //If overlay is at top (= embedded content is at the very top of the overlay): Adjust overlay itself
                adjustOverlay(to: y - keyboardHeight, animated: animated, scrollDirection: scrollDirection)
            } else {
                //Else content was scrolled/adjusted, but is not on top yet. Just remember currentTranslationOffset (we need that later for proper adjustment of the overlay)
                currentTranslationOffset = gesture.location(in: panelDragContainer).y
            }
        } else {
            //if the overlay is not at its maximum sheet state position, we just adjust the overlay
            adjustOverlay(to: y - keyboardHeight, animated: animated, scrollDirection: scrollDirection)
        }
    }

    private func adjustOverlay(to yPosition: CGFloat, animated: Bool, scrollDirection: SheetScrollDirection = .unchanged) {
        let constant = adjustOverlayPosition(for: yPosition)

        panelTopConstraint.constant = constant

        if animated {
            UIView.animate(withDuration: 0.1,
                delay: 0,
                options: .curveLinear,
                animations: {
                    [weak self] in
                        self?.view.layoutIfNeeded()
                })
        } 
    }

    private func adjustOverlayPosition(for yPosition: CGFloat) -> CGFloat {
        let minAllowedPos = verticalPosition(for: .sheetDefault)
        let maxAllowedPos = verticalPosition(for: .maximum)
        let adjustedOverlayPosition = max(yPosition, maxAllowedPos)

        return adjustedOverlayPosition > minAllowedPos ? minAllowedPos : adjustedOverlayPosition
    }

    private func overlayIsAtTop() -> Bool {
        return panelTopConstraint.constant <= verticalPosition(for: .maximum)
    }

    private func verticalPosition(for state: SheetState) -> CGFloat {
        return viewModel.verticalPosition(for: state)
    }

    func addObservers() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(keyboardDidShow),
            name: UIResponder.keyboardDidShowNotification,
            object: nil
        )

        NotificationCenter.default.addObserver(
            self,
            selector: #selector(keyboardWillHide),
            name: UIResponder.keyboardWillHideNotification,
            object: nil
        )
    }

    @objc func keyboardDidShow(_ notification: Notification) {
        if let keyboardFrame: NSValue = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue {
            let keyboardRectangle = keyboardFrame.cgRectValue
            keyboardHeight = keyboardRectangle.height
        }
    }

    @objc func keyboardWillHide(_ notification: Notification) {
        keyboardHeight = 0
    }
}	
```

Some of the logic can be factored out into a ContainerViewModel. The ContainerViewModel declares the available SheetStates in an enum and initializes their vertical positions in a map. It contains also the logic to search for the nearest state the overlay should be adjusted to, when a pan gesture ended:

```
final class ContainerViewModel {

    enum SheetState: String, CaseIterable {
        case minimized
        case teaser
        case default
        case expanded
        case maximum
    }

    private var statePosition: Array<CGFloat> = Array<CGFloat>(repeating: 0, count: SheetState.allCases.count)
    private var innerItemOffset: CGFloat = 280.0

    init(viewHeight: CGFloat) {
        setupSheetState(viewHeight: viewHeight)
    }

    func setupSheetState(viewHeight: CGFloat) {
        let minHeight = viewHeight - innerItemOffset

        statePosition[SheetState.sheetDefault.rawValue] = minHeight
        statePosition[SheetState.teaser.rawValue] = (minHeight + SheetStateConfig.maximumStatePosition) / 2
        statePosition[SheetState.maximum.rawValue] = SheetStateConfig.maximumStatePosition
    }

    func verticalPosition(for state: SheetState) -> CGFloat {
        return statePosition[state.rawValue]
    }

    func nearestState(for verticalTranslation: CGFloat, currentState: SheetState) -> SheetState {
        var smallestDiff: CGFloat = CGFloat.greatestFiniteMagnitude
        var nextState: SheetState? = currentState

        for (state, position) in statePosition.enumerated() {
            let currentDiff = abs(verticalTranslation - position)
            if currentDiff < smallestDiff {
                smallestDiff = currentDiff
                nextState = SheetState(rawValue: state)
            }
        }

        return nextState ?? SheetState.sheetDefault
    }
}
```

The ChildViewController is responsible for scrolling/setting constraints of the inner content, menu & filters, and calling the delegate methods the ContainerViewController is using.

```
final class ChildViewController: UIViewController {

  func setContainerOverlayDelegate() {
        if let containerViewController = navigationController?.parent as? ContainerViewController {
            containerViewController.containerOverlayDelegate = self
        }
    }


extension ChildViewController: ContainerOverlayDelegate {
    func dragContent(velocity: CGFloat) -> Bool {
        guard let rootViewController = rootViewController as? EmbeddedViewController else {
            return true
        }
        
        //First, we always adjust the constraints of filter/menu, if necessary:
        var adjusted = adjustFilterAndNavigation(distanceFromTop: velocity)
        
        //If the content fits vertically on the screen, we just need to adjust filter/menu
        if rootViewController.fitsVerticallyOnScreen() {
            return !adjusted
        }
        
        //If there was no need to adjust the filter/menu constraints, we scroll the content:
        if !adjusted {
            rootViewController.scrollContent(with: velocity)
            
            //Also in the case after scrolling the content, we may need to adjust filter/menu constraints
            if rootViewController.isAtTop() {
                adjusted = adjustFilterAndNavigation(distanceFromTop: velocity)
            }
        }
        
        //If either menu/filter or content has changed, we scrolled something inside there
        return !adjusted && rootViewController.isAtTop()
    }
    
    func didDragMapOverlay() {
        (rootViewController as? EmbeddedViewController)?.bounce()
    }

private func adjustFilterAndNavigation(distanceFromTop: CGFloat) -> Bool {
        let constant: CGFloat = filter.isEmpty ? Self.menuTopHeight : Self.filterTopHeight + (searchBar?.frame.size.height ?? 0)
        let scrollDirection = SheetScrollDirection.directionForVelocity(velocity: distanceFromTop)
            
        if scrollDirection == .down {
            guard searchBarTopConstraint.constant > -constant else {
                return false
            }
            
            searchBarTopConstraint.constant = max(-constant, searchBarTopConstraint.constant + distanceFromTop)
            return true
        } else {
            guard searchBarTopConstraint.constant < 0, distanceFromTop < constant else {
                return false
            }
            
            searchBarTopConstraint.constant = min(0, searchBarTopConstraint.constant + distanceFromTop)
            return true
        }
    }

}
```

Finally, the EmbeddedViewController reacts to scroll events from the ChildViewController:

```
extension EmbeddedViewController {
    func scrollContent(with velocity: CGFloat) {
        guard scrollView != nil else {
            return
        }
        
        scrollView.contentOffset.y -= velocity
    }
    
    func bounce() {
        scrollView.bounceToMaxContentOffset()
    }
    
    func isAtTop() -> Bool {
        return scrollView.contentOffset.y <= scrollView.minContentOffset.y
    }
    
    func fitsVerticallyOnScreen() -> Bool {
        return scrollView.fitsVerticallyOnScreen()
    }
}
```

To encapsulate the scroll view specific actions, there is an extension to UIScrollView:

```
extension UIScrollView {
    var minContentOffset: CGPoint {
        return CGPoint(x: -contentInset.left, y: -contentInset.top)
    }

    var maxContentOffset: CGPoint {
        return CGPoint(x: contentSize.width - bounds.width + contentInset.right, y: contentSize.height - bounds.height + contentInset.bottom + 16)
    }
    
    func bounceToMaxContentOffset() {
        if contentOffset.y > maxContentOffset.y + 16 {
            let rect = CGRect(x: contentOffset.x, y: maxContentOffset.y + 16, width: 1, height: 1)
            scrollRectToVisible(rect, animated: true)
        }
    }
    
    func fitsVerticallyOnScreen() -> Bool {
        return bounds.height + contentInset.bottom >= contentSize.height
    }
}
```
