---
layout: post
title: "Navigation Drawer and Composable Scaffold"
post-excerpt: "FILL IN BEFORE POSTING"
social-image: "Fill in before posting"
header: "fill in before posting"
---

Navigation is an important feature in any Android app. One of the common ways to display navigation is through the navigation drawer. This post adds the navigation drawer to the Tweetish app through the use of the `Scaffold` component.

## Other options

If you look through the compose documentation you will likely find the `ModalDrawerLayout` which implements the typical navigation drawer. This is the first component I tried using to add the drawer but it did not work as I expected.

Adding the component to the app involved specifying the state of the drawer, passing a state change listener, setting the drawer content, and setting the body content. I initially set the drawer content to be empty just to verify everything worked. Running the app and opening the drawer showed the first issue.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_10/drawer_under_app_bar.png" alt="Emulator screenshot showing the navigation drawer drawn under the app bar"/>
</div>

The navigation drawer is drawn below the app bar. This is contrary to the [Material Design specifications for the modal drawer](https://material.io/components/navigation-drawer/#modal-drawer) that recommends it be drawn on top of the app bar.

Testing out the gestures also resulted in another unexpected issue.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_10/body_content_still_clickable.png" alt="Emulator screenshot showing the screen content is still clickable under the navigation drawer."/>
</div>

I was able to click on the share buttons in the main screen content behind the drawer. This is another issue since the content behind the drawer should not react to any click events when the drawer is open.

Still, I persisted adding the content to the drawer. Running the app gave me the third strike.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_10/drawer_content_under_app_bar.png" alt="Emulator screenshot showing the navigation drawer content being drawn under the app bar."/>
</div>

The navigation drawer is not just drawn below the app bar, it is actually drawn behind it. This causes the navigation content to be drawn below the app bar which is not very useful. I looked into adding elevation to the `ModalDrawerLayout` but it does not accept a modifier parameter to change the elevation.

These issues required a change in strategy.

## Better option

I turned to the Compose channel on the [Kotlin slack workspace](http://slack.kotlinlang.org/). Zach Klipp recommended a better component that should suit my needs.

> You probably want to use Scaffold.
> It will make sure your app bars, drawers, FABs, and content are laid out correctly
> Not sure about the touch-through issue though.

Checking out the `Scaffold` component looked promising. The function accepts all the parameters I would need for my app bar, drawer content, body content, and it even has parameters for the floating action button covered in my next post.

<script src="https://gist.github.com/BrianGardnerAtl/80da9438d06ef4f2ff12627783b6d857.js"></script>

Creating the `Scaffold` is straightforward since all of my components are broken out into small composable functions.

<script src="https://gist.github.com/BrianGardnerAtl/98cc1a81c7e538fe6e5d21faac28b4e9.js"></script>

The `TweetBar` is passed to the `topAppBar` parameter and the `TweetList` is passed to the `bodyContent`. The `drawerContent` is left blank for now to verify everything works as expected. Running the app verifies that the drawer is now drawn above the app bar.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_10/drawer_above_app_bar.png" alt="Emulator screenshot showing the drawer drawn above the app bar when using the Scaffold component."/>
</div>

The next step is filling out the drawer content. Adding a new composable function is cleaner than implementing it within the `drawerContent` parameter.

<script src="https://gist.github.com/BrianGardnerAtl/85fb91f5434b5e52a89680147195d2b4.js"></script>

For now it is just a column with various text items to match the Twitter app. I'm not going to focus on styling the components since it works the same as any other composable items.

Adding the `TweetNavigation` to the `Scaffold` involved putting it in the `drawerContent` lambda.

<script src="https://gist.github.com/BrianGardnerAtl/f037d0cf86216a1094796b8bc60a60b6.js"></script>

Running the app confirms that the drawer content is filled out.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_10/drawer_contents_expanded.png" alt="Emulator screenshot showing the drawer contents expanding to fill the size of the drawer."/>
</div>

The next step is to hook up the navigation icon button. The drawer should open when it is selected. This first involved setting up a `ScaffoldState`. This object specifies the current drawer state, either open or closed, as well as if the drawer state should change using gestures.

Wrapping the `ScaffoldState` creation in a `state {}` block provides access to the state itself as well as a change function that can be invoked to set the state. Once the state property is set it can be passed to the `scaffoldState` parameter on the `Scaffold`.

<script src="https://gist.github.com/BrianGardnerAtl/a0b40791f36e6c320c76789a773a9673.js"></script>

The `onScaffoldStateChange` function can then be used to change the state of the scaffold when the navigation icon is clicked.

First, the `TweetBar` needs to take in a click listener as a parameter to handle the navigation icon click.

<script src="https://gist.github.com/BrianGardnerAtl/c7b9c8f967f7e1b94aec091f1731a582.js"></script>

Then, the `ListScreen` can pass in the listener that modifies the `scaffoldState` and calls the `onScaffoldStateChange` listener to the `TweetBar` component.

<script src="https://gist.github.com/BrianGardnerAtl/e29ef93dce14b7f0a9b123d6a6b9f13c.js"></script>

One last cleanup task involved updating the `TweetBarPreview` so the project will build. An empty lambda is enough to satisfy the compiler.

<script src="https://gist.github.com/BrianGardnerAtl/7a1eee01eb13d7ad310f5fc3cb449e09.js"></script>

The listener queries the current state of the drawer and sets it to the opposite. Running the app shows that clicking on the navigation icon opens the drawer.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_10/drawer_opens_on_nav_icon_click.webm" type="video/webm">
        Emulator screen recording of the navigation drawer opening when the navigation icon is clicked.
    </video>
</div>

With that, the navigation drawer component is complete. A future post will examine how to perform the navigation when a navigation item is clicked. This will likely tie-in the navigation architecture component for even more Jetpack goodies.

Thanks for reading and stay tuned for more!
