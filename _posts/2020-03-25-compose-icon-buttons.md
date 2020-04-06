---
layout: post
title: "Container Deprecation and Icon Buttons"
post-excerpt: "Learn how to replace the Container component and a better way to draw icon resources"
social-image: "/assets/images/compose_4_5/social_image.png"
header: "/assets/images/compose_4_5/header_image.jpg"
---

This post aims at fixing an impending issue with my current Tweet view implementation. In the `dev08` release of Compose the `Container` component is going to be removed. I have several usages of this component which I want to remove before updating my version.

## Better icon drawing

The current usages of `Container` facilitate drawing vector assets for the action row items. While I could replace these instances with a `Box` component (the simple replacement for `Container`), there is a more suitable component available.

I can instead use the `Icon` component. This component has three parameters. The first is a `VectorAsset` which can be provided by the `vectorResource()` function. The other two parameters are optional. A `modifier` and `tint` can be provided to configure how the icon is drawn to the screen.

A regular `Icon` can be used for the `Comment`, `Like`, and `Retweet` items.

<script src="https://gist.github.com/BrianGardnerAtl/91cc5dd88e24bba857825251df80ee94.js"></script>

All of the parameters needed by the `Container` can be directly passed to the `Icon`. The `LayoutSize` modifier allows me to specify how large the icons need to be. The `tint` parameter allows me to handle coloring the icon without using the `drawVector()` function.

This implementation is only used for the `Comment`, `Like`, and `Retweet` items because they still need to be wrapped in the `Clickable` function. This is because both the `Icon` and `Text` should react to the click event.

The `Share` function can use a different component. It still needs the `Icon` to draw the share vector to the screen, but it can be wrapped in an `IconButton` to handle the click event instead.

<script src="https://gist.github.com/BrianGardnerAtl/843c5d40ad354f9636336b8bb81aa164.js"></script>

The `IconButton` wraps the `Icon` and has a parameter that accepts the `onClick` listener. Running the app now reveals an issue.

<img class="post-image" src="/assets/images/compose_4_5/share_icon_larger.png" alt="Preview pane showing the share icon is larger than the other action row icons."/>

The share icon is now taking up more space than the other icons, even though I specified the size of the `Icon` to be `24.dp`. The reason for this is found inspecting the source for `IconButton`.

<script src="https://gist.github.com/BrianGardnerAtl/db67721b48f19ca657856c19ce81a700.js"></script>

The `IconButton` is implemented using a `Box`. It has a default modifier applied called `IconButtonSizeModifier` which is defined as:

<script src="https://gist.github.com/BrianGardnerAtl/1259dedbcb4fc4b355edf38dfc55fc21.js"></script>

Since I do not set the size of my `IconButton` it defaults to `48.dp`. To fix this I need to explicitly specify the size of the button.

<script src="https://gist.github.com/BrianGardnerAtl/9d506c74c2c88ffc370e8d3636a2d673.js"></script>

This forces the share icon to be the correct size.

<img class="post-image" src="/assets/images/compose_4_5/share_icon_correct_size.png" alt="Preview pane showing the share icon the correct size."/>

## Add ripple animation

One other change becomes apparent when interacting with the `Share` button. The `IconButton` provides a ripple animation out of the box.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_4_5/share_button_ripple.webm" type="video/webm">
        Emulator screen recording of the ripple animation when clicking the share icon.
    </video>
</div>

It would be nice for the other action row items to have a ripple as well. Adding this animation involves wrapping the `Clickable` function with a `Ripple`. This function provides a ripple animation when tapped. The `bound` parameter indicates if the animation should be constrained to the bounds of the view or if it should be allowed to extend beyond.

<script src="https://gist.github.com/BrianGardnerAtl/1308616a358cf26a46ca1f3e018507f1.js"></script>

Providing `false` for the bound property causes the animation to go outside the size of the button.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_4_5/other_action_item_ripples.webm" type="video/webm">
        Emulator screen recording of the ripple animation for the other action row items.
    </video>
</div>

By default, the ripple size is based on the size of the `Ripple` contents. The `radius` property can be provided to force a certain size but this runs the risk of the ripple being smaller than the content, which would look odd.

Providing `true` for the bound property forces the ripple to stay within the view bounds.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_4_5/constrained_ripples.webm" type="video/webm">
        Emulator screen recording of the ripple animations bound to the size of the contents.
    </video>
</div>

This may work better for buttons since the animation will not bleed over into the other content. However, in this case I will leave the ripples unbounded so they match the share ripple. The current `IconButton` implementation does not allow modifications to the `Ripple` animation. I would rather have the animations be consistent in shape, even if their sizes are different.

With that, all of the deprecated `Container`s have been removed from my project and drawing the icons has been improved. As a bonus, the action row items now have a nice ripple animation to give the user some visual feedback.

Thanks for reading and stay tuned for more!

Photo by [Stefany Andrade](https://unsplash.com/@stefany_andrade) on [Unsplash](https://unsplash.com)
