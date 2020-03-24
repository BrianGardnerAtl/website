---
layout: post
title: "Compose Simple State"
post-excerpt: "Learn how to add basic view state and how to conditionally add views in your composable functions."
header: "/assets/images/compose_4/header_image.jpg"
---

So far all of the data displayed in the Tweet view is static. The composable functions have parameters for the data they display but these values are hard-coded in `TweetView`. In this post the Tweet data is collected into a model object and fed to the composable functions.

Displaying the text data, such as the display name and twitter handle, does not require updates to those composable functions. Some of the action row functions need updates in order to display the interaction counts for the various actions.

## Create a Tweet model

The first step is to create a model object that stores the data for a single Tweet. A data class is perfect for this since it only acts as data storage.

<script src="https://gist.github.com/BrianGardnerAtl/a60dc87281caf1e9c51eeb0e96858855.js"></script>

For now all of the data in the Tweet class is read-only. This may need to change later when the action row items modify the state.

Next, the `TweetView` function needs to take a `Tweet` as a parameter.

<script src="https://gist.github.com/BrianGardnerAtl/fad4ef9c5800bd373f94daafedc5386a.js"></script>

This requires updates to the `onCreate` function in `MainActivity`. It needs to build a `Tweet` object and pass it to the `TweetView`.

<script src="https://gist.github.com/BrianGardnerAtl/45120005bcd43d0fb45ec2539ca7caf0.js"></script>

Similar code added to the preview function allows the project to build successfully.

<script src="https://gist.github.com/BrianGardnerAtl/ca590ac8f00e3d6060b264c8d55548e6.js"></script>

## Display the text

With the sample Tweet data loaded it can be passed to the relevant, low-level composable functions to display. Passing this data to `UserInfoRow` and `TweetContent` is straightforward.

<script src="https://gist.github.com/BrianGardnerAtl/3aa15930e31292f6f673eff1d3ef3d58.js"></script>

The data is pulled from the tweet object and passed directly to the other functions. The whole tweet class is not passed to these other functions in order to keep them clean. Their only concern is taking in their String data and displaying it. This helps simplify any test code for these classes. It also prevents requiring updates to those functions if the Tweet class ever changes.

## Display the interaction count

Next, some of the action row functions require changes to display the interaction counts. First, the counts are passed to the `ActionRow` so they can be forwarded to the other functions.

<script src="https://gist.github.com/BrianGardnerAtl/e2347a99133361f19ebc3361ea63e285.js"></script>

The `TweetView` function then passes those counts to the `ActionRow`.

<script src="https://gist.github.com/BrianGardnerAtl/a26e7d9b0e7ec92c2a3022783356b5b3.js"></script>

A count parameter is added to the `Comment`, `Retweet`, and `Like` functions.

<script src="https://gist.github.com/BrianGardnerAtl/95a7b2a18a5c6b9031a79a18862bb862.js"></script>

And finally, the counts are passed from the `ActionRow` to each of the functions.

<script src="https://gist.github.com/BrianGardnerAtl/adeacbcc248ee0e1db678c03a58c3ad7.js"></script>

Once the count values are passed to the functions, each one needs an update to display the count next to the icon. Since the icon and count are displayed next to each other, `Row` is the perfect element to arrange them.

The `Text` compose element is used to display the count. Padding is added to the text so there is a gap between the icon and count. A `TextStyle` is also applied to configure the size and color of the count text.

<script src="https://gist.github.com/BrianGardnerAtl/9a0ced2931bc368e8fafd60234fc9090.js"></script>

With that in place, the preview shows the counts in the Tweet view.

<img class="post-image" src="/assets/images/compose_4/action_row_counts.png" alt="Preview pane showing the tweet view with the action row counts displayed"/>

### Conditional view creation

The Tweet view looks good but there is one edge case that needs fixing. If the count for one of the interactions is zero then the text should not display. In the old view system on Android this would require modifying the visibility of the view. With Compose, the views can be conditionally added.

<script src="https://gist.github.com/BrianGardnerAtl/1855c779d083efb58bf43ac7ef1df9d6.js"></script>

Wrapping the `Text` in an if statement requires that the count is greater than zero. If it is not then the text is not added to the Row. Modifying the preview function to set the comment and like counts in the sample Tweet to zero displays this in the preview.

<img class="post-image" src="/assets/images/compose_4/action_row_counts_hidden.png" alt="Preview pane showing the tweet view with the zero counts hidden"/>

With that, the simple state implementation is complete. Over the next couple blog posts the `Comment`, `Like`, and `Retweet` components will be able to update the view state. The like and retweet will toggle on and off, and comment will increment the count since users can comment multiple times on the same Tweet.

Thanks for reading and stay tuned for more!

Photo by [Hal Gatewood](https://unsplash.com/@halgatewood) on [Unsplash](https://unsplash.com)
