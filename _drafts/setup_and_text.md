---
layout: post
title: "Becoming a Composer"
---

# Becoming a Composer

[Jetpack Compose](https://developer.android.com/jetpack/compose) is a new
library from the Android team that simplifies UI creation. Small, composable
functions create the building blocks of your view which are combined into
higher-level components. These are pieced together to create your full UI.

In this blog series I'm going to focus on learning Jetpack Compose by
reacreating some well known views from popular applications. My first copy-cat
attempt is a Tweet view which provides plenty of room to explore composable
functions and deal with view state.

## Tweet view parts

![Insert a tweet view here]()

The view for a single tweet is relatively small but it holds a lot of data.
There's information about the user, such as their profile photo, display
name, and Twitter handle. There's also information for the tweet itself, such as
the content of the tweet and the time since posting. Lastly, there are actions
the reader can take on the Tweet, like commenting, retweeting, liking, and sharing.

Each of these individual components can be broken up into composable functions.
These functions should be small. Their only responsability is to take in some
basic data to display and provide a component to do so.

Once these individual components are in place, they are combined together to
create the full view. Each post in this series will look at a different part of
the Tweet view. This first post focuses on setting up a project with Compose and
displaying the user's display name, Twitter handle, and Tweet time.

### Composable Tweet Strategy



## Project setup

The first step to creating an app with Compose is ensuring you have the right
version of Android Studio. Version 4 and up should have the tools needed, but at
this time it is still in canary so you will need to download it from
[the preview page](https://developer.android.com/studio/preview) if you do not
have it yet. Once it is installed you are ready to go.

Creating the project is similar to the regular flow with the exception of your
starter activity. When prompted, enter 'Empty Compose Activity' to get setup
with a basic template.

<img src="/assets/images/compose_1/empty_compose_activity.png" alt="Empty
compose activity selection" width="700"/>

After that, just make sure your min SDK version is at least 21 and you can
create the project. Once the project is ready you should be able to build the
app to see the preview page populate. If you do not see the preview page when
your `MainActivity` class is open, click on the split button in the top-right of
the screen to see the code and preview.

<img src="/assets/images/compose_1/split_button.png" alt="Code and
preview split button" width="700"/>

### Version updates

At the time of this writing, the version of the Jetpack Compose library provided
in the template project is slightly behind the latest version. I did not run
into issues with this until trying to deal with vector image loading but this
seems like a better place to take care of it. I'll refer back here when the
series gets to that point to make sure your project is setup correctly.

First, open up the app-level `build.gradle` file and update the compose library
dependencies. At the time of this writing they need to be updated to dev04 from
dev02.

```kotlin
dependencies {
    ...
    implementation 'androidx.ui:ui-layout:0.1.0-dev04'
    implementation 'androidx.ui:ui-material:0.1.0-dev04'
    implementation 'androidx.ui:ui-tooling:0.1.0-dev04'
    ...
}
```

You will also need to add some custom options into the android block.

```kotlin
android {
  ...

  composeOptions {
      kotlinCompilerVersion "1.3.61-dev-withExperimentalGoogleExtensions-20200129"
      kotlinCompilerExtensionVersion "0.1.0-dev04"
  }
}
```

With these in place I have the right version of compose as well as the
right kotlin compiler extensions. For details on the current version of compose,
check out the [compose release notes](https://developer.android.com/jetpack/androidx/releases/compose).

Finally, the bulk of the default code in `MainActivity` can be removed to
simplify things.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {

            }
        }
    }
}
```

## Showing some user data

The first piece of Tweet data to display is the user information at the top of
the tweet. These are the user's display name, Twitter handle, and Tweet time.
Displaying a piece of text in compose involves creating a composable function
whose only responsibility is to diplay the text. Since there are three text
items, add three functions for each item.

I'll note that these functions are not defined inside of the `MainActivity`
class. They should be usable from other places as well. I just put them in the
same file for convenience.

```kotlin
@Composable
fun DisplayName(name: String) {
    Text(
        text = name
    )
}

@Composable
fun Handle(handle: String) {
    Text(
        text = handle
    )
}

