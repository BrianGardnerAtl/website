---
layout: post
title: "Display a List of Tweets"
post-excerpt: "FILL IN BEFORE POSTING"
---

In this post, the user will be able to see a list of tweets instead of just one. There are a couple components available to achieve this. I will review the pros and cons of each, as well as dive into the best candidate to implement the feature. Along the way, I will highlight the issues I encountered as well as some open issues I have not resolved yet.

## Multiple options

Currently there are a couple options for displaying a list of items. The first is the `VerticalScroller`. This component accepts a lambda as the last parameter to generate the children to display. The issue with this component is that it creates all of the child views at once. This works well for a small number of items but the memory performance is poor when the number of items to display grows.

The `dev05` release of compose introduced the `AdapterList` component. This functions similarly to the `RecyclerView`, but it is much easier to setup. The `AdapterList` only generates the views that are currently on the screen so the performance is much better for large lists of items.

## Display a list of tweets

The list of tweets is represented as another composable function. To get something displaying quickly, it initially accepts a list of `Tweet` objects as a parameter.

<script src="https://gist.github.com/BrianGardnerAtl/ad09d795ab399b2278127fbd174ed857.js"></script>

This list is passed to the `data` parameter of the `AdapterList`. As each item is displayed on screen, the associated `Tweet` object is passed to the `itemCallback` lambda.

<script src="https://gist.github.com/BrianGardnerAtl/88fa83142735b69b8adfdc80e49769bf.js"></script>

The `Tweet` object is wrapped in a `MutableState` and passed to the `TweetView`.

<script src="https://gist.github.com/BrianGardnerAtl/b7aaff469c24b21fb9623e23b904d7bf.js"></script>

## List preview

In order to see the current list, a new preview function is needed. To simplify things, a generator function is created first to generate a test list of Tweets. It returns a `MutableList` since the list state will need to change as the user clicks on the action row items in the separate `TweetView`s.

<script src="https://gist.github.com/BrianGardnerAtl/a09b30b23a769224a6c8913b74029d48.js"></script>

This generator function is used in a new preview function so the tweet list can be viewed.

<script src="https://gist.github.com/BrianGardnerAtl/a8ece4ff0e30e95cdc9f5aed61d34113.js"></script>

After building the project, the `TweetList` preview is visible in the preview pane.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_8/tweet_list_preview.png" alt="Preview pane showing the tweet list view."/>
</div>

Only 7 of the tweets are displayed in the preview so it must determine how many items to display based on a test screen size. Either way, it is looking good so far. One improvement is to add some padding to the `AdapterList` to give it some extra space.

<script src="https://gist.github.com/BrianGardnerAtl/7fdded9e1e92017103bb2a948d386d85.js"></script>

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_8/tweet_list_with_padding.png" alt="Preview pane showing the tweet list view with added padding."/>
</div>

Seeing the list in the preview is nice but I want to see how it performs on a device. Updating the `onCreate` function allows me to run it on my emulator.

<script src="https://gist.github.com/BrianGardnerAtl/986c1ac1808a1a31e2d56ff129235515.js"></script>

With that in place, I can run the app and see the `TweetList` in action. The performance right now is pretty janky but I expect it to improve quite a bit in the future.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_8/tweet_list_performance.webm" type="video/webm">
        Emulator screen recording of the tweet list scrolling performance.
    </video>
</div>

## Current issues

The scrolling performance is one issue, but there is another one that is more severe. Clicking on the action row items on any of the `TweetView`s no longer updates the state of the view. My first thought is that even though the `TweetView` is trying to update its state, the overall state of the `AdapterList` is not being changed. Since the `AdapterList`'s data is not changing it is not recomposing the individual rows.

To address this, the listener functions need to be brought up to the `TweetList` function instead of the `TweetView`. The `TweetView` is updated to only take in a `Tweet` object instead of a `MutableState` as well. The `TweetList` should be the only view with state since it needs to track the list of tweets as a whole.

First, the `TweetView` will be simplified. The parameter is made a regular `Tweet` object and the listener declarations are removed in favor of additional parameters.

<script src="https://gist.github.com/BrianGardnerAtl/e89b1395dff1146fcd6dde468fa3ce9e.js"></script>

The `TweetList` is updated next to create the required listeners and pass them into the `TweetView`. The index of the individual tweet is also needed in order for the listeners to update the correct place in the tweet list when the user clicks on the action row items. In order for the list to be updated, its type must be changed to `MutableList` as well.

<script src="https://gist.github.com/BrianGardnerAtl/c41ea533cd97480a643f640876b9a68d.js"></script>

To get the project to build, the preview function for the `TweetView` also requires updates to match the new parameter list.

<script src="https://gist.github.com/BrianGardnerAtl/e2acef05381c8143720839af46a426ca.js"></script>

With these updates in place, the app can build. One more step is needed. The `AdapterList` needs a `MutableState` parameter so it knows to recompose its child views when the list changes.

<script src="https://gist.github.com/BrianGardnerAtl/273cf5137a04a820a877b39d15847889.js"></script>

This requires updates to the `onCreate` function in `MainActivity` as well as updates to the preview function so the project can build.

<script src="https://gist.github.com/BrianGardnerAtl/808fbdbc1595771746c90384ee2119d6.js"></script>

<script src="https://gist.github.com/BrianGardnerAtl/294e571f16e2ef5a9b6bf665fc6762af.js"></script>

With these updates the `AdapterList` should observe any updates made to the list of tweets and update the corresponding `TweetView` row accordingly. However, the `AdapterList` is still not updating the individual rows. Based on my discussions with some of the folks working on Compose it seems like this is likely a bug in dev07. A bug ticket has been filed for this issue so I am hopeful it will be resolved soon!

With that, a list of tweets is now visible on the screen. `AdapterList` takes the performance benefits of `RecyclerView` and greatly simplifies the setup.

Thanks for reading and stay tuned for more!
