---
layout: post
title: "Compose Toggleable Buttons"
post-excerpt: "FILL IN BEFORE POSTING"
---

# Modify State With Toggle Buttons

[In my previous post](fill_this_in), a `Tweet` class is added to hold the state for the `TweetView`. In this post, some of the action buttons are set up to modify the state when clicked.

The comment button will increment the comment count since a user can leave multiple comments on a single Tweet. The like and retweet buttons will act as a toggle. These also need a selected state so users know if they have liked or retweeted that Tweet.

# Increment the comment count

A few things are needed to handle a click on the comment action item. The click event lambda needs to handle updating the comment count, and the `Tweet` class needs to be updated so our composable functions can react to changes to its properties.

Modifying the click lambda is first. Incrementing the comment count will require moving the declaration of the lambda to a higher-order composable function. It is currently initialized in the `ActionRow` function, but there is no reference to the `Tweet` class here. Passing the tweet down to the `ActionRow` would pollute it with unnecessary information. Instead, the event listener will be lifted up to the `TweetView` function.

First, the `ActionRow` is updated to accept an event listener for a click on the comment view. This listener is passed to the `Comment` function.

```kotlin
@Composable
fun ActionRow(
    commentCount: Int,
    commentClick: (() -> Unit),
    retweetCount: Int,
    likeCount: Int
) {
    val context = ContextAmbient.current
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceAround
    ) {
        Comment(commentCount, commentClick)
        // Other action row items collapsed
    }
}
```

Next, the `TweetView` function implements the event listener and passes it to the `ActionRow` function.

```kotlin
@Composable
fun TweetView(tweet: Tweet) {
    val commentClick: (() -> Unit) = {
        // update Tweet comment count
    }
    Row {
        Column {
            UserInfoRow(
                // user info row parameters collapsed
            )
            TweetContent(content = tweet.content)
            ActionRow(
                commentCount = tweet.commentCount,
                commentClick = commentClick,
                retweetCount = tweet.retweetCount,
                likeCount = tweet.likeCount
            )
        }
    }
}
```

With the listener hooked up, the next step is to modify the `Tweet` class. The `commentCount` property needs to be changed from `val` to `var` so the listener can write to it.

```kotlin
data class Tweet(
    val displayName: String,
    val handle: String,
    val time: String,
    val content: String,
    var commentCount: Int, // now a var instead of a val
    val retweetCount: Int,
    val likeCount: Int
)
```

Then the value can be changed in the `TweetView`.

```kotlin
@Composable
fun TweetView(tweet: Tweet) {
    val commentClick: (() -> Unit) = {
        tweet.commentCount += 1
    }
    Row {
        // row contents collapsed
    }
}
```

Running the app now surfaces a problem. Clicking on comment does not change the value shown on the screen. Adding a log statement in the listener and running the app again verifies that it is reacting to the click event, but it is not changing the UI.

```kotlin
@Composable
fun TweetView(tweet: Tweet) {
    val commentClick: (() -> Unit) = {
        tweet.commentCount += 1
        Log.d("Comment", "Count incremented")
    }
    Row {
        // row contents collapsed
    }
}
```

<img class="post-image" src="/assets/images/compose_5/comment_listener_log.png" alt="Log statement printed after clicking on the comment action item"/>

The reason this happens is there is nothing telling the `TweetView` to redraw itself when the contents of the `Tweet` changes. Compose provides an annotation for this functionality. Adding the **@Model** annotation to the `Tweet` class tells the `TweetView` to listen for changes to the tweet properties. If one of the properties changes, then the `TweetView` will redraw itself to display the new data.

```kotlin
@Model
data class Tweet(
    // tweet properties collapsed
)
```

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

```kotlin
@Composable
fun Like(count: Int, liked: Boolean, onLikeChanged : (Boolean) -> Unit) {
    Toggleable(value = liked, onValueChange = onLikeChanged) {
        val icon = vectorResource(R.drawable.ic_like)
        Row {
            // row contents collapsed
        }
    }
}
```

The contents of the `Row` are unchanged so they are collapsed for brevity.

