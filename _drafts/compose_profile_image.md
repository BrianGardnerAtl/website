---
layout: post
title: "Display a Round Profile Image"
post-excerpt: "FILL IN BEFORE POSTING"
---

Twitter would not be complete without a round profile image on the left side of the Tweet. This post focuses on this last component of the Tweet view. For right now, a default profile image will be displayed for each Tweet instead of trying to download a photo.

## Current issue

Downloading images into an `ImageView` on Android is mostly a solved problem. Libraries like [Glide](https://github.com/bumptech/glide) and [Picasso](https://square.github.io/picasso/) provide fluent APIs for downloading images, performing transformations, and implement caching to boost app performance.

Unfortunately, these libraries do not currently work with Jetpack Compose. They rely on loading images into something like an `ImageView` so it may be a little while before they are updated to work with Compose.

## Profile image compose function

The initial setup for the profile image mirrors the action row items. A vector asset is needed for the default user photo. This asset is loaded with the `vectorResource` function, which is displayed in a `Container` using the `drawVector` function.

The icon is added using the [Vector Asset Studio](https://developer.android.com/studio/write/vector-asset-studio) just like the other action row icons. I used the `person` icon for the default asset and named it `ic_profile_photo_default`.

With the icon in place, the next step is adding a `@Composable` annotated function for the profile image and grab a reference to the default photo.

<script src="https://gist.github.com/BrianGardnerAtl/e29f8bfaa8ff74aea529b7fd90a12d65.js"></script>

Next, the default photo is drawn into a `Container` with the `drawVector` function. I tried a few different sizes and ended up liking how the profile image looked at `36dp`.

<script src="https://gist.github.com/BrianGardnerAtl/04f5d978aa392922123aafd502db165d.js"></script>

Before continuing further, I want to be able to preview my profile image and see how it looks. I could try adding it to the `TweetView` in its current state so it shows up in the existing preview but I would rather just focus on the profile image at this point. What I can do is create another `@Preview` annotated function in the same file.

<script src="https://gist.github.com/BrianGardnerAtl/3e8ca1e45ea70281f1ac2cf0ad4ac710.js"></script>

After building the project, both preview functions will render in the preview pane. This is very useful for checking how separate compose functions render in the same file.

<img class="post-image" src="/assets/images/compose_7/two_preview_functions.png" alt="Preview pane showing two composable views rendered"/>

While the icon is displayed on the screen, it is difficult to see on a light background. Drawing the icon on top of a dark gray background would help it stand out. This is accomplished by wrapping the icon `Container` in a `Surface` function and providing a background color. The `Surface` is drawn first and the icon is then drawn on top of it.

<script src="https://gist.github.com/BrianGardnerAtl/ec69631156c5f8c0b97b96829832f839.js"></script>

Building the project updates the preview pane and shows the new background.

<div class="post-icon">
    <img src="/assets/images/compose_7/profile_image_with_background.png" alt="Preview pane showing the profile image preview with a dark gray background color."/>
</div>

The background color looks good but the square shape does not. The profile image should be circular to match Twitter. Modifying the shape of a component is the responsibility of the `drawClip` modifier function. This function accepts a shape as a parameter and it will clip it's children to that shape.

<script src="https://gist.github.com/BrianGardnerAtl/73b88b33ee2636da7677d43bda5a5286.js"></script>

This code produces the pleasing, round profile image I am looking for.

<div class="post-icon">
    <img src="/assets/images/compose_7/round_profile_image.png" alt="Preview pane showing the circle clipped profile image."/>
</div>

The last update to the profile image is to add padding. I attempted to add the padding directly to the `drawClip` modifier on the `Surface` but this caused the surface to render as a square again. I am not sure if this is a bug on the `Surface` or not, but I had to find another solution.

What I came up with is to wrap the `Surface` function in another `Container` and add the padding to that.

<script src="https://gist.github.com/BrianGardnerAtl/a610a4dd306471361d2d85c888eb982a.js"></script>

This ensures that the profile image always has a buffer between it and the other `TweetView` contents.

<div class="post-icon">
    <img src="/assets/images/compose_7/profile_image_with_padding.png" alt="Preview pane showing the profile image with additional padding."/>
</div>

## Display profile image in TweetView

With the profile image complete it just needs to be added to the `TweetView`. All of the `Column` contents in the `TweetView` remain the same. All that is needed is a top-level `Row` that displays the `ProfileImage` first.

<script src="https://gist.github.com/BrianGardnerAtl/64d9a8a38ce887e28794cf7ce9305b09.js"></script>

This displays the user's profile image in the correct place in the `TweetView`.

<img class="post-image" src="/assets/images/compose_7/tweet_view_with_profile_image.png" alt="Preview pane showing the profile image displayed in the tweet view."/>

With that, the basic Tweet view is complete! It has been a long journey to get here but along the way I have learned a lot about how compose displays data on the screen and I have grown to love this type of view creation. I am excited to see what Jetpack Compose introduces in the future and how it will change Android development.

Fortunately, this blog series is not ending here. In future posts I am going to focus on displaying a list of my `TweetView`s, displaying a floating action button above the list for adding a Tweet, and modifying the theme of my app.

Thanks again for reading and stay tuned for more!
