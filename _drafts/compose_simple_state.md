---
layout: post
title: "Compose Simple State"
post-excerpt: "Learn how to add basic view state and how to conditionally add views in your composable functions."
---

# Adding a Dash of State

So far all of the data displayed in the Tweet view is static. The composable functions have parameters for the data they display but these values are hard-coded in `TweetView`. In this post the Tweet data is collected into a model object and fed to the composable functions.

Displaying the text data, such as the display name and twitter handle, does not require updates to those composable functions. Some of the action row functions need updates in order to display the interaction counts for the various actions.

## Create a Tweet model

The first step is to create a model object that stores the data for a single Tweet. A data class is perfect for this since it only acts as data storage.

```kotlin
data class Tweet(
    val displayName: String,
    val handle: String,
    val time: String,
    val content: String,
    val commentCount: Int,
    val retweetCount: Int,
    val likeCount: Int
)
```

For now all of the data in the Tweet class is read-only. This may need to change later when the action row items modify the state.

Next, the `TweetView` function needs to take a `Tweet` as a parameter.

```kotlin
@Composable
fun TweetView(tweet: Tweet) {
    Row {
        // row contents collapsed
    }
}
```

This requires updates to the `onCreate` function in `MainActivity`. It needs to build a `Tweet` object and pass it to the `TweetView`.

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
            retweetCount = 10,
            likeCount = 1000
        )
        setContent {
            MaterialTheme {
                TweetView(
                    tweet
                )
            }
        }
    }
}
```

Similar code added to the preview function allows the project to build successfully.

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
        retweetCount = 10,
        likeCount = 1000
    )
    MaterialTheme {
        TweetView(
            tweet
        )
    }
}
```

## Display the text

With the sample Tweet data loaded it can be passed to the relevant, low-level composable functions to display. Passing this data to `UserInfoRow` and `TweetContent` is straightforward.

```kotlin
@Composable
fun TweetView(tweet: Tweet) {
    Row {
        Column {
            UserInfoRow(
                name = tweet.displayName,
                handle = tweet.handle,
                time = tweet.time
            )
            TweetContent(content = tweet.content)
            ActionRow()
        }
    }
}
```

The data is pulled from the tweet object and passed directly to the other functions. The whole tweet class is not passed to these other functions in order to keep them clean. Their only concern is taking in their String data and displaying it. This helps simplify any test code for these classes. It also prevents requiring updates to those functions if the Tweet class ever changes.

## Display the interaction count

Next, some of the action row functions require changes to display the interaction counts. First, the counts are passed to the `ActionRow` so they can be forwarded to the other functions.

```kotlin
@Composable
fun ActionRow(
    commentCount: Int,
    retweetCount: Int,
    likeCount: Int
) {
    val context = ContextAmbient.current
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceAround
    ) {
        // row content collapsed
    }
}
```

The `TweetView` function then passes those counts to the `ActionRow`.

```kotlin
@Composable
fun TweetView(tweet: Tweet) {
    Row {
        Column {
            UserInfoRow(
                name = tweet.displayName,
                handle = tweet.handle,
                time = tweet.time
            )
            TweetContent(content = tweet.content)
            ActionRow(
                commentCount = tweet.commentCount,
                retweetCount = tweet.retweetCount,
                likeCount = tweet.likeCount
            )
        }
    }
}
```

A count parameter is added to the `Comment`, `Retweet`, and `Like` functions.

```kotlin
@Composable
fun Comment(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        // clickable content collapsed
    }
}

@Composable
fun Retweet(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        // clickable content collapsed
    }
}

@Composable
fun Like(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        // clickable content collapsed
    }
}
```

And finally, the counts are passed from the `ActionRow` to each of the functions.

