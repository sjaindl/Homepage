---
title: 'Google Maps Overlay & Two-Finger-Scrolling'
media_order: uinavigation.png
date: '21:27 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - Android
        - 'Google Maps'
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

Google Maps is a very much-used library on Android. This blog post demonstrates how a Two-Finger-Scroll can be implemented.

===

Two-Finger scrolling and hint to use 2 fingers:

<video controls  height="300" src="googlemapsscroll/scroll.mov" style="display: block; margin: auto;">
Your browser does not support the video tag.
</video>

First we define an interface to show hints:

```
interface MapScrollInterceptor {
    fun showHint()
}
```

The core logic lies in the custom MapView class that is designed to react on double finger movements only. Movements with one finger only are intercepted with requestDisallowInterceptTouchEvent. The pointerCount indicates how many fingers are tapping the MapView:

```
class DoubleTouchableMapView : MapView {
    constructor(context: Context) : super(context)
    constructor(context: Context, options: GoogleMapOptions) : super(context, options)

    var interceptorCallback: MapScrollInterceptor? = null

    override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {
        val touchEvent = ev ?: return false

        parent.requestDisallowInterceptTouchEvent(touchEvent.pointerCount > 1)
        return super.dispatchTouchEvent(ev)
    }

    override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
        val touchEvent = ev ?: return false

        if (touchEvent.action == MotionEvent.ACTION_MOVE) {
            if (touchEvent.pointerCount < 2) {
                //Intercept move with one finger only
                interceptorCallback?.showHint()
            }

            return touchEvent.pointerCount < 2
        }

        return super.onInterceptTouchEvent(ev)
    }
}
```

The GoogleMapsFragment implements the MapScrollInterceptor interface and contains the MapView. It shows (and hides again) the hint using animations:

```
class GoogleMapsFragment : Fragment(), MapScrollInterceptor {

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    setupMap()
    mapView.onCreate(savedInstanceState)
    mapView.interceptorCallback = this
}

override fun showHint() {
    if (isAnimating) {
        return
    }

    isAnimating = true

    map_scroll_hint.animate()
            .setStartDelay(0L)
            .setDuration(250L)
            .alpha(0.8f)
            .setListener(object : AnimatorListenerAdapter() {
                override fun onAnimationEnd(animation: Animator?) {
                    super.onAnimationEnd(animation)
                    hideHint()
                }
            })
            .start()
}

private fun hideHint() {
    map_scroll_hint.animate()
            .setStartDelay(2000L)
            .setDuration(250L)
            .alpha(0.0f)
            .setListener(object : AnimatorListenerAdapter() {
                override fun onAnimationEnd(animation: Animator?) {
                    super.onAnimationEnd(animation)

                    isAnimating = false
                }
            })
            .start()
}
```

The layout of GoogleMapsFragment looks the following:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <FrameLayout
            android:id="@+id/map_view_container"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <TextView
            android:id="@+id/map_scroll_hint"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:alpha="0.0"
            android:background="@android:color/darker_gray"
            android:gravity="center"
            android:text="@string/use_two_fingers_to_scroll_the_map"
            android:textColor="@android:color/white"
            android:textSize="@dimen/textSizeButton"
            app:layout_constraintBottom_toBottomOf="@+id/map_view_container" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```