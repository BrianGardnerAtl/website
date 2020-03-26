---
layout: post
title: "Composable App Bar"
post-excerpt: "FILL IN BEFORE POSTING"
social-image: "Fill in before posting"
header: "fill in before posting"
---

This post focuses on styling my copy-cat Tweet app with an App Bar. I will take a look at the component used to add thea pp bar as well as its various parameter options.

## Create an app bar

Compose has a couple components available for top and bottom navigation bars. Implementing an app bar is the job of the `TopAppBar` component. This add the bar at the top of the screen and it has parameters for a title, navigation icon, and action icons.

The first step is to add a new `@Composable` function representing the app bar for this screen. It will configure a `TopAppBar` with just a title for now. The `title` parameter accepts a composable lambda where I can provide a `Text` component to display.

<script src="https://gist.github.com/BrianGardnerAtl/8b881988d87907e26b4f5d9025e70894.js"></script>

Adding a preview function allows me to see how things look.

<script src="https://gist.github.com/BrianGardnerAtl/2170070cb91d4cf6e7e54807ffca1833.js"></script>

<img class="post-image" src="/assets/images/compose_9/initial_app_bar_preview.png" alt="Preview pane showing the initial top app bar with title"/>

The app bar is automatically tinted with the theme's primary color, but it can be changed using the `color` property. A future post will modify the theme of the app so I will leave it as-is at the moment.

Next, I want to display my new app bar on the screen. This requires a new composable function where the `TweetList` and `TweetAppBar` are combined. Since this new component will represent the entire screen, I call it `ListScreen`.

<script src="https://gist.github.com/BrianGardnerAtl/6b9c3abe0878dbd85b35b14f3f2337fd.js"></script>

This component accepts the list `MutableState` so it can forward it to the `TweetList`. This component is then used inside the `onCreate` function to display it when the app runs.

<script src="https://gist.github.com/BrianGardnerAtl/204d09f017a6e422bc5d2b5f78c000de.js"></script>

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_9/double_app_bar.png" alt="Emulator screenshot showing two app bars displayed."/>
</div>

## Double trouble

Running the app shows an interesting behavior. There are two app bars displayed. It turns out that my `MainActivity` is using the default app theme which automatically applies a dark app bar. Updating the `AppTheme` removes this duplicate app bar.

<script src="https://gist.github.com/BrianGardnerAtl/376da33128d6a978669ed787fa575450.js"></script>

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_9/one_app_bar.png" alt="Emulator screenshot showing only one app bar."/>
</div>

## Navigation icon

App bars are often used for navigation purposes. The most common is displaying a hamburger icon indicating there is a navigation drawer off screen.

Adding this requires a drawable resource. Using the [Vector Asset Studio](https://developer.android.com/studio/write/vector-asset-studio), I add the `dehaze` asset and name it `ic_nav_drawer`.

To add the icon to the `TopAppBar`, the `navigationIcon` parameter is used. This accepts a composable lambda to configure the icon. In this case, I use the `IconButton` component that wraps an `Icon`. I am not handling the click event yet (covered in a future post), so I pass in an empty lambda for the `onClick` parameter of the `IconButton`.

<script src="https://gist.github.com/BrianGardnerAtl/42dab32c2b0f21f4b54e6ba0bbde2fab.js"></script>

Running the app shows the new navigation icon in the app bar.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_9/app_bar_with_navigation_icon.png" alt="Emulator screenshot showing the app bar with the navigation icon."/>
</div>

## Option menu icons

Another common use of the app bar is to provide menu items for controls the user can take on the screen. A logical control for a list screen is an add button. The first step is to create a new drawable resource. Using the [Vector Asset Studio](https://developer.android.com/studio/write/vector-asset-studio) again, I selected the `add` icon and named it `ic_add`.

To display the option menu item in the `TopAppBar` there is a parameter called `actions`. This parameter also accepts a composable lambda where multiple items can be configured. This lambda operates within a `RowScope` since all of the icons are displayed in a `Row`.

The `IconButton` and `Icon` combo is used here as well to display the add action.

<script src="https://gist.github.com/BrianGardnerAtl/c4a6d72ea7ad5ca499e6ad3f624c8bd5.js"></script>

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_9/app_bar_with_add_action.png" alt="Emulator screenshot showing the app bar with the add action."/>
</div>

Duplicating the `IconButton` allows multiple to be displayed in the app bar.

<script src="https://gist.github.com/BrianGardnerAtl/0e4a4ce9ce52d075f10680a7626b35b4.js"></script>

These are displayed as expected.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_9/app_bar_with_multiple_add_actions.png" alt="Emulator screenshot showing the app bar with multiple add actions."/>
</div>

This leads to a question. When there are too many items to display in an app bar, the extra items should go in an overflow menu. What happens if there are many items in the `actions` lambda?

<script src="https://gist.github.com/BrianGardnerAtl/9c2659bf0732ede46ab37fb27763b6dc.js"></script>

Running this displays some interesting behavior.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_9/app_bar_with_too_many_add_actions.png" alt="Emulator screenshot showing the app bar with the actions pushing the title out of the way."/>
</div>

### Overflow issues

This is obviously an issue because the action items are not placed in an overflow menu by default. They are pushing the title out of the way and not all of the icons are even displayed in the toolbar (there are 9 actions declared in the previous snippet and only 7 are shown in the screenshot).

Currently there is no mechanism to force an overflow menu with the `TopAppBar`. This could be manually implemented by adding an overflow action and handling displaying a modal with the additional actions but this is a lot of extra work. The overflow option is coming in a future release so the best option is to limit the number of actions to something small and to test the app to ensure the title in the app bar is not compressed.

This post looked at the `TopAppBar` component to display an app bar for my Tweetish app. My next post will focus on using a `Floating Action Button` for the add action instead of the app bar action. Future posts will look into using the navigation drawer, bottom navigation, and applying a theme to the app.

Thanks for reading and stay tuned for more!
