# RCA — CleverTap In-App Video Plays Muted (cannot autoplay with sound)

## Summary
A CleverTap **custom-HTML in-app (interstitial)** campaign embeds an autoplaying HTML5 `<video>`.
On device the video autoplays **muted**, and it cannot be made to autoplay **with sound**
("unmuted by default") from the campaign HTML. Audio only starts after a manual user tap.

## Expected vs Actual
- **Expected:** video autoplays **with sound** (unmuted by default).
- **Actual:** video autoplays **muted**; sound plays only after a user gesture (tap).

## Root Cause
CleverTap renders custom-HTML in-app messages inside a native **WebView**
(Android `WebView` / iOS `WKWebView`). These WebViews enforce a "media requires user gesture"
autoplay policy **by default**:
- **Android:** `WebSettings.mediaPlaybackRequiresUserGesture` defaults to `true`.
- **iOS (WKWebView):** `mediaTypesRequiringUserActionForPlayback` defaults to requiring a gesture.

Under this policy the OS permits autoplay **only if the media is muted**; autoplay **with sound**
is blocked until a user gesture. This is enforced at the **native layer**, so:
- The HTML can set `video.muted = false`, but the WebView overrides it — the video either
  refuses to autoplay or plays muted.
- **No campaign-side HTML/CSS/JavaScript can override this** (OS security design).

## Evidence
- Template explicitly runs `v.muted = false; v.volume = 1; v.play()`. On device, the unmuted
  `play()` is **rejected**; falling back to `v.muted = true` is the only way the video autoplays.
- When the same video is started by a **manual tap** (user gesture), it plays **with sound** —
  confirming the blocker is the gesture requirement, not the video file or the template.

## Ruled out (does NOT fix it)
- Removing `muted` / setting `muted=false` in JS — WebView still blocks sound-on autoplay.
- YouTube embed — separate failure (Error 153) and YouTube also cannot autoplay with sound.
- Any change to the campaign HTML — cannot override a native WebView setting.

## Dev team confirmation (2026-05-27)
WIOM mobile dev team reviewed and confirmed this is **not app-fixable**:
- The in-app WebView is created **internally by the CleverTap SDK**; the app never touches it.
- `CleverTapInAppListener.kt` only receives lifecycle callbacks (show/dismiss/click) — **no WebView access**.
- The SDK exposes **no API** to set `mediaPlaybackRequiresUserGesture = false` on its in-app WebView.
- The OS rejects the unmuted `play()` **before** the campaign JS runs — no HTML/JS can override a native `WebSettings` flag.

**Conclusion: neither an app-side change nor a campaign-HTML change can fix this.**

## Fix — CleverTap SDK change (via CleverTap support ticket)
CleverTap must add one line in their SDK's in-app WebView initialization:

**Android**
```java
webView.getSettings().setMediaPlaybackRequiresUserGesture(false);
```

**iOS (WKWebView equivalent)**
```swift
config.allowsInlineMediaPlayback = true
config.mediaTypesRequiringUserActionForPlayback = []   // [] = no gesture required
```

Action: **raise a support ticket with CleverTap** requesting this change (ideally exposed as an
opt-in flag). Until CleverTap ships it, no app or campaign change will enable autoplay-with-sound.

## Interim (until the fix ships)
Campaign goes live with one of:
- **Autoplay muted** (sound turns on at first screen touch), or
- **Tap-to-play with sound** (poster + play button; one tap plays with audio).

Once the WebView flag is changed app-side, the existing HTML autoplays **with sound** with no
campaign changes.

## References
- Android `WebSettings.setMediaPlaybackRequiresUserGesture(boolean)`
- iOS `WKWebViewConfiguration.mediaTypesRequiringUserActionForPlayback`
- CleverTap custom HTML in-app notifications (developer docs)
