---
title: Hello world!
redirect_from:
  - /guide.html
  - /guide-v1.html
---

For simple use cases, getting started with `ExoPlayer` consists of implementing
the following steps:

1. Add ExoPlayer as a dependency to your project.
1. Create a `SimpleExoPlayer` instance.
1. Attach the player to a view (for video output and user input).
1. Prepare the player with a `MediaSource` to play.
1. Release the player when done.

These steps are described in more detail below. For a complete example, refer to
`PlayerActivity` in the [main demo app][].

## Adding ExoPlayer as a dependency ##

### Add repositories ###

The first step to getting started is to make sure you have the Google and
JCenter repositories included in the `build.gradle` file in the root of your
project.

~~~
repositories {
    google()
    jcenter()
}
~~~
{: .language-gradle}

### Add ExoPlayer modules ###

Next add a dependency in the `build.gradle` file of your app module. The
following will add a dependency to the full ExoPlayer library:

~~~
implementation 'com.google.android.exoplayer:exoplayer:2.X.X'
~~~
{: .language-gradle}

where `2.X.X` is your preferred version (the latest version can be found by
consulting the [release notes][]).

As an alternative to the full library, you can depend on only the library
modules that you actually need. For example the following will add dependencies
on the Core, DASH and UI library modules, as might be required for an app that
plays DASH content:

~~~
implementation 'com.google.android.exoplayer:exoplayer-core:2.X.X'
implementation 'com.google.android.exoplayer:exoplayer-dash:2.X.X'
implementation 'com.google.android.exoplayer:exoplayer-ui:2.X.X'
~~~
{: .language-gradle}

The available library modules are listed below. Adding a dependency to the full
ExoPlayer library is equivalent to adding dependencies on all of the library
modules individually.

* `exoplayer-core`: Core functionality (required).
* `exoplayer-dash`: Support for DASH content.
* `exoplayer-hls`: Support for HLS content.
* `exoplayer-smoothstreaming`: Support for SmoothStreaming content.
* `exoplayer-ui`: UI components and resources for use with ExoPlayer.

In addition to library modules, ExoPlayer has multiple extension modules that
depend on external libraries to provide additional functionality. Browse the
[extensions directory][] and their individual READMEs for details.

### Turn on Java 8 support ###

If not enabled already, you need to turn on Java 8 support in all `build.gradle`
files depending on ExoPlayer, by adding the following to the `android` section:

~~~
compileOptions {
  targetCompatibility JavaVersion.VERSION_1_8
}
~~~
{: .language-gradle}

## Creating the player ##

You can create an `ExoPlayer` instance using `ExoPlayerFactory`. The factory
provides a range of methods for creating `ExoPlayer` instances with varying
levels of customization. For the vast majority of use cases one of the
`ExoPlayerFactory.newSimpleInstance` methods should be used. These methods
return `SimpleExoPlayer`, which extends `ExoPlayer` to add additional high level
player functionality. The code below is an example of creating a
`SimpleExoPlayer`.

~~~
SimpleExoPlayer player = ExoPlayerFactory.newSimpleInstance(context);
~~~
{: .language-java}

ExoPlayer instances must be accessed from a single application thread. For the
vast majority of cases this should be the application's main thread. Using the
application's main thread is a requirement when using ExoPlayer's UI components
or the IMA extension.

The thread on which an ExoPlayer instance must be accessed can be explicitly
specified by passing a `Looper` when creating the player. If no `Looper` is
specified, then the `Looper` of the thread that the player is created on is
used, or if that thread does not have a `Looper`, the `Looper` of the
application's main thread is used. In all cases the `Looper` of the thread from
which the player must be accessed can be queried using
`Player.getApplicationLooper`.

## Attaching the player to a view ##

The ExoPlayer library provides a `PlayerView`, which encapsulates a
`PlayerControlView`, a `SubtitleView`, and a `Surface` onto which video is
rendered. A `PlayerView` can be included in your application's layout xml.
Binding the player to the view is as simple as:

~~~
// Bind the player to the view.
playerView.setPlayer(player);
~~~
{: .language-java}

If you require fine-grained control over the player controls and the `Surface`
onto which video is rendered, you can set the player's target `SurfaceView`,
`TextureView`, `SurfaceHolder` or `Surface` directly using `SimpleExoPlayer`'s
`setVideoSurfaceView`, `setVideoTextureView`, `setVideoSurfaceHolder` and
`setVideoSurface` methods respectively. You can also use `PlayerControlView` as
a standalone component, or implement your own playback controls that interact
directly with the player. `SimpleExoPlayer`'s `setTextOutput` and `setId3Output`
methods can be used to receive caption and ID3 metadata during playback.

## Preparing the player ##

In ExoPlayer every piece of media is represented by a `MediaSource`. To play a
piece of media you must first create a corresponding `MediaSource` and then
pass this object to `ExoPlayer.prepare`. The ExoPlayer library provides
`MediaSource` implementations for DASH (`DashMediaSource`), SmoothStreaming
(`SsMediaSource`), HLS (`HlsMediaSource`) and regular media files
(`ProgressiveMediaSource`). The following code shows how to prepare the player
with a `MediaSource` suitable for playback of an MP4 file.

~~~
// Produces DataSource instances through which media data is loaded.
DataSource.Factory dataSourceFactory = new DefaultDataSourceFactory(context,
    Util.getUserAgent(context, "yourApplicationName"));
// This is the MediaSource representing the media to be played.
MediaSource videoSource = new ProgressiveMediaSource.Factory(dataSourceFactory)
    .createMediaSource(mp4VideoUri);
// Prepare the player with the source.
player.prepare(videoSource);
~~~
{: .language-java}

## Controlling the player ##

Once the player has been prepared, playback can be controlled by calling methods
on the player. For example `setPlayWhenReady` starts and pauses playback, the
various `seekTo` methods seek within the media,`setRepeatMode` controls if and
how media is looped, `setShuffleModeEnabled` controls playlist shuffling, and
`setPlaybackParameters` adjusts playback speed and pitch.

If the player is bound to a `PlayerView` or `PlayerControlView`, then user
interaction with these components will cause corresponding methods on the player
to be invoked.

## Releasing the player ##

It's important to release the player when it's no longer needed, so as to free
up limited resources such as video decoders for use by other applications. This
can be done by calling `ExoPlayer.release`.

[main demo app]: {{ site.release_v2 }}/demos/main/
[extensions directory]: {{ site.release_v2 }}/extensions/
[release notes]: {{ site.release_v2 }}/RELEASENOTES.md
