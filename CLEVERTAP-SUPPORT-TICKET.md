# CleverTap Support Ticket — enable video autoplay-with-sound in custom HTML In-App WebView

**Subject:** Custom HTML In-App video cannot autoplay with sound — request to disable
`mediaPlaybackRequiresUserGesture` in the SDK's in-app WebView

**Account / App:** WIOM — <App ID / account name>
**SDK versions:** Android <fill> · iOS <fill>
**Platform(s) affected:** Android (and iOS) native in-app notifications

## Issue
We run a **Custom HTML In-App notification** (interstitial) containing an HTML5
`<video autoplay>`. On device the video plays **muted** and cannot autoplay **with sound**.
Audio only starts after a manual user tap.

## What we've verified
- The template explicitly runs `video.muted = false; video.volume = 1; video.play()`.
- The unmuted `play()` is **rejected by the OS** before our JS can take effect.
- Starting the same video via a **manual tap** plays it **with sound** — confirming the blocker
  is the WebView's user-gesture media policy, not our video file or HTML.
- Our mobile dev team confirmed the in-app WebView is created **internally by the CleverTap SDK**;
  the app has no access to it (`CleverTapInAppListener` only exposes show/dismiss/click callbacks),
  and the SDK exposes no API to change this setting.

## Root cause
The SDK's in-app WebView uses the default policy
`WebSettings.mediaPlaybackRequiresUserGesture = true` (Android) /
`mediaTypesRequiringUserActionForPlayback` requiring a gesture (iOS), which blocks
autoplay-with-sound at the native layer. This cannot be overridden from campaign HTML/JS or app code.

## Request
Please update the SDK's in-app WebView initialization to allow media autoplay with sound —
ideally exposed as an opt-in configuration flag:

**Android**
```java
webView.getSettings().setMediaPlaybackRequiresUserGesture(false);
```

**iOS (WKWebView)**
```swift
configuration.allowsInlineMediaPlayback = true
configuration.mediaTypesRequiringUserActionForPlayback = []
```

## Questions
1. Is there any existing SDK config/flag to enable autoplay-with-sound for custom HTML in-apps?
2. If not, can this be added? What is the expected timeline / SDK version?

## Use case / impact
Marketing interstitial video must play **with sound by default** to be effective. We currently
have to fall back to either autoplay-muted or tap-to-play, which reduces campaign impact.
