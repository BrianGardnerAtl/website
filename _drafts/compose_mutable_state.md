---
layout: post
title: "Immutable Data, Mutable State"
post-excerpt: "FILL IN BEFORE POSTING"
---

This post focuses on improving the state implementation from [my previous blog post](). At the end of the post the `Tweet` class is mutable. The click listeners update the interaction count values as well as the like and retweet state. While this does work, I prefer to keep my classes immutable. This forces me to be explicit when the state changes, thus reducing side effects from other areas in the code.

The first goal is to make the `Tweet` class immutable. In this case it is not technically immutable since `val` just means read-only, but it's good enough for this use case.

<script src="https://gist.github.com/BrianGardnerAtl/f3f03a1a80ce80d1af45eefcfba4eb0b.js"></script>

Now that the `Tweet` cannot be updated by the listeners, another mechanism is needed to serve this purpose. This is where `MutableState` comes in to play. The javadocs of the file state

```
* The [MutableState] class can be used several different ways. For example, the most basic way is to store the returned state
* value into a local immutable variable, and then set the [MutableState.value] property on it.
```

This object is used to store an immutable variable. In this case, the `Tweet` object is wrapped by this `MutableState`. To do this, first the `TweetView` parameter needs to be changed to the `MutableState` type.

<script src="https://gist.github.com/BrianGardnerAtl/ca379144d7ffef5c7344dd6db8ae41d1.js"></script>

Next, the `onCreate()` and preview functions need to wrap the `Tweet` object in a `MutableState`. There is a `mutableStateOf()` function that can accomplish this.

```kotlin
fun <T> mutableStateOf(
    value: T,
    areEquivalent: (old: T, new: T) -> Boolean = ReferentiallyEqual
): MutableState<T> = ModelMutableState(value, areEquivalent)
```

This function accepts the state value as the first parameter. The second is a function that determines if two states are equal. The default value of this is `ReferentiallyEqual` which just determines if the two value objects have the same reference. This works for an immutable value object but if the value object can change, there is also a `StructurallyEqual` option that checks if the properties of the two value objects are equal.

In this case, the `ReferentiallyEqual` will be used since I will create a new `Tweet` class any time the data needs to change. I can update the `onCreate()` function to implement the state.

<script src="https://gist.github.com/BrianGardnerAtl/b55205faaae072e522aa830bd1c60d72.js"></script>

The same is needed in the preview function.

<script src="https://gist.github.com/BrianGardnerAtl/52c8636dab5967857dd0e96a13d1725b.js"></script>

With that in place, the `TweetView` function can now use the state to setup the data and listeners. First, the `Tweet` is pulled out of the `MutableState` using the `value` property.

<script src="https://gist.github.com/BrianGardnerAtl/0dabda5b77cac681f3d3aed3047c9990.js"></script>

Accessing the value property in `TweetView` subscribes it to any changes to the MutableState. When the `Tweet` object changes the new data will be sent to `TweetView`, which will display it.

Each of the three listeners: `commentClick`, `retweetToggle`, and `likeToggle` need updates in order to work with the new immutable `Tweet` object. The `copy()` function is used heavily to modify only the necessary data. The `commentClick` function is a good place to start since the logic is straightforward.

<script src="https://gist.github.com/BrianGardnerAtl/c5b96fe4eb061949da07c53f68736509.js"></script>

The new count value is calculated from the current `Tweet` value. This new count is fed to the `copy()` function to create a new `Tweet` object for the updated state. Setting the new state is as simple as setting the `value` on the `MutableState` object.

Similar logic is applied to the `retweetToggle` and `likeToggle` listeners. These are slightly more complex because they decide how to modify the count based on the current toggle state.

<script src="https://gist.github.com/BrianGardnerAtl/7f0b5c70a5391999e5c624cb3a7e823b.js"></script>

With this in place the app should build and run successfully. It should also function the same as before. Clicking on the comment action increments the comment count, while clicking on retweet or like toggles the count up and down.

Now the `Tweet` class is immutable but the state can still be changed thanks to `MutableState`. This class will be a big help in projects because having immutable data classes for the state value requires more explicit changes for the state.

My next post will focus on showing a profile image for the user, which will wrap up the basic Tweet view. Further posts will show how to display a list of Tweets as well as how to show a floating action button over the list.

Thanks for reading and stay tuned for more!
