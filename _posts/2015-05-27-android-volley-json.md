---
layout: post
title: The best of both worlds
description: ""
modified: 2015-05-27
tags: [Android, Mobile]
categories: [Android]
---

I’ve been using [Volley](https://developer.android.com/training/volley/index.html) – Google’s HTTP library – for a while now . Working on a daily basis with JSON, I was missing a fantastic feature that – another great library – [Retrofit](http://square.github.io/retrofit/) has. The capability of parsing the JSON response directly to a java object. HTTP request goes in, Java parsed object comes out. As simples as it sounds.

So, I made two custom requests that try to achieve the retrofit feature using Volley and Gson “A Java library to convert JSON to Java objects and vice–versa”. For simplicity’s sake of the example application, both gson lib and volley lib were already included in the libs folder. One can get them on [https://github.com/google/gson](https://github.com/google/gson) and [https://android.googlesource.com/platform/frameworks/volley](https://android.googlesource.com/platform/frameworks/volley).

The application is pretty simple. One activity with two fragments in split screen. One of the fragments calls for a JSONObject, and the other one calls for a JSONArray. When the request is correctly parsed, a toString() puts the content of the parsed class inside a TextView.

The sample application can be found in [https://github.com/ivocosme/gsonenhancingvolley](https://github.com/ivocosme/gsonenhancingvolley)

Leaving AsyncTasks and other “lower level thread like” approaches was one of the best decisions that I could have taken as a developer. Code got cleaner, reduced complexity, no more “if != null”, among other things.