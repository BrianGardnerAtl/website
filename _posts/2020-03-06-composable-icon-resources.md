---
layout: post
title: "Composable Icon Resources"
post-excerpt: "Learn how to use vector icons with compose and how to space items out in a Row."
---

[My previous blog post](/2020/02/27/compose-setup-and-text.html) covers setting up an Android project with Jetpack Compose and displaying text. This post focuses on displaying icons for the Tweet's action row.

But first, a quick detour. The Tweet content lives between the user information row and the action row. This content is wrapped in a `Text` element along with an applied `TextStyle`. Since this composable item is not contained in a `Row` it also needs padding applied to it directly.

```kotlin
@Composable
fun TweetContent(content: String) {
    return Text(
        text = content,
        style = TextStyle(
            color = Color.Black,
            fontSize = 12.sp
        ),
        modifier = LayoutPadding(8.dp)
    )
}
```

Next up is displaying the content in the preview to verify it looks correct. This is simplified by combining the `UserInfoRow` and `TweetContent` composable items in another composable function. The action row will also be added to this new composable function.

```kotlin
@Composable
fun TweetView() {
    Row {
        Column {
            UserInfoRow(
                name = "Brian Gardner",
                handle = "@BrianGardnerDev",
                time = "7m"
            )
            TweetContent(content = "Sample tweet content")
        }
    }
}
```

Updating the `@Preview` function displays the current view in the preview pane in Android Studio.

```kotlin
@Preview
@Composable
fun TwitterPreview() {
    MaterialTheme {
        TweetView()
    }
}
```

<img class="post-image" src="/assets/images/compose_2/tweet_content_preview.png" alt="Preview pane displaying the user info row and tweet content"/>

### Display the action row

With the Tweet content in place, the action row is the next item to tackle. In my implementation the action row will contain four user actions for a Tweet.

- Comment
- Retweet
- Like
- Share

Each of these actions display an image and react to click events. The Retweet and Like actions also have a selected state to modify their appearance.

Each action requires a separate compose function. Another function combines all of these actions into a row. To get something showing in the preview, `Button` is used as a placeholder for each action.

```kotlin
@Composable
fun ActionRow() {
    Row {
        Comment()
        Retweet()
        Like()
        Share()
    }
}

@Composable
fun Comment() {
    Button(text = "Comment")
}

@Composable
fun Retweet() {
    Button(text = "Retweet")
}

@Composable
fun Like() {
    Button(text = "Like")
}

@Composable
fun Share() {
    Button(text = "Share")
}
```

Adding the `ActionRow` into the `TweetView` displays the action buttons in the preview.

```kotlin
@Composable
fun TweetView() {
    Row(
        modifier = LayoutPadding(8.dp)
    ) {
        Column {
            UserInfoRow(
                name = "Brian Gardner",
                handle = "@BrianGardnerDev",
                time = "7m"
            )
            TweetContent(content = "Sample tweet content")
            ActionRow()
        }
    }
}
```

<img class="post-image" src="/assets/images/compose_2/initial_action_buttons.png" alt="Preview pane displaying the initial action buttons"/>

The buttons verify that the action row displays on the screen but it does not match the Tweet view. For this, I need icons. Android Studio provides a tool to create vector assets with the default material icons. To learn how to use it, check out [Vector Asset Studio](https://developer.android.com/studio/write/vector-asset-studio).

I found suitable icons for the comment, retweet, and share actions. I use `mode comment` for the comment action, `loop` for the retweet action, and `share` for the share action. I also set the color for these vector assets to white. White assets are easier to tint the correct color.

Surprisingly, I could not find a heart icon to use for the like action. A quick search on StackOverflow returned [this page](https://stackoverflow.com/questions/45618391/heart-shaped-button-in-android). The first answer has a vector path that is perfect for my use case (thanks Uddhav Gautam!). My only modification sets the `fillColor` to white.

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="24dp"
        android:height="24dp"
        android:viewportWidth="24.0"
        android:viewportHeight="24.0">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M12,21.35l-1.45,-1.32C5.4,15.36 2,12.28 2,8.5 2,5.42 4.42,3 7.5,3c1.74,0 3.41,0.81 4.5,2.09C13.09,3.81 14.76,3 16.5,3 19.58,3 22,5.42 22,8.5c0,3.78 -3.4,6.86 -8.55,11.54L12,21.35z"/>
</vector>
```

The action functions need to load and display these vector assets. Each function can load its icon using the `vectorResource()` function. The `Button` components can be removed since they are no longer needed.

```kotlin
@Composable
fun Comment() {
    val icon = vectorResource(R.drawable.ic_comment)
}

@Composable
fun Retweet() {
    val icon = vectorResource(R.drawable.ic_retweet)
}

@Composable
fun Like() {
    val icon = vectorResource(R.drawable.ic_like)
}

@Composable
fun Share() {
    val icon = vectorResource(R.drawable.ic_share)
}
```

When the images are loaded, the action items need a place to draw their icon. There is currently no composable item like `ImageButton` but the icons can be drawn manually. There is a `DrawVector()` function that handles drawing vector assets but it needs to be drawn into something. There is a composable called `Container` that fits this use case.

```kotlin
@Composable
fun Comment() {
    val icon = vectorResource(R.drawable.ic_comment)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon
        )
    }
}