```kotlin
@Composable
fun ActionRow(
    commentCount: Int,
    retweetCount: Int,
    likeCount: Int
) {
    val context = ContextAmbient.current
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceAround
    ) {
        Comment(commentCount) {
            Toast.makeText(context, "Clicked on comment", Toast.LENGTH_SHORT).show()
        }
        Retweet(retweetCount) {
            Toast.makeText(context, "Clicked on retweet", Toast.LENGTH_SHORT).show()
        }
        Like(likeCount) {
            Toast.makeText(context, "Clicked on like", Toast.LENGTH_SHORT).show()
        }
        Share {
            Toast.makeText(context, "Clicked on share", Toast.LENGTH_SHORT).show()
        }
    }
}
```

Once the count values are passed to the functions, each one needs an update to display the count next to the icon. Since the icon and count are displayed next to each other, `Row` is the perfect element to arrange them.

The `Text` compose element is used to display the count. Padding is added to the text so there is a gap between the icon and count. A `TextStyle` is also applied to configure the size and color of the count text.

```kotlin
@Composable
fun Comment(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_comment)
        Row {
            Container(
                height = 24.dp,
                width = 24.dp
            ) {
                // container contents collapsed
            }
            Text(
                text = "$count",
                modifier = LayoutPadding(8.dp, 0.dp, 0.dp, 0.dp),
                style = TextStyle(
                    fontSize = 18.sp,
                    color = Color.LightGray
                )
            )
        }
    }
}

@Composable
fun Retweet(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_retweet)
        Row {
            Container(
                height = 24.dp,
                width = 24.dp
            ) {
                // container contents collapsed
            }
            Text(
                text = "$count",
                modifier = LayoutPadding(8.dp, 0.dp, 0.dp, 0.dp),
                style = TextStyle(
                    fontSize = 18.sp,
                    color = Color.LightGray
                )
            )
        }
    }
}

@Composable
fun Like(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_like)
        Row {
            Container(
                height = 24.dp,
                width = 24.dp
            ) {
                // container contents collapsed
            }
            Text(
                text = "$count",
                modifier = LayoutPadding(8.dp, 0.dp, 0.dp, 0.dp),
                style = TextStyle(
                    fontSize = 18.sp,
                    color = Color.LightGray
                )
            )
        }
    }
}
```

With that in place, the preview shows the counts in the Tweet view.

<img class="post-image" src="/assets/images/compose_4/action_row_counts.png" alt="Preview pane showing the tweet view with the action row counts displayed"/>

### Conditional view creation

The Tweet view looks good but there is one edge case that needs fixing. If the count for one of the interactions is zero then the text should not display. In the old view system on Android this would require modifying the visibility of the view. With Compose, the views can be conditionally added.

```kotlin
@Composable
fun Comment(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_comment)
        Row {
            Container(
                height = 24.dp,
                width = 24.dp
            ) {
                // container content collapsed
            }
            if (count > 0) {
                Text(
                    // text content collapsed
                )
            }
        }
    }
}

@Composable
fun Retweet(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_retweet)
        Row {
            Container(
                height = 24.dp,
                width = 24.dp
            ) {
                // container content collapsed
            }
            if (count > 0) {
                Text(
                    // text content collapsed
                )
            }
        }
    }
}

@Composable
fun Like(count: Int, onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_like)
        Row {
            Container(
                height = 24.dp,
                width = 24.dp
            ) {
                // container content collapsed
            }
            if (count > 0) {
                Text(
                    // text content collapsed
                )
            }
        }
    }
}
```

Wrapping the `Text` in an if statement requires that the count is greater than zero. If it is not then the text is not added to the Row. Modifying the preview function to set the comment and like counts in the sample Tweet to zero displays this in the preview.

<img class="post-image" src="/assets/images/compose_4/action_row_counts_hidden.png" alt="Preview pane showing the tweet view with the zero counts hidden"/>

With that, the simple state implementation is complete. Over the next couple blog posts the `Comment`, `Like`, and `Retweet` components will be able to update the view state. The like and retweet will toggle on and off, and comment will increment the count since users can comment multiple times on the same Tweet.

Thanks for reading and stay tuned for more!
