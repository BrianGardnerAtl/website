---
layout: post
title: "Bottom Navigation Bar With Compose"
post-excerpt: "FILL OUT BEFORE POSTING"
social-image: "FILL OUT BEFORE POSTING"
header: "FILL OUT BEFORE POSTING"
---

The Twitter app uses a navigation drawer for some navigation destinations but it also uses a bottom bar for the most often used screens: home, search, notifications, and messages.  This post adds the bottom bar component to my sample app and also shows how the docked floating action button positions work with the bottom bar.

## Create the bottom bar component

The bottom bar hosts several `IconButton` components to display the navigation options and handle the click events. Check out [one of my previous posts](https://briangardner.tech/2020/03/25/compose-icon-buttons.html) if you want to learn more about the `IconButton`. The first step is adding icon resources for these buttons. I use vector resources for each destination. In [the asset studio](https://developer.android.com/studio/write/image-asset-studio) the `home`, `search`, `notifications`, and `mail` clip art work for this example.

Once the icons are ready the composable function for the bottom bar is added. Four listeners are passed as parameters. These are hooked up to the `IconButton` components to handle the click events.

<script src="https://gist.github.com/BrianGardnerAtl/8b67991bc77b89206ea4ed87c65ac128.js"></script>

The content parameter for the `BottomAppBar` uses a `RowScope`. This means I can use `Modifier.weight()` to ensure that each `IconButton` takes up the same width in the row.

## Add bottom bar to `Scaffold`

Once the initial component is setup, it is added to the `Scaffold` component to display it. 

<script src="https://gist.github.com/BrianGardnerAtl/67aa32f4525037cd8be2681fb3f7172c.js"></script>

Once the `Scaffold` knows about the bottom bar it is shown on the screen with the other contents.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_12/bottom_app_bar.png" alt="Emulator screenshot showing the bottom app bar displayed on screen."/>
</div>

The color of the bar is set automatically using the primary color of the app theme. The color of the icons are also set from the theme. In a future post I will show how to create a custom theme with Compose.

## FAB docked positions

Another thing to notice is that the floating action button moved up so it is not overlapping with the bottom app bar. This helpful positioning comes from the `Scaffold` arranging the components on the screen.

In my [floating action button post](https://briangardner.tech/2020/05/08/compose-floating-action-button.html) I covered the `End` and `Center` positions for the button. There are two other positions that work with the bottom app bar by docking the button within the bar. The button will overlap with the bar and the bar will have a cutout to make room.

Implementing this requires a couple steps. First, the position of the floating action button is set in the `Scaffold` using the `floatingActionButtonPosition` property. In this example I use `EndDocked` but you can also use `CenterDocked` if you want the button in the middle.

<script src="https://gist.github.com/BrianGardnerAtl/612a0f53de4ef0c2ff6706a67d4e6e5b.js"></script>

Running the app shows the floating action button overlapping with the bottom app bar.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_12/fab_overlap_bottom_bar.png" alt="Emulator screenshot showing the floating action button overlapping with the bottom app bar."/>
</div>

The reason for this is that the bottom app bar does not yet know anything about the floating action button position. The `Scaffold` positions the button but the bottom bar needs to know where it is in order to layout itself.

There is a property on the `BottomAppBar` called `FabConfiguration` that handles this. When the bottom bar is setup in the `Scaffold`, the lambda provided accepts a `FabConfiguration` parameter that can be passed to the `BottomAppBar`. First, the `TweetBottomBar` is updated to accept a `FabConfiguration` parameter and pass it to the `BottomAppBar` constructor. The cutout shape is also provided so the bottom bar knows what shape the button is.

<script src="https://gist.github.com/BrianGardnerAtl/7f01d7d293d227d6200bed6545151ac5.js"></script>

Then the `FabConfiguration` is passed to the `TweetBottomBar` from within the `Scaffold`.

<script src="https://gist.github.com/BrianGardnerAtl/0a1eed81f46a6a99ed425faefe4b04f8.js"></script>

Running the app now shows another issue.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_12/fab_overlap_bottom_bar_contents.png" alt="Emulator screenshot showing the floating action button displayed with a cutout in the bottom app bar, but overlapping the contents."/>
</div>

The floating action button is positioned correctly and the bottom app bar has a cutout, but the messages icon is being cut off. In order to avoid this, an empty `Box` can be added to the `BottomAppBar` content to ensure the icons are not clipped by the button.

<script src="https://gist.github.com/BrianGardnerAtl/e9f959114b7ecadfb2cad189a5af009b.js"></script>

Once this spacer is in place the icons are pushed away from the floating action button.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_12/final_cutout_position.png" alt="Emulator screenshot showing the floating action button displayed with a cutout in the bottom app bar, but overlapping the contents."/>
</div>

With that, the bottom app bar is complete. It provides a simple means to add navigation icons in easy reach for your users. Simplifying the floating action button cutout is a nice bonus as well.

Thanks for reading and stay tuned for more!