@Composable
fun Retweet() {
    val icon = vectorResource(R.drawable.ic_retweet)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon
        )
    }
}

@Composable
fun Like() {
    val icon = vectorResource(R.drawable.ic_like)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon
        )
    }
}

@Composable
fun Share() {
    val icon = vectorResource(R.drawable.ic_share)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon
        )
    }
}
```

It is tempting to refactor this code to a base function since they only differ in their resource IDs. This would prove painful later on since some of the buttons have different behavior than others. Having a little patience ensures I do not end up with [the wrong abstraction](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction).

Once each action item has a container with a vector asset, the preview updates to show the action row.

<img class="post-image" src="/assets/images/compose_2/action_row_with_icons.png" alt="Preview pane displaying the icon version of the action row"/>

### Tint the icons

These icons work but they are difficult, if not impossible, to see on a light background. The `tintColor` attribute on the `DrawVector` function provides an easy way to specify the color of the icon.

```kotlin
@Composable
fun Comment() {
    val icon = vectorResource(R.drawable.ic_comment)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon,
            tintColor = Color.LightGray
        )
    }
}

@Composable
fun Retweet() {
    val icon = vectorResource(R.drawable.ic_retweet)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon,
            tintColor = Color.LightGray
        )
    }
}

@Composable
fun Like() {
    val icon = vectorResource(R.drawable.ic_like)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon,
            tintColor = Color.LightGray
        )
    }
}

@Composable
fun Share() {
    val icon = vectorResource(R.drawable.ic_share)
    Container(
        height = 24.dp,
        width = 24.dp
    ) {
        DrawVector(
            vectorImage = icon,
            tintColor = Color.LightGray
        )
    }
}
```

<img class="post-image" src="/assets/images/compose_2/action_row_tinted_icon.png" alt="Preview pane showing the action row with tinted icons"/>

### Space out the action row items

The action row has a similar problem to the original user info row. All of the icons are pushed to the left with no padding between them. Instead of manually adding spacing between the icons, I want my icons to be evenly spaced out.

The `Row` needs two attributes to add space. First, it needs the `LayoutWidth.Fill` modifier to take up the full width of the view. This leads to a question. The action row already has the `LayoutPadding` modifier, so how can another one be added?

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutPadding(8.dp)
    ) {
        Comment()
        Retweet()
        Like()
        Share()
    }
}
```

The solution is found by looking in the `Modifier.kt` file. In the comment at the top of the file is this line:

```
 * Modifier elements may be combined using `+` Order is significant; modifier elements to the left
 * are applied before modifier elements to the right.
```

Multiple modifiers can be combined using the `+` operator. Adding in the `LayoutWidth.Fill` modifier forces the row to take up the full width.

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp)
    ) {
        Comment()
        Retweet()
        Like()
        Share()
    }
}
```

<img class="post-image" src="/assets/images/compose_2/action_row_full_width.png" alt="Preview pane showing the full width action row"/>

Once the row is appropriately sized, providing an `Arrangement` parameter tells the row how to position the child elements. The default is `Arrangement.Begin` which places all of the items at the start of the row with no padding. Other options that do not add padding are `Arrangement.Center` and `Arrangement.End`.

#### Center Arrangement

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.Center
    ) {
        Comment()
        Retweet()
        Like()
        Share()
    }
}
```

<img class="post-image" src="/assets/images/compose_2/action_row_center_arrangement.png" alt="Preview pane showing the action row with a center arrangement"/>

#### End Arrangement

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.End
    ) {
        Comment()
        Retweet()
        Like()
        Share()
    }
}
```

<img class="post-image" src="/assets/images/compose_2/action_row_end_arrangement.png" alt="Preview pane showing the action row with an end arrangement"/>

There are three other arrangement options that provide spacing between the items: `SpaceEvenly`, `SpaceBetween`, and `SpaceAround`.

The code comments in `Flex.kt` describe what each of these do.

#### SpaceEvenly Arrangement

Place children such that they are spaced evenly across the main axis, including free space before the first child and after the last child.

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceEvenly
    ) {
        Comment()
        Retweet()
        Like()
        Share()
    }
}
```

<img class="post-image" src="/assets/images/compose_2/action_row_space_evenly_arrangement.png" alt="Preview pane showing the action row with a space evenly arrangement"/>

#### SpaceBetween Arrangement

Place children such that they are spaced evenly across the main axis, without free space before the first child or after the last child.

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceBetween
    ) {
        Comment()
        Retweet()
        Like()
        Share()
    }
}
```

<img class="post-image" src="/assets/images/compose_2/action_row_space_between_arrangement.png" alt="Preview pane showing the action row with a space between arrangement"/>

#### SpaceAround Arrangement

Place children such that they are spaced evenly across the main axis, including free space before the first child and after the last child, but half the amount of space existing otherwise between two consecutive children.

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceAround
    ) {
        Comment()
        Retweet()
        Like()
        Share()
    }
}
```

<img class="post-image" src="/assets/images/compose_2/action_row_space_around_arrangement.png" alt="Preview pane showing the action row with a space around arrangement"/>

My preference here is for the `SpaceAround` version. The row looks better with space before the comment icon and after the share icon, but not the full amount of space that `SpaceEvenly` uses.

In this post I added the Tweet content and action row to the Tweet view, as well as how to load and draw vector resources, and the different arrangement options for a row. Stay tuned for the next post in the series about handling click events!
