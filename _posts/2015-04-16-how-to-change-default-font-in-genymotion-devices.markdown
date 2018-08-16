---
layout: post
title:  "How to change default Font in Genymotion Devices (or in any rooted Android device)"
date:   2015-04-16 18:51:00
tags: android genymotion english system font tutorial
---
Today, while I surveyed, as all days, the "[genymotion](https://twitter.com/search?f=realtime&q=genymotion&src=typd)" results in Twitter, [Sergey Ryabov](https://twitter.com/colriot) asked this question: 

<blockquote class="twitter-tweet" data-lang="fr"><p lang="en" dir="ltr"><a href="https://twitter.com/Genymotion?ref_src=twsrc%5Etfw">@Genymotion</a> Consider a feature request. How about adding Custom Locale with Redacted [<a href="http://t.co/OlEatJYH87">http://t.co/OlEatJYH87</a>] font? Is it even possible?</p>&mdash; Sergey Ryabov (@colriot) <a href="https://twitter.com/colriot/status/588645473164140544?ref_src=twsrc%5Etfw">16 avril 2015</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<br />
As I answered that it could be possible, there is almost no chance to add it to a close backlog...

And by the way, why would he want to do that? Font is Font, Locale is Locale, there is no real point to change the Font when changing the Locale. Maybe for different languages with different symbols, graphemes or writing systems but not much more. Let's assume he just wanted to change the Font, using the Locale settings.

_What do we want here:_

- Showcase an app with private data
- Hide the text from the app as it is private using an alternative special font
- Don't change the app source code

The easiest way will be to use an emulated rooted device (so we can do what we want, break it and trash it after the shwocase). We will use [Genymotion](https://genymotion.com). As an example showcased application, we will use Twitter. So we start here:

{: .center}
<img src="{{ site.url }}/assets/images/genymotion_fonts_genymotion_twitter_base.png" alt="Twitter in Genymotion without modification" style="width: 300px; margin-left: auto; margin-right: auto"/>


Then we download the desired font. Here, the [Redacted Script Font](https://github.com/christiannaths/Redacted-Font) will hide the text behind a beautiful scratch.

We will now use the functionalities of [ADB](https://developer.android.com/studio/command-line/adb) (Android Debug Bridge) to communicate and modify the Android system fonts.

As [Genymotion](https://genymotion.com) devices are already rooted, we will just remount the system partition in write mode:

{% highlight bash %}
$ adb remount
remount succeeded
{% endhighlight %}

Then we will replace the default system fonts _Roboto_ (Normal, Light and Bold) by the Redacted ones.

In order to backup the existing font, we just rename them by adding _.old_ at the end:

{% highlight bash %}
$ for filename in `adb shell ls /system/fonts/Roboto*`; do `adb shell mv /system/fonts/$filename{,.old}`; done
{% endhighlight %}

We now copy all the Redacted Light font in the system folder, with the Roboto Light and Thin fonts names:

Light:
{% highlight bash %}
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-light.ttf /system/fonts/Roboto-Light.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-light.ttf /system/fonts/Roboto-LightItalic.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-light.ttf /system/fonts/Roboto-Thin.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-light.ttf /system/fonts/Roboto-ThinItalic.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-light.ttf /system/fonts/RobotoCondensed-Light.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-light.ttf /system/fonts/RobotoCondensed-LightItalic.ttf
{% endhighlight %}

Regular:
{% highlight bash %}
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/Roboto-Italic.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/Roboto-Medium.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/Roboto-MediumItalic.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/Roboto-Regular.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/RobotoCondensed-Italic.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/RobotoCondensed-Regular.ttf
{% endhighlight %}

Bold:
{% highlight bash %}
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-bold.ttf /system/fonts/Roboto-Black.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/Roboto-BlackItalic.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/Roboto-Bold.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/Roboto-BoldItalic.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/RobotoCondensed-Bold.ttf
$ adb push ~/Downloads/Redacted-Font-old-sources/fonts/redacted-script-regular.ttf /system/fonts/RobotoCondensed-BoldItalic.ttf
{% endhighlight %}

and then, let's restart the device

{% highlight bash %}
$ adb reboot
{% endhighlight %}

After the restart of the device, let's reopen our showcased application, and *Voila*:

{: .center}
<img src="{{ site.url }}/assets/images/genymotion_fonts_genymotion_twitter_redacted.png" alt="Twitter in Genymotion without modification" style="width: 300px; margin-left: auto; margin-right: auto"/>




