---
layout: post
title: Activity? wat r u doin? Activity? Stahp!
description: ""
modified: 2015-05-28
tags: [Android, Mobile]
categories: [Android]
---

Every Android developer knows that you should not mess with the “finish” system of the activity lifecycle. You should just call finish() and let the OS take care of the rest for itself. However, every once in a while you get a request like “I don’t want to know how, but I want everything to just stop”. So, you drink a coup of courage, ignore all those guidelines that you learned and just look at the old java methods to shutdown an application – System.exit(0) (More info on [http://docs.oracle.com/javase/6/docs/api/java/lang/System.html#exit(int)](http://docs.oracle.com/javase/6/docs/api/java/lang/System.html#exit(int))).

The app actually shutdown BUT as I strangely discovered, it didn’t stay dead (lol). The app started again… After question all those java years in college, I found that the activity that was calling the method was NOT the only activity on the stack. It was on top, but it was not the only one.. I went from Activity A to Activity B without finishing it, so both were on the stack.

I’ve created a sample application that demonstrates this situation. There are two activities – ActivityOne and ActivityTwo. The first one calls the second and the second shows a dialog. When the user presses the button, the app closes and relaunches.

Consideration Notes:

Check for the “Logging…” tag on Logcat to see the activity creation.
If you just call the exit method without the dialog, the app will enter an infinite loop.
If you want to test the correct use case, remove the // from the finish() method on ActivityOne
So, boys and girls, this is an example of why you should not mess with the activity lifecycle. It exists, may not be perfect, but it works. Call the methods and let the OS take care of the rest.

The sample application can be downloaded from: [https://github.com/ivocosme/forcefinish](https://github.com/ivocosme/forcefinish)