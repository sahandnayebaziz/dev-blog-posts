---
title: "The End of Figurative"
slug: the-end-of-figurative
date: 2024-12-08
category: detailspro
link: https://figurative.design/release-notes/
---

Matías Martínez, via the blog for Figurative:

> I've decided to pull the plug on this project and removed the app from the App Store on November 25, 2024. This decision came just a few hours after reading this post by Wil Shipley on Mastodon: 
>
> > Amazon has shut off the feed that allowed Delicious Library to look up items, unfortunately limiting the app to what users already have (or enter manually).
I wasn’t contacted about this.
> >
> >I’ve pulled it from the Mac App Store and shut down the website so nobody accidentally buys a non-functional app.
>
> If you're reading this, you probably already know what a truly marvelous app Delicious Library was. Figurative, however, was constantly breaking. I realized it was time to stop—even as a free app, it wasn’t fair to you.

Figurative was an awesome iPad app that, in a way that I've never shared, played a big role in how I ended up making DetailsPro. 

It was the summer of 2020, I had just left Apple in May, and my tip-top plan was to make *Sketch for iPad*. That was it—all I wanted to build was an iPad app that could read and write Sketch files. I actually started on that plan and had a TestFlight out under the name "Layer Outer" which was quite short-lived because I ultimately tried out Figurative.

It wasn't until I tried out Figurative that I realized just how much Figma was eating Sketch's lunch. Right before my eyes was a simple, even flawed wrapper for Figma that was already miles ahead of what my *Sketch for iPad* could have ever been. Who needed Sketch for iPad when I already had a mostly-working *Figma for iPad (and Magic Keyboard)?*

So I abandoned Layer Outer. I was already having too much fun in Figurative and it wasn't even Figma's version of iPad Figma. 

After some time, I started to itch for wanting to put together SwiftUI views while I was away from my Mac and that's when I started working on DetailsPro.

Cheers to you Matías, for taking the awesome work of everyone at Figma and making it even more fun to use.

---

That was 2020. I'd be remiss if I didn't mention that I've had very similar feelings to then, this time while using Figma's Auto Layout features and their new UI3, which are awesome. 

I too often ponder when the natural end of the road will be for DetailsPro. In 2020, most were new to SwiftUI and hadn't tried it yet, so a tool that gave them an easy way to noodle around was welcomed. In 2024, I have many more requests for a design tool that can do everything Xcode can, which is not something that is possible to build, so I often feel I'm letting people down.

Plus, as Figma gets better with Auto Layout and official Apple UI Kits, maybe the draw of designing directly in SwiftUI lessens.

Much in the way Matías bumped up against limitations, I too run into things that can't be done simply due to the nature of SwiftUI. Things like how I can't let someone design in native iOS-style SwiftUI without the DetailsPro app running in Catalyst. Xcode can launch iOS simulators to get iOS proportions, but as a third-party developer, I have no such ability without leaning on Catalyst, which naturally means designing in native *Mac* proportions then goes out the window as a compromise. And we all know Catalyst isn't going to be around forever, so what I am to do? One day, suddenly the DetailsPro app on Mac switches to displaying everything with Mac-based magical UI values instead of the iOS ones that have been available to my users all this time?

It's not today that I feel the end is here, but it doesn't feel that far off. And much in the same way as Matías and Figurative, I know people will have had a blast using DetailsPro, but when do you know if the tool and the environment just don't jive anymore like they used to?

And, just to make things even funnier: at the end of 2024, Sketch announced they are starting to work on their own version of Auto Layout.


