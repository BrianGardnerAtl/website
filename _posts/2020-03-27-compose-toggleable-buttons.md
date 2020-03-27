---
layout: post
title: "Compose Toggleable Buttons"
post-excerpt: "Learn how to modify the state of a model class and how Toggleable is different from Clickable."
social-image: "/assets/images/compose_5/social_image.png"
header: "/assets/images/compose_5/header_image.jpg"
---

[In my previous post](https://briangardner.tech/2020/03/25/compose-icon-buttons.html), the `Container` components are removed since they will be deprecated soon. The vector assets are also drawn using the `Icon` and `IconButton` components. In this post, some of the action buttons are set up to modify the state when clicked.

The comment button will increment the comment count since a user can leave multiple comments on a single Tweet. The like and retweet buttons will act as a toggle. These also need a selected state so users know if they have liked or retweeted that Tweet.

# Increment the comment count

A few things are needed to handle a click on the comment action item. The click event lambda needs to handle updating the comment count, and the `Tweet` class needs to be updated so our composable functions can react to changes to its properties.

Modifying the click lambda is first. Incrementing the comment count requires moving the declaration of the lambda to a higher-order composable function. It is currently initialized in the `ActionRow` function, but there is no reference to the `Tweet` class there. Passing the tweet down to the `ActionRow` would pollute it with unnecessary information. Instead, the event listener will be lifted up to the `TweetView` function.

First, the `ActionRow` is updated to accept an event listener for a click on the comment view. This listener is passed to the `Comment` function.

<script src="https://gist.github.com/BrianGardnerAtl/015158ff2fbc21d2926b105954eece31.js"></script>

Next, the `TweetView` function implements the event listener and passes it to the `ActionRow` function.

<script src="https://gist.github.com/BrianGardnerAtl/1dc71e331eb13003e9ccf65a246c07b7.js"></script>

With the listener hooked up, the next step is to modify the `Tweet` class. The `commentCount` property needs to be changed from `val` to `var` so the listener can write to it.

<script src="https://gist.github.com/BrianGardnerAtl/0dc3d534e3e26487f430a1cbee126cb5.js"></script>

Then the value can be changed in the `TweetView`.

<script src="https://gist.github.com/BrianGardnerAtl/212a805371242c9af4ebbfc96583ebfc.js"></script>

Running the app now surfaces a problem. Clicking on comment does not change the value shown on the screen. Adding a log statement in the listener and running the app again verifies that it is reacting to the click event, but it is not changing the UI.

<script src="https://gist.github.com/BrianGardnerAtl/e6bce9738d8c733586fd25ea7a1126d3.js"></script>

<img class="post-image" src="/assets/images/compose_5/comment_listener_log.png" alt="Log statement printed after clicking on the comment action item"/>

The reason this happens is there is nothing telling the `TweetView` to redraw itself when the contents of the `Tweet` changes. Compose provides an annotation for this functionality. Adding the **@Model** annotation to the `Tweet` class tells the `TweetView` to listen for changes to the tweet properties. If one of the properties changes, then the `TweetView` will redraw itself to display the new data.

<script src="https://gist.github.com/BrianGardnerAtl/be47e2519f7a6ac5cdc6b3103ed86b3d.js"></script>

The class comment on the annotation provides details on exactly what this annotation does.

```
 * [Model] can be applied to a class which represents your application's data model, and will cause
 * instances of the class to become observable, such that a read of a property of an instance of
 * this class during the invocation of a composable function will cause that component to be
 * "subscribed" to mutations of that instance. Composable functions which directly or indirectly
 * read properties of the model class, the composables will be recomposed whenever any properties
 * of the the model are written to.
```

Since `TweetView` reads the `commentCount` property from the `Tweet`, it implicitly subscribes to changes to the value. When the `commentClick` listener modifies that value, the `TweetView` is redrawn to the screen.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_5/comment_count_increment.webm" type="video/webm">
        Emulator screen recording of the comment count incrementing on click.
    </video>
</div>

With that, the comment functionality is finished. Next, the like and retweet actions are setup.

## Implement toggle actions

The like and retweet actions are slightly different than comment. Instead of incrementing the count for each click, they need to toggle on and off since a user can only like or retweet once. While this could be implemented using the `Clickable` function, there is a better way.

The `dev05` release of Compose introduced the `Toggleable` function which is exactly what the like and retweet actions need. This function accepts a `value` parameter indicating the current state of the toggle. It also accepts an `onValueChange` listener that receives the new value as a boolean flag when clicked. `Like` is updated first.

<script src="https://gist.github.com/BrianGardnerAtl/d7ba8243396ec8465f56dc95baeb0392.js"></script>

The `Clickable` function is changed to `Toggleable`. This function requires a value parameter and a value change listener. A boolean flag, `liked`, is passed to the `Like` function which is forwarded to `Toggleable`. The `onClick` listener parameter is updated to `onLikeChanged` which requires a `Boolean` property for the lambda. This listener is passed as the `onValueChange` property to `Toggleable`.

Similar changes are made to `Retweet`.

<script src="https://gist.github.com/BrianGardnerAtl/4e1f776d0d9019aa792eecafce97b630.js"></script>

In fact, these are the exact same changes made to `Like`. Since this is the final form of these functions, this duplication can be extracted to a new composable function.

<script src="https://gist.github.com/BrianGardnerAtl/7c3484f0d2906c10727d6f27f8aefea2.js"></script>

`Like` and `Retweet` are simplified by calling through to this new function.

<script src="https://gist.github.com/BrianGardnerAtl/6c914bfd68807d8c3915da95b016bea6.js"></script>

With the low-level functions finished, the `ActionRow` function needs updates to pass the new parameters. The new toggle listeners will be passed from `TweetView` just like the `commentClick` function so `ActionRow` can continue to have know knowledge of the `Tweet` class itself.

<script src="https://gist.github.com/BrianGardnerAtl/c7fb738d5fcb33f49d131f233e37c80e.js"></script>

The `TweetView` now needs to provide all these parameters to the `ActionRow`. There is one thing to fix first. The `Tweet` class needs properties for the liked and retweeted state. These are added as `var` properties so they can change. The `retweetCount` and `likeCount` properties are also changed to `var`s so they can be updated as well.

<script src="https://gist.github.com/BrianGardnerAtl/041bf26144dad311f696cf9b86090868.js"></script>

The `onCreate()` in `MainActivity` and the preview function need to add these properties to the `Tweet` so the project can build.

<script src="https://gist.github.com/BrianGardnerAtl/3ed284985ecdb4b98d50ef2b515ec51b.js"></script>

<script src="https://gist.github.com/BrianGardnerAtl/88c447e7b4e98f46ea5c022d428a1590.js"></script>

Now the `TweetView` can provide the necessary data to the `ActionRow`.

<script src="https://gist.github.com/BrianGardnerAtl/b4d8a2aad6a613601bf25abed2f2553d.js"></script>

The toggle functions set the current state on the tweet, then modify the count based on which way the toggle goes. If the user likes the tweet the count is incremented, and if they unlike it then it is decremented.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_5/toggle_actions_implemented.webm" type="video/webm">
        Emulator screen recording of the retweet and like actions toggling when clicked.
    </video>
</div>

To wrap up, a little color added to the toggles helps the users figure out their current state at a glance. The light gray color is used for the off state but a different color would be nice for the on state.

To implement this, a selected color is added as a property of the `ToggleImage` function. This color is then set as the `tintColor` fo the `DrawVector` function as well as the `color` parameter of the `TextStyle` for the count text.

<script src="https://gist.github.com/BrianGardnerAtl/115ad0655b4d0472b26419ee7dbc9f86.js"></script>

The color green is used for `Retweet` and red is used for `Like`.

<script src="https://gist.github.com/BrianGardnerAtl/7233025d510c9da31a3407c485917325.js"></script>

With this in place, the app now styles the like and retweet actions based on their state.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_5/toggle_actions_styled.webm" type="video/webm">
        Emulator screen recording of the retweet and like actions toggling when clicked.
    </video>
</div>

In this post the action row is setup to modify the state of the view based on click events. The comment count increments each time it is clicked while the like and retweet counts toggle back and forth.

In the next post the state management will be further improved by making the `Tweet` class read only. Instead of having `var` properties, everything will be declared as a `val` to have an effectively immutable data class. This will provide some data protection because the individual model objects cannot be updated.

Thanks for reading and stay tuned for more!

Photo by [Hal Gatewood](https://unsplash.com/@halgatewood) on [Unsplash](https://unsplash.com)
