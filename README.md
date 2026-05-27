# wiom-inapp-video

Self-hosted video asset for WIOM CleverTap in-app (interstitial) campaigns.

## Asset
- `seva_stithi_full_v19.mp4` — Seva Stithi interstitial video (~2 MB)

## CDN URL (use this in the CleverTap `<video>` src)
```
https://cdn.jsdelivr.net/gh/Vikaswiom/wiom-inapp-video@main/seva_stithi_full_v19.mp4?playsinline=1
```

## In-app template
See `index.html` — autoplay **unmuted** on native Android/iOS, with a close (✕) button.
Paste into CleverTap → In-App Notification → Custom HTML, and tick **Include JavaScript**.