The `Clickable` function is changed to `Toggleable`. This function requires a value parameter and a value change listener. A boolean flag, `liked`, is passed to the `Like` function which is forwarded to `Toggleable`. The `onClick` listener parameter is updated to `onLikeChanged` which requires a `Boolean` property for the lambda. This listener is passed as the `onValueChange` property to `Toggleable`.

Similar changes are made to `Retweet`.

```kotlin
@Composable
fun Retweet(count: Int, retweeted: Boolean, onRetweetChanged : (Boolean) -> Unit) {
    Toggleable(value = retweeted, onValueChange = onRetweetChanged) {
        val icon = vectorResource(R.drawable.ic_retweet)
        Row {
            // row contents collapsed
        }
    }
}
```

In fact, these are the exact same changes made to `Like`. Since this is the final form of these functions, this duplication can be extracted to a new composable function.

```kotlin
@Composable
fun ToggleImage(
    @DrawableRes iconId: Int,
    count: Int,
    checked: Boolean,
    onValueChange: ((Boolean) -> Unit)
) {
    val icon = vectorResource(iconId)
    Toggleable(value = checked, onValueChange = onValueChange) {
        Row {
            Container(
                expanded = true,
                height = 24.dp,
                width = 24.dp
            ) {
                DrawVector(
                    vectorImage = icon,
                    tintColor = Color.LightGray
                )
            }
            if (count > 0) {
                Text(
                    text = "$count",
                    modifier = LayoutPadding(8.dp, 0.dp, 0.dp, 0.dp),
                    style = TextStyle(
                        color = Color.LightGray,
                        fontSize = 18.sp
                    )
                )
            }
        }
    }
}
```

`Like` and `Retweet` are simplified by calling through to this new function.

```kotlin
@Composable
fun Retweet(count: Int, retweeted: Boolean, onRetweetChanged : (Boolean) -> Unit) {
    ToggleImage(
        iconId = R.drawable.ic_retweet,
        count = count,
        checked = retweeted,
        onValueChange = onRetweetChanged
    )
}

@Composable
fun Like(count: Int, liked: Boolean, onLikeChanged : (Boolean) -> Unit) {
    ToggleImage(
        iconId = R.drawable.ic_like,
        count = count,
        checked = liked,
        onValueChange = onLikeChanged
    )
}
```

With the low-level functions finished, the `ActionRow` function needs updates to pass the new parameters. The new toggle listeners will be passed from `TweetView` just like the `commentClick` function so `ActionRow` can continue to have know knowledge of the `Tweet` class itself.

```kotlin
@Composable
fun ActionRow(
    commentCount: Int,
    commentClick: (() -> Unit),
    retweeted: Boolean,
    retweetCount: Int,
    onRetweetChanged: (Boolean) -> Unit,
    liked: Boolean,
    likeCount: Int,
    onLikeChanged: (Boolean) -> Unit
) {
    val context = ContextAmbient.current
    Row(
        // row parameters collapsed
    ) {
        Comment(commentCount, commentClick)
        Retweet(retweetCount, retweeted, onRetweetChanged)
        Like(likeCount, liked, onLikeChanged)
        Share {
            Toast.makeText(context, "Clicked on share", Toast.LENGTH_SHORT).show()
        }
    }
}
```

The `TweetView` now needs to provide all these parameters to the `ActionRow`. There is one thing to fix first. The `Tweet` class needs properties for the liked and retweeted state. These are added as `var` properties so they can change. The `retweetCount` and `likeCount` properties are also changed to `var`s so they can be updated as well.

```kotlin
@Model
data class Tweet(
    val displayName: String,
    val handle: String,
    val time: String,
    val content: String,
    var commentCount: Int,
    var retweeted: Boolean,
    var retweetCount: Int,
    var liked: Boolean,
    var likeCount: Int
)
```

