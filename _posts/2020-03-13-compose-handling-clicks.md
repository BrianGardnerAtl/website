---
layout: post
title: "Handle Composable Clicks"
post-excerpt: "Learn how to respond to click events from composable functions as well as how to access the Context object."
---

The action row buttons look good so far but they are not functional. Tapping
on them does nothing which is disappointing for users. This post focuses
on making composable views clickable. When finished, the actions displays a `Toast`
when clicked to verify the events are received. In the next couple posts, the composable views
will accept state as parameters and the action buttons will modify the state.

## Capture Click Event

Handling clicks with composable views differs from the regular view system on
Android. The old way is by setting a `View.OnClickListener`
implementation on the view to handle the events. Composable functions do not
have this listener so a different approach is needed.

Instead, Compose has a class called `Clickable`. This object wraps the composable view that needs to react to click events. `Clickable` accepts an `onClick` function as a parameter that is called for each click event.

Implementing this requires two steps. The first is to wrap the action row items in the `Clickable` class and the second is to pass in the click listeners. Views should only be responsible for displaying data. This also means that they should not handle their own click events. For this, the click listeners are passed as parameters to the action row functions. One of the higher-level functions is responsible for declaring the listeners instead.

<script src="https://gist.github.com/BrianGardnerAtl/f742bc778ff842086e9e57d258a19c86.js"></script>

The `Comment` function now accepts the `onClick` lambda and passes it to the `Clickable` component. None of the previous code is modified, it is just wrapped inside of the clickable element. The same treatment is applied to the `Retweet`, `Like`, and `Share` functions.

<script src="https://gist.github.com/BrianGardnerAtl/a5a566c08cef29089bbe38074a322c39.js"></script>

With the action row functions ready, the next step is to pass in the click listeners from the `ActionRow`. Kotlin syntax for passing lambdas outside of the parenthesis makes the code cleaner.

<script src="https://gist.github.com/BrianGardnerAtl/307d0868f9ba5f818281abfaf90e0923.js"></script>

With the lambdas in place, all that is needed is the toast.

## Show the Toast

The toast messages will indicate that the click events are successfully handled. Making the toast requires a `Context` object, the string to display, and the duration. This lead to an interesting question.

The composable functions are not defined inside of my `MainActivity` because I want them to be usable from other classes. They are also not a typical Android `View` class. This means that the typical way of accessing the `Context` object does not work. Usually, this is accessed by reference to the activity or pulled from the view itself.

Compose provides **Ambient** components to access information from composable functions. Some components you can access are context, screen density, layout direction, and coroutine context. Each of these is contained in an ambient class. This class is accessible from any composable function. The `current` property is used to access the ambient component value in a composable function.

<script src="https://gist.github.com/BrianGardnerAtl/0cb83975ea70bafff1dd8f8fb106b818.js"></script>

With the context object, a `Toast` can now be displayed to the user. Each action row item will show a different message to differentiate them.

<script src="https://gist.github.com/BrianGardnerAtl/97234f45c7eec8282a5dab90b9e2389c.js"></script>

In order to see the toast messages in action, the app needs to run on a device or emulator. To do this, I need to tell `MainActivity` to display my `TweetView` as its contents.

<script src="https://gist.github.com/BrianGardnerAtl/387ec14c8c8174249be7436556dee0ef.js"></script>

With that in place I can run the app to see my `TweetView`. Clicking on the action row items displays the toast messages.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_3/compose_toast.png" alt="Toast message displayed on an emulator after clicking on the Like action row view"/>
</div>

With that, my action row views are clickable. In the next couple blog posts I will implement state for my composable function. The action row will display the number of comments, retweets, and likes. The state will also be modified when the user clicks on the corresponding action row views.

Thanks for reading and stay tuned for more!
