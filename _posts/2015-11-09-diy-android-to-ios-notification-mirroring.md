---
layout: post
title: DIY Android-to-iOS notification mirroring
---

In one of my courses a few years ago, I met [someone](https://twitter.com/ImperialX92) with a mobile broadband (or data-only) plan on their phone. Now that idea was completely insane to me at the time. You actually can receive calls and texts with them, but you can't call out or text people back. Good luck explaining that to everyone who you gave your number to!

But the thing is, if you're a heavy data user, you're saving a lot of money[^valuedata] if you can make do without the calls or texts. So guess what I've been trying recently? Yep! So how'd I deal with no calling or texting out? Dual-wield. Done, right? Well, people need to call in, and if you're dual-wielding, you can't reasonably expect people dialing up your data SIM while you call them with another[^narrative]. So what if it was all kept to the secondary phone? Well I'm using an iPhone, and have a spare Android phone lying around. Time to put the droid to use! *(Spoiler: I didn't get call notifications working)*

There's plenty of notification mirroring products and services out there in the wild for both iOS and Android these days[^winpho]. For Android, the most well known at this point is probably [Pushbullet](https://www.pushbullet.com/)[^pushbullet], but sadly they only forward notifications to computers. I did some more research and came across [Klone](http://www.kloneapp.com/), which will forward to iOS, but:

> **Secure?**
>
> We do not store your notifications. All transmission from your phone to our server is via HTTP SSL which is the web standard for security. Once our server sends out the notification to Apple Push servers, we delete the notification immediately. All logs are also filter of any of your notification content.

Which is nice and all, but, well, SSL only provides security to data between my phone and their servers. I like not having my banking verification texts shuttled around in plaintext by anything other than the phone and the mirror. Sorry.

