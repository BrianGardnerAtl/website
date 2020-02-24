---
layout: post
title: "Handle Composable Clicks"
post-excerpt: ""
---

# Handle Composable Clicks

The action row buttons look good so far but they are not yet functional. Tapping
on them does nothing which would be disappointing for users. This post focuses
on making composable views clickable. When finished, the view displays a toast
when clicked to verify it works. In the next couple posts, the composable views
will accept state as parameters and the action buttons will be updated to modify
the state.

# Capture Click

Handling clicks with composable views differs from the regular view system on
Android. The old way to handle clicks is to provide a `View.OnClickListener`
implementation to the view to handle the events. Composable functions do not
have this listener so a different approach is needed.

Instead, Compose has a class called `Clickable`. This object wraps the composable view that needs to react to click events. `Clickable` accepts an `onClick` function as a parameter that fires when the view is clicked.

Implementing this requires two steps. The first is to wrap the action row items in the `Clickable` class and the second is to pass in the click listeners. Views should only be responsible for displaying data. This also means that they should not handle their own click events. For this, the click listeners are passed as parameters to the action row composable functions so one of the higher-level functions is responsible for the clicks instead.

```kotlin
@Composable
fun Comment(onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_comment)
        Container(
            height = 24.dp,
            width = 24.dp
        ) {
            // container contents collapsed
        }
    }
}
```

The `Comment` function now accepts the `onClick` lambda and passes it to the `Clickable` component. None of the previous code is modified, it is just wrapped inside of the clickable element. The same treatment is applied to the `Retweet`, `Like`, and `Share` functions so they can handle clicks as well.

```kotlin
@Composable
fun Retweet(onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_retweet)
        Container(
            height = 24.dp,
            width = 24.dp
        ) {
            // container contents collapsed
        }
    }
}

@Composable
fun Like(onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_like)
        Container(
            height = 24.dp,
            width = 24.dp
        ) {
            // container contents collapsed
        }
    }
}

@Composable
fun Share(onClick : () -> Unit) {
    Clickable(onClick = onClick) {
        val icon = vectorResource(R.drawable.ic_share)
        Container(
            height = 24.dp,
            width = 24.dp
        ) {
            // container contents collapsed
        }
    }
}
```

With the action row composable functions ready, the next step is to pass in the click listeners from the `ActionRow` function. Kotlin syntax for passing lambdas outside of the parenthesis makes for cleaner code.

```kotlin
@Composable
fun ActionRow() {
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceAround
    ) {
        Comment {
            // display toast
        }
        Retweet {
            // display toast
        }
        Like {
            // display toast
        }
        Share {
            // display toast
        }
    }
}
```

With the lambdas in place, all that is needed is the toast.

# Make Toast

The toast messages will indicate that the click events are successfully handled. Making the toast requires a `Context` object, the string to display, and the duration. This lead to an interesting question.

The composable functions are not defined inside of my `MainActivity` because I want them to be usable from other places. They are also not a typical Android `View` class. This means that my typical way of accessing the `Context` object will not work. Typically this is accessed by reference to the activity or just pulled from the view itself.

Compose provides **Ambient** components to access information from composable functions. Some components you can access are context, screen density, layout direction, and coroutine context. Each of these is contained in an ambient class. This class is accessible from the composable functions. The `current` is used to access the ambient component value in a composable function.

```kotlin
@Composable
fun ActionRow() {
    val context = ContextAmbient.current
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceAround
    ) {
        // action row items collapsed
    }
}
```

With the context object, Toasts can now be displayed to the user. Each action row item will show a different message so I know which one is clicked on.

```kotlin
@Composable
fun ActionRow() {
    val context = ContextAmbient.current
    Row(
        modifier = LayoutWidth.Fill + LayoutPadding(8.dp),
        arrangement = Arrangement.SpaceAround
    ) {
        Comment {
            Toast.makeText(context, "Clicked on comment", Toast.LENGTH_SHORT).show()
        }
        Retweet {
            Toast.makeText(context, "Clicked on retweet", Toast.LENGTH_SHORT).show()
        }
        Like {
            Toast.makeText(context, "Clicked on like", Toast.LENGTH_SHORT).show()
        }
        Share {
            Toast.makeText(context, "Clicked on share", Toast.LENGTH_SHORT).show()
        }
    }
}
```

In order to see the toast messages in action, the app needs to run on a device or emulator. To do this, I'll need to tell `MainActivity` to display my `TweetView` as its contents.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                TweetView()
            }
        }
    }
}
```

With that in place I can run the app to see my `TweetView`. Clicking on the action row items displays the toast messages.

<div class="center-screenshot">
    <img class="post-device-screenshot" src="/assets/images/compose_3/compose_toast.png" alt="Toast message displayed on an emulator after clicking on the Like action row view"/>
</div>

With that, my action row views are clickable. In the next couple blog posts I will implement view state for my composable function. The action row will display the number of comments, retweets, and likes. The state will also be modified when the user clicks on the corresponding action row views.

Thanks for reading and stay tuned for more!