The `onCreate()` in `MainActivity` and the preview function need to add these properties to the `Tweet` so the project can build.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val tweet = Tweet(
            displayName = "Brian Gardner",
            handle = "@BrianGardnerDev",
            time = "7m",
            content = "This is a test tweet",
            commentCount = 100,
            retweeted = false,
            retweetCount = 10,
            liked = false,
            likeCount = 1000
        )
        setContent {
            // setContent contents collapsed
        }
    }
}
```

```kotlin
@Preview
@Composable
fun TwitterPreview() {
    val tweet = Tweet(
        displayName = "Brian Gardner",
        handle = "@BrianGardnerDev",
        time = "7m",
        content = "This is a test tweet",
        commentCount = 100,
        retweeted = false,
        retweetCount = 10,
        liked = false,
        likeCount = 1000
    )
    MaterialTheme {
        // contents collapsed
    }
}
```

Now the `TweetView` can provide the necessary data to the `ActionRow`.

```kotlin
@Composable
fun TweetView(tweet: Tweet) {
    val commentClick: (() -> Unit) = {
        tweet.commentCount += 1
    }
    val retweetToggle: ((Boolean) -> Unit) = { retweet ->
        tweet.retweeted = retweet
        if (retweet) {
            tweet.retweetCount = tweet.retweetCount + 1
        } else {
            tweet.retweetCount = tweet.retweetCount - 1
        }
    }
    val likedToggle: ((Boolean) -> Unit) = { liked ->
        tweet.liked = liked
        if (liked) {
            tweet.likeCount = tweet.likeCount + 1
        } else {
            tweet.likeCount = tweet.likeCount - 1
        }
    }
    Row {
        Column {
            UserInfoRow(
                // user info row parameters collapsed
            )
            TweetContent(content = tweet.content)
            ActionRow(
                commentCount = tweet.commentCount,
                commentClick = commentClick,
                retweeted = tweet.retweeted,
                retweetCount = tweet.retweetCount,
                onRetweetChanged = retweetToggle,
                liked = tweet.liked,
                likeCount = tweet.likeCount,
                onLikeChanged = likedToggle
            )
        }
    }
}
```

The toggle functions set the current state on the tweet, then modify the count based on which way the toggle goes. If the user likes the tweet the count is incremented, and if they unlike it then it is decremented.

<div class="center-screenshot">
    <video class="post-emulator-recording" controls preload="auto">
        <source src="/assets/images/compose_5/toggle_actions_implemented.webm" type="video/webm">
        Emulator screen recording of the retweet and like actions toggling when clicked.
    </video>
</div>

To wrap up, a little color added to the toggles helps the users figure out their current state at a glance. The light gray color is used for the off state but a different color would be nice for the on state.

To implement this, a selected color is added as a property of the `ToggleImage` function. This color is then set as the `tintColor` fo the `DrawVector` function as well as the `color` parameter of the `TextStyle` for the count text.

```kotlin
@Composable
fun ToggleImage(
    @DrawableRes iconId: Int,
    count: Int,
    checked: Boolean,
    selectedColor: Color,
    onValueChange: ((Boolean) -> Unit)
) {
    val icon = vectorResource(iconId)
    val color = if (checked) {
        selectedColor
    } else {
        Color.LightGray
    }
    Toggleable(value = checked, onValueChange = onValueChange) {
        Row {
            Container(
                expanded = true,
                height = 24.dp,
                width = 24.dp
            ) {
                DrawVector(
                    vectorImage = icon,
                    tintColor = color
                )
            }
            if (count > 0) {
                Text(
                    text = "$count",
                    modifier = LayoutPadding(8.dp, 0.dp, 0.dp, 0.dp),
                    style = TextStyle(
                        color = color,
                        fontSize = 18.sp
                    )
                )
            }
        }
    }
}
```

The color green is used for `Retweet` and red is used for `Like`.

```kotlin
@Composable
fun Retweet(count: Int, retweeted: Boolean, onRetweetChanged : (Boolean) -> Unit) {
    ToggleImage(
        iconId = R.drawable.ic_retweet,
        count = count,
        checked = retweeted,
        selectedColor = Color.Green,
        onValueChange = onRetweetChanged
    )
}

@Composable
fun Like(count: Int, liked: Boolean, onLikeChanged : (Boolean) -> Unit) {
    ToggleImage(
        iconId = R.drawable.ic_like,
        count = count,
        checked = liked,
        selectedColor = Color.Red,
        onValueChange = onLikeChanged
    )
}
```

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