@Compsable
fun PostTime(time: String) {
    Text(
        text = "time"
    )
}
```

Each function is annontated with `@Compose` so the system knows these are
composable functions for building the UI. Each function takes in the string the
view needs to display and displays it using the `Text` function.

Now that these low-level functions are in place, they can be composed in a
higher-level function. The goal here is to display them in  a row at the top of
the Tweet view so another composable function is needed.

```kotlin
@Composable
fun UserInfoRow(name: String, handle: String, time: String) {
    Row {
        DisplayName(name)
        Handle(handle)
        PostTime(time)
    }
}
```

To display items in a row with compose, you just use the `Row` component. This
component takes a lambda argument with each of the composable items it needs to
render.

### Checking progress in the preview

Building composable functions is quick and I want to be able to verify they look
correct before moving forward. Adding the views to my `MainActivity` view is one
option but then I have to wait for the project to build and deploy each time I
want to change something and see how it looks. Instead, I can take advantage of
the `@Preview` annotation.

At the bottom of the generated `MainActivity` class there is a default preview
function. This uses the default Hello Android! text so it is not useful so I'll
change it to display my `UserInfoRow`.

```kotlin
@Preview
@Composable
fun TwitterPreview() {
    MaterialTheme {
        UserInfoRow(
            name = "Brian Gardner",
            handle = "@BrianGardnerDev",
            time = "7m"
        )
    }
}
```

The `MaterialTheme` allows me to apply a theme to my composable functions if I
want but I'm not using it right now. In a later blog post I'll take a look at
what the theme does and how to modify it.

When I add in the `UserInfoRow` in my preview function I can open the split view
to see the preview. If it does not show up then I need to build the project to
see my view.

<img src="/assets/images/compose_1/user_info_row_view.png" alt="User info row
view displayed in the preview pane" width="700"/>

The text is displayed and it is shown in a row, but it still looks pretty bad.
All of the text is the same size and there is no space between any of the
items. By default, the `Row` will take up as much space as all of the child
items and will leave no extra space for padding.

### Adding some space

Fortunately, adding space to the text is relatively straightforward. I will take
a look at the `DisplayName` composable first. There is a `modifier` parameter to
the `Text` composable where padding can be applied.

```kotlin
@Composable
fun DisplayName(name: String) {
    Text(
        text = name,
        modifier = LayoutPadding(0.dp, 0.dp, 8.dp, 0.dp)
    )
}
```

The `LayoutPadding` object has two constructors. The one I use above allows me
to specify the padding for the left, top, right, and bottom of the view
individually. I only pass in a value for the right padding so I can add space
between the `DisplayName` and the `Handle`. The other constructor takes in a
single parameter and applies that value to all four areas around the view.

<img src="/assets/images/compose_1/padding_after_display_name.png" alt="Padding between
display name and handle composable views" width="700"/>

With that padding in place, I just need to add some padding after the Handle as
well.

```kotlin
@Composable
fun Handle(handle: String) {
    Text(
        text = handle,
        modifier = LayoutPadding(0.dp, 0.dp, 8.dp, 0.dp)
    )
}
```

@Composable
fun Handle(handle: String) {
    Text(
        text = handle,
        modifier = LayoutPadding(0.dp, 0.dp, 8.dp, 0.dp),
        style = TextStyle(
            color = Color.DarkGray, // TODO update this with an appropriate theme color
            fontSize = 12.sp
        )
    )
}

@Composable
fun PostTime(time: String) {
    Text(
        text = time,
        style = TextStyle(
            color = Color.DarkGray, // TODO update this with an appropriate theme color
            fontSize = 12.sp
        )
    )
}<img src="/assets/images/compose_1/user_info_row_spaced_out.png" alt="Padding
added between handle and post time composable views" width="700"/>

### Styling text

Last up for this post is to style the text to look more like the Tweet view. The
`DisplayName` should be bold with a black color to draw attention while the `Handle` and
`PostTime` should be regular and a gray color. All of the text should also be
smaller. I'll start with the `DisplayName`.

To modify the style of `Text` there is a `style` parameter where you can pass in
a `TextStyle` object. This object has plenty of options for modifying how the
text is displayed on the screen.

```kotlin
@Composable
fun DisplayName(name: String) {
    Text(
        text=name,
        modifier = LayoutPadding(0.dp, 0.dp, 8.dp, 0.dp),
        style = TextStyle(
            color = Color.Black,
            fontSize = 12.sp,
            fontWeight = FontWeight.Bold
        )
    )
}
```

For the `DisplayText` I just need three of the attributes. I pass in a color,
font size, and font weight. Looking at this implementation it is clear that the
Android team put a lot of effor into making a clear API that is easy to
understand.

A similar `TextStyle` can be applied to the `Handle` and `PostTime` functions.

```kotlin
@Composable
fun Handle(handle: String) {
    Text(
        text = handle,
        modifier = LayoutPadding(0.dp, 0.dp, 8.dp, 0.dp),
        style = TextStyle(
            color = Color.DarkGray,
            fontSize = 12.sp
        )
    )
}

@Composable
fun PostTime(time: String) {
    Text(
        text = time,
        style = TextStyle(
            color = Color.DarkGray,
            fontSize = 12.sp
        )
    )
}
```

Once the text styles are in place and the preview refreshes, things are looking
much better.

<img src="/assets/images/compose_1/styled_user_info_row.png" alt="Styled
user info row" width="700"/>

In this post I have shown how to setup a project to use Jetpack Compose and
display some text, as well as how to style the text and add some padding. In my
next few blog posts I will look at displaying images, handling click events, as
well as dealing with state in Compose. Stay tuned for more!

For more information there's a great [tutorial](https://developer.android.com/jetpack/compose/tutorial)
and [codelab](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics)
from Google that will walk you through some of the basics.