So what now? If you're enough of a power user, you've probably got [Tasker](http://tasker.dinglisch.net/)[^lolios]. You can find it on Google Play [here](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) for US$2.99 (currently AU$3.49). The best way I can think of describing this is like an IFTTT for your phone: you set up triggers which run tasks. Yep, we're rolling our own.

To start off, Tasker offers triggers under Application, Day, Event, Location, State and Time. We're interested in events, which are helpfully [documented here](http://tasker.dinglisch.net/userguide/en/help/eh_index.html) (Tasker just throws parameters at you when you're setting up events, so documentation really helps). Though there is a "Notification" event, I spent ages working with this and wondering why it wasn't working. As it turns out, it's [not a bug](http://www.androidcentral.com/tasker-advanced-use)[^wellwell]:

> Acting based on a notification used to be something that Tasker could do out of the box. It actually still has a native Notification event context, however as more and more apps have moved to the notifications API introduced in Android 4.3, they no longer work with it. Since AutoNotification can intercept notifications using the new API, Tasker's developer has chosen to let users rely on João Dias and this plugin rather than waste time fixing the native notifications context.

Aww. Alright, so from here we'll be needing to grab ourselves a copy of [AutoNotification](https://play.google.com/store/apps/details?id=com.joaomgcd.autonotification), which is free to download. Notification intercepting doesn't come for free, so you'll also need to unlock the full version via IAP, or interestingly via an ["unlock key" app](https://play.google.com/store/apps/details?id=com.joaomgcd.autonotification.unlock), for US$1.34 (currently AU$1.74).

After grabbing a copy and heading over to Tasker, you can go create a profile -> Event -> Plugin -> AutoNotification -> Intercept. That'll bring you to the Event Edit screen, which has a lone Configuration heading.

![Tasker AutoNotification Intercept event edit screen](/blog/images/tasker-autonotification-blank.png)

Tapping the pencil icon shunts you into AutoNotification's configuration. Coming out brings you back and shows you the configuration you've made, as well as some macros which will come in handy next. Most important are `%antitle` and `%antext`.

You'll then hit the Task Edit screen, where you can then add actions to run with the notification you've got. Originally, inspired by Seth Clifford's idea of using [Slack as a personal information centre](http://sethclifford.me/2015/09/using-slack-as-a-personal-information-center/), I was briefly considering seeing if I could make Tasker notify Slack. As it turns out, Pushbullet offers an action!

![Pushbullet action in the Tasker action plugins](/blog/images/tasker-pushbulletaction.png)

On top of that, they do [end-to-end encryption](https://blog.pushbullet.com/2015/08/11/end-to-end-encryption/)[^slackisstillcool], so they're looking like a pretty good option for this. The current Tasker action they provide lets you send a push with a title and a message. Remember the macros from earlier, `%antitle` and `%antext`? Drop them into Pushbullet and return to Tasker, and we're done!

To cap off with some sweet irony, you can head over to Pushbullet's mirroring options and send a test notification. That'll test out Pushbullet's own mirroring capabilities, but moments later you'll get the same notification everywhere you have Pushbullet installed. Like, say, an iPhone!

{% include retinaimg.html src="/blog/images/pushbullet-taskertest.png" alt="Pushbullet on iOS showing a forwarded notification" %}

Are we done? Well, I'm pretty happy with what I got, at least for notifications. I've swapped out that Android phone for another experiment though, which I might blog about later on. But here's two caveats.

Firstly, I didn't get phone call alerts coming through before I swapped out. Tasker provides a "Phone Ringing" event, which I tried to wire up to another Pushbullet event. For some reason, it wouldn't trigger, which I didn't get figured out. I did some cursory research, but it didn't seem to be a common enough issue. For the whole purpose of receiving phone calls in a secondary device, that's a pretty big blow to the usability of this.

Secondly, notifications get forwarded exactly as they are. If you're familiar with iOS, you know that multiple notifications from one app will just keep piling up and up and up, until your homescreen becomes a nice, giant list. Android helpfully groups messages, but on the other hand, that means things like this:

{% include retinaimg.html src="/blog/images/pushbullet-taskergroupedtext.png" alt="Pushbullet on iOS showing a forwarded grouped notification" %}

If you're cool with those and/or think you can figure out calls, this is definitely something you could try. That is, if you're somehow in the same rare situation of, "Hey, I want to see notifications from my Android phone on my iPhone/iPad/whatever else". But hey, that's why you're here, right? (Right?)

  [^valuedata]: Alternatively you can play the waiting game. For example, compare TPG's [2013 plans](https://web.archive.org/web/20130304221720/http://www.tpg.com.au/mobile/plans) with their [late-2015 plans](https://web.archive.org/web/20151107235410/https://www.tpg.com.au/mobile/plans). Or Yatango's [2013 prices](https://web.archive.org/web/20130409065404/https://www.yatangomobile.com.au/#pricingAnchor) with their [2015 prices](https://web.archive.org/web/20150121211959/https://yatango.com.au/mobile/pricing/personalised). Funnily enough I'm with them and they've [gone into administration](http://www.crn.com.au/News/410046,optus-reseller-yatango-mobile-australia-hits-the-wall.aspx).
  [^narrative]: I've changed around the narrative of the post multiple times (I've chosen not to write multiple posts, so that's why there's a lot in the footnotes). The last narrative focused around living data-only, and posed the problem that I didn't port out my number to my data SIM. If I did, that'd pretty much kill a whole chunk of issues *if and only if* the problem of calling/texting out was solved. I've bought some Skype credit before, and they helpfully provide Caller ID. Which would be great if it [actually worked all the time](https://www.google.com.au/search?q="Please+note+that+Skype+cannot+guarantee+that+caller+identification+will+always+be+displayed").
  [^winpho]: And not for Windows Phone, as of writing. Are there even APIs available to let you do something like that?
  [^pushbullet]: I like some of the things they've been doing recently like [end-to-end encryption](https://blog.pushbullet.com/2015/08/11/end-to-end-encryption/), which backs up their popularity with some great product decisions. I'll be interested to see how they monetise down the road though. Pushbullet notification mirroring also exists on iOS, and uses BLE to do it (though it supports Macs only). I quite like that model a lot more than the one with Android, though it definitely is more restrictive. That aside, this *is* a post about Android notification mirroring!
  [^lolios]: At which point you might be wondering why I'm not using Android. I really do appreciate that you can do a lot more with Android phones, but my continued use of iOS is more something that's stemmed out of my late-high school background in hobby web design/development. Platform wars are not for me :)
  [^wellwell]: At the time I actually didn't know this since I was doing this whole thing for fun, and switched straight to AutoNotification. But there you have it!
  [^slackisstillcool]: That's not a strike against Slack. After all, that's not something that makes sense for them. It would make it bit hypocritical to go for that option though, since I walked down this path to avoid my texts going around in plaintext.
