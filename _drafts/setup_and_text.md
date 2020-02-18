---
layout: post
title: "Becoming a Composer"
post-excerpt: "Dive into the world of composable views and learn how to setup
a project with Jeptack Compose and implement a copy-cat Tweet view."
---

# Becoming a Composer

[Jetpack Compose](https://developer.android.com/jetpack/compose) is a new
library from the Android team that simplifies UI creation. Small, composable
functions create the building blocks of your view. Combining these building
blocks forms higher-level views and your full UI.

This blog series focuses on Jetpack Compose by reacreating some well known
views from popular applications. My first copy-cat attempt is a Tweet view.
This example provides room to explore composable functions and manage state.

## Tweet view parts

![Insert a tweet view here]()

The view for a single tweet is small but it holds a lot of data.

1. User information
* Profile photo
* Display Name
* Twitter handle
2. Tweet information
* Content
* Time since posting
3. Actions
* Comment
* Retweet
* Like
* Share

For a composable app, these components are broken up into separate functions.
These functions should be small. Their only responsability is taking in basic
data to display and provide a component to do so.

Once these individual components are in place, they are combined together to
create the full view. Each post in this series will look at a different part of
the Tweet view. This first post focuses on setting up a project with Compose and
displaying the user's display name, Twitter handle, and Tweet time.

### Composable Tweet Strategy

TODO: Fill out this section

## Project setup

The first step to use Compose app is downloading the right version of Android
Studio. Version 4 and up has the required tools, but at this time it is still
in canary. Canary versions can be downloaded from
[the preview page](https://developer.android.com/studio/preview).

The only difference when creating a new project is the starter activity.
The 'Empty Compose Activity' provides the basic template for a composable
activity.

<img class="post-image" src="/assets/images/compose_1/empty_compose_activity.png" alt="Empty
compose activity selection"/>

The min SDK version must be at least 21 before creating the project.
Once the project is ready, building the app populates the preview pane.
If the preview pane is not open, clicking on the split button in the top-right
of the screen opens it on the right.

<img src="/assets/images/compose_1/split_button.png" alt="Code and
preview split button" class="post-image"/>

### Version updates

At the time of this writing, the version of the Jetpack Compose library provided
in the template project is behind the latest version. I did not run
into issues with this until trying to deal with vector image loading but
updating it now ensures I am using the current APIs.

At the time of this writing I update the versions to dev04 from dev02.

```kotlin
dependencies {
    ...
    implementation 'androidx.ui:ui-layout:0.1.0-dev04'
    implementation 'androidx.ui:ui-material:0.1.0-dev04'
    implementation 'androidx.ui:ui-tooling:0.1.0-dev04'
    ...
}
```

The compiler requires some additional compose options in the android block.
These options specify the Kotlin compiler version and extension versions so
everything works with compose.

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

Finally, deleting the bulk of the default code in `MainActivity` helps simplify things.

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

## Show some user data

The first composable functions I added are for the user information at the top
of the tweetn These are the user's display name, Twitter handle, and Tweet time.
Displaying text in compose involves creating a composable function
whose only responsibility is to diplay the text. Since there are three text
items, I add three functions for each item.

Note that these functions are not defined inside of the `MainActivity`
class. They should be usable from other classes as well. I just put them in the
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

Each function has the `@Compose` annotatioun so the system knows these are
composable functions. Each function has a `String` parameter for the text to
display.

With these low-level functions in place, I combine them into a higher-level
function representing the row of text.

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

To display items in a row with compose, you use the `Row` component. This
component accepts a lambda argument with each of the items it needs to
render.

### Checking progress in the preview

Building composable functions is quick and I want to be able to verify they look
correct before continuing. Adding the views to my `MainActivity` view is one
option but then I have to wait for the project to build and deploy each time I
want to change something. A better alternative is the `@Preview` annotation.

Applying this annotation to a composable function renders its content in
the preview pane in Android Studio. I use this to create a preview function to
view the progress of my tweet view. For now, it displays my current
`UserInfoRow`.

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

<img src="/assets/images/compose_1/user_info_row_view.png" alt="User info row
view displayed in the preview pane" class="post-image"/>

The `MaterialTheme` allows me to apply a theme to my composable functions if I
want but I am not using it right now. In a later blog post I'll take a look at
what the theme does and how to modify it.

While the row displays all of the text, it still looks pretty bad.
All of the text is the same size and there is no space between any of the
items. By default, the `Row` takes up as much space as all of the child
items and leaves no extra space for padding.

### Adding some space

Fortunately, adding space to the text is straightforward. I started by looking
at the `DisplayName` composable. There is a `modifier` parameter to
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

The `LayoutPadding` object has two constructors. The one I use above specifies
the padding for the left, top, right, and bottom of the view separately.
I only pass in a value for the right padding so I can add space
between the `DisplayName` and the `Handle`. The other constructor accepts a
single parameter and applies that value to all four areas around the view.

<img src="/assets/images/compose_1/padding_after_display_name.png" alt="Padding between
display name and handle composable views" class="post-image"/>

With that padding in place, I copy this padding to the `Handle` composable to
add space between the twitter handle and post time.

```kotlin
@Composable
fun Handle(handle: String) {
    Text(
        text = handle,
        modifier = LayoutPadding(0.dp, 0.dp, 8.dp, 0.dp)
    )
}
```
<img src="/assets/images/compose_1/user_info_row_spaced_out.png" alt="Padding
added between handle and post time composable views" class="post-image"/>

### Styling text

Last up for this post is styling the text to look more like the Tweet view. The
`DisplayName` should be bold with a black color to draw attention while the `Handle` and
`PostTime` should be a regular font style and a gray color. All of the text should also be
smaller. I'll start with the `DisplayName`.

To modify the style of `Text` there is a `style` parameter that accepts
a `TextStyle` object. The options on this class allow full control of how the
text is displayed.

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

For the `DisplayText` I need three of the attributes. I pass in a color,
font size, and font weight. This implementation makes clear that the
Android team put a lot of effor into making a clear API that is easy to
understand.

A similar `TextStyle` applied to the `Handle` and `PostTime` functions makes the
text look right.

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

Once the text styles are in place and the preview refreshes, things look
much better.

<img src="/assets/images/compose_1/styled_user_info_row.png" alt="Styled
user info row" class="post-image"/>

In this post I setup a project using Jetpack Compose and displayed some text,
as well as styled the text and added padding.
The next few blog posts focus on displaying images, handling click events, and
dealing with state in Compose. Stay tuned for more!

For more information there is a great [tutorial](https://developer.android.com/jetpack/compose/tutorial)
and [codelab](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics)
from Google that will walk you through some of the basics.
