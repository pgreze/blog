---
title: "From Netlify to Firebase Hosting"
date: 2020-04-03T21:38:16+09:00
draft: true
---

## Ensure your Analytics is still working

[Google Analytics For Dummies](https://medium.com/firebase-developers/google-analytics-vs-firebase-analytics-vs-google-analytics-97ca645a8aff)

- Google Analytics, Classic Edition
- (??) Firebase Analytics, the “Google Analytics for Firebase, but everybody still calls it Firebase Analytics” Edition

The **Classic Edition** can be seen on the Google Analytics website or the [Android App](https://play.google.com/store/apps/details?id=com.google.android.apps.giant) (last update: November 2018...).
It's relying on a **Tracking ID** following the format UA-XXXXXXXX-X.

Getting Started guide: https://support.google.com/analytics/answer/1008015

**Firebase Analytics** can be seen on the Firebase console or the Google Analytics website.

# Redirect from Netlify

As expected from the well famous Netlify, there's a pretty easy way to redirect 
to your new site by adding few lines
[in your netlify.toml](https://docs.netlify.com/configure-builds/file-based-configuration/#redirects):

```
[[redirects]]
  from = "/*"
  to = "https://pgreze.dev"
  status = 301
  force = true
```
