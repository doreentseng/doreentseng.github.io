---
layout: post
title: Add Fonts in React Native
date: 2018-06-30
excerpt: "Add custom fonts in react native app."
category: ReactNative
tags: 
- React
- ReactNative
- app
- font
comments: true
---

It's easy to add custom font in react native app.

### Step 1
Place the font file in the directory inside project. For example, mine is in `./assets/fonts/`.

### Step 2
Add the following line in your package.json:

```json
{
    "rnpm": {
        "assets": ["./assets/fonts"]
    }
}
```

### Step 3
Run the terminal:

```shell_session
react-native link
```

Congrats if you got the successful output! And now you can use the fonts both in IOS & Android.