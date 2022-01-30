---
layout: post
read_time: true
show_date: true
title:  Improving 3D print quality
date:   2022-01-29 10:00:00 +1000
description: What have a learned from the first 9 months of owning a 3D printer
img: posts/2022-01-29-3d-printer-first-year-lessons/hero.jpg
tags: [3dprinting,personal]
author: Peter Dodemont
---
This article will cover what I have learned while printing for the almost a year. In that time, I have printed some large multi-part pieces, as well as several "upgrades" for the printer itself. I finished off the [previous 3D printing article](\3d-printer-beginning.html) with the installation of the BLTouch, so it only seems fitting to start this article where I left off on the previous one.

## Measuring and Prototyping
One of the first things I printed once I had the BLTouch installed is a great example of how useful 3D printers can be. The print in question is a simple hook. The reason I wanted to print a hook, is that I have a number of cleaning rags that I wanted to hang up to dry out of sight in the laundry. I originally put in a towel rail, but that even the larger ones proved to be too small to dry all of them, meaning I'd have to put away any that were dry before I could hang the ones I just used. So, I figured hanging them on hooks would easier. I had a number of different size hooks lying around, but none of them fit properly. That is when I decided to 3D print one. I figured I was not the first person to ever want a hook, so I did a quick search for hooks on [Thingiverse](https://www.thingiverse.com/). One of the first few results was a [Customizable U-Hook](https://www.thingiverse.com/thing:1367661), this was exactly what I was after. I went through the instructions, and it covers off what applications you need and what setting change what part of the hook.
So, I went off and took all the measurements I needed, and full of confidence printed a hook. As you can imagine this did not end well. The hook didn't fit over the towel rail and was too chunky to be off use. So I went back and changed some settings and printed another one. And this one was a failure as well. This time the hook was too large and would almost fall off the towel rail. At the time I only had a limited amount of filament, so I was pretty annoyed at having wasted filament on 2 hooks. This is when I started looking into how to minimize filament waste when testing, which in turn led me to prototyping. I knew what prototyping was, but for some reason I did not think to apply it here. For those that might not know, as per [Wikipedia](https://en.wikipedia.org/wiki/Prototype) *"a prototype is an early sample, model, or release of a product built to test a concept or process"*. Because these are early or test models they are usually of a lower quality. In 3D printing you create prototypes by changing the layer height. Both my first 2 hooks were printed at a layer height of 0.1mm which is usually referred to as high quality prints. After that I changed the layer height to 0.4mm for the next few hooks. Which meant they used about 4 times less material and printer 4 times as fast (time wasn't a real issue for these hooks as they are small). At the same time, I also invested in a set of digital calipers so that I could take very precise measurements. I won't go into details on the calipers, other than to say that they are a must when you want to 3D print parts to need to fit over or on something.
After a few more attempts I got everything right and was able to print the hooks at the highest quality without having any of them go to waste. I even went from not having enough room, to having room to spare.
![Laundry Hooks](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/laundry.jpg "Laundry Hooks")
And here is a picture of my first hook on the left in yellow, with 2 of the "final" hooks next to it.
![Finished Hooks](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/hooks.jpg "Finished Hooks")

## Improving Print Quality - Benchmarking
After having done a few more prints, I noticed that the quality of the prints would vary even if I didn't change any settings. This was most visible when doing prints using different filament, but it would even happen when doing prints with the same filament. The first step in any process that wants to improve things, is getting an accurate and repeatable measurement. In the 3D printing world, the [#3DBenchy](https://www.3dbenchy.com/) is what is used most commonly as a benchmark. This "simple" looking boat has been designed to specifically to put 3D printers to the test. Not only that, but it also doesn't use a lot of material and prints quite quickly.
Below is a picture of 2 Benchys that were printed without any quality calibration and incorrect calibration. I don't remember which one is which, but you can see that there are quite a few imperfections on both.
![First Benchys](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/first-benchys.jpg "First Benchys")

## Improving Print Quality - Cooling
Now that I have a benchmark to use, it was time to start the process of improving the print quality. One of the first thing people online recommended to upgrade was the cooling. Improving the cooling will improve a few parts of the print, but most notably it improves overhangs and bridges. This is because with improved cooling you are cooling down the freshly extruded filament before it has time to zag or droop into the air gap that sits below it (like in the middle section of a bridge).
The stock cooling that comes with the printer is adequate, but you can get much better results by using cooling solutions that were designed by the community. Which brings us to another awesome thing about 3D printers, that you print new parts for your printer on your printer. After a bit of research, I settled on a the [CiiCooler](https://www.thingiverse.com/thing:2004629). There are other designs out there, but I choose this one as it provides 300-degree cooling field and allows for easy access to the nozzle should I need to access it (e.g., for cleaning). While I am able to print the cooler itself, I did need to buy some parts like the radial fan and some nuts and bolts. But these were all really cheap and probably all up cost less than a cup of coffee. Once I had all the parts in hand, I was able to install it all.
This is what it looks like assembled but not mounted
![CiiCooler Assembled](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/ciicooler-assembled.jpg "CiiCooler Assembled")
And here is what it looks like mounted
![CiiCooler Mounted](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/ciicooler-mounted.jpg "CiiCooler Mounted")
You might have noticed the mounted one looks shinier than the assembled one, this is because after having done all the upgrades and calibration I went back and reprinted it to get a higher quality one with less defects.
I did not have any issues with mounting, I did however run into problems printing when I first installed it. Below are a picture and a slow-motion video showing the issues. Basically, the filament is lifting up and after a certain time the nozzle just runs straight into the filament.
![CiiCooler Failed Prints](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/ciicooler-failed-prints.jpg "CiiCooler Failed Prints")
{% include youtubeplayer.html id="nIPIKR_mSiI" %}
The issue was that the filament was printed too hot causing it to lift rather than stay in place. This didn't make much sense to me as this new cooling solution was supposed to be better. After numerous attempts to fix it (as you can see in the background of the picture). I went all the way back to the beginning and discovered that the fan wasn't actually spinning. My first thought was maybe the fan is faulty, so let me swap it for another (the fan came in a 3-pack when I bought it, so I had it lying around anyway). Same thing, the fan would not spin. It was extremely unlikely  both fans would be faulty, so I thought maybe I damaged the board or connector. I got the old fan back out and plugged it in, it started spinning no issues. While then unplugging the old fan I noticed that the wires weren't the same on both, so I swapped the wired on the new fan and the problem was solved. Now that we have a working cooling solution let's have a look at some before and after pictures of Benchy.
The first one is the overhang. This looks pretty good with both the stock and CiiCooler.
Stock
![Stock Overhang](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/stock-overhang.jpg "Stock Overhang")
CiiCooler
![CiiCooler Overhang](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/ciicooler-overhang.jpg "CiiCooler Overhang")
Then, we have a picture of a bridge (as in the arch on the door of a Benchy). This has some pretty rough edges to it when printed with the stock cooler, but with the CiiCooler it's significantly smoother, with only very minor imperfections.
Stock
![Stock Bridge](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/stock-bridge.jpg "Stock Bridge")
CiiCooler
![CiiCooler Bridge](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/ciicooler-bridge.jpg "CiiCooler Bridge")
Finally, we have a more extreme bridge, in the form of the ceiling on the inside of the Benchy. While it is hard to see, there is quite a bit of drooping on the stock cooler. With the CiiCooler, it again looks very clean with minimal if any drooping.
Stock
![Stock Ceiling](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/stock-ceiling.jpg "Stock Ceiling")
CiiCooler
![CiiCooler Ceiling](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/ciicooler-ceiling.jpg "CiiCooler Ceiling")

## Improving Print Quality - Print Temperature
Getting the print temperature right is probably the most important setting for overall quality improvements as the temperature can impact numerous different parts, from not being able to extrude enough material, to extruding too much, to sagging or lifting, to prints not sticking to the bed ... the list goes on and on. So, you might wonder why it is not first in the list. This is simply because I didn't really pay attention to it until after I had the issue with the cooling. It was never really an issue, which means it is a lot more forgiving. Even if you get it wrong, your print won't be ruined.
A very important point to realize, is that the ideal print temperature will vary between filaments. So, calibrating the print temperature should be repeated for each new type and brand of filament you get.
The best way I found to do that calibration is a "temperature" tower. With a temperature tower, you generally print the same pattern over and over stacked on top of each other and very the temperature at each new module of the stack. I first came across them while looking into the issue I had with the cooler but was not able to print any successfully as the problem was not the printing temperature. Once I had fixed my cooling issues, I got back to the temperature tower to ensure I was printing at the right temperature.
I settled on the [Smart compact temperature calibration tower](https://www.thingiverse.com/thing:2729076). This tower has a number of other tests included, like overhangs and bridges, that will help determine the ideal printing temperature.
Below is a picture of 6 temperature towers I printed for different filaments I have. These were all printer with the exact same settings.
![Temp Towers](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/temp-towers.jpg "Temp Towers")
And as you can tell the different can be significant. Some are clearly stringier and most struggle with very steep overhangs.
The 2 "failed" towers are from the same rainbow filament and they both failed for different reasons. The left most one got detached from the print plate and the one next to it, the gear that pushes the filament out got loose so it stopped extruding all together.

A quick note that to have the temperature change at each section you need to configure specific settings in your slicing software. The process for having the temperature changes happen vary depending on your slicer, but that is nothing a quick search can't help you with.

## Improving Print Quality - Stringing
The next quality improvement I will talk about will be reducing stringing. Stringing happens when you need to move the print head from one section on the print to another without extruding any material. When this happens some fine (or sometimes less fine) strands of material stay connected to the first part and drag across to the 2nd part. It's probably easier to understand when looking at a picture, so here is one of some really bad stringing
![Stringing](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/stringing.jpg "Stringing")
As with most things on 3D printers there are numerous settings to play around with that can impact stringing. But the 2 main ones are retraction distance and retraction speed. First let me explain what retraction is. Enabling retraction means the printer will pull the filament back up a little bit when it needs to make one of the aforementioned moves. This doesn't pull back any of the molten material, but it reduces the pressure so that it doesn't get pushed out as fast anymore. Retraction distance as you might have guessed is how much filament is retracted and retraction speed is how fast this happens.
Distance is probably self-explanatory, but speed matters because a higher speed causes a "cleaner" break. Molten filament is quite stretchy, so if you pull it slowly it will just stretch out rather than break.
For my stringing test I used [this model](https://www.thingiverse.com/thing:909901) from Thingiverse. It uses minimal amount of filament and prints very quickly. So, I started by leaving one of the settings at the default value and then set the other to 0. I would then print the test, go back change that one setting from 0 up in increments (0.5mm for distance and 10 mm/s for speed). If the new print looks better than the old increment again, if the new print looks worse you know the ideal sits somewhere in between. So, half your increment and keep halving it until you get your best possible result. This will take quite a few prints, but at the small size they are it's worth it. I did about 20 before getting it dialed in. I also recommend writing down the settings you used on the test prints as it will become hard to remember and differentiate them otherwise.
![All The Stringing Prints](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/stringing-all.jpg "All The Stringing Prints")
A final note to remember is that this is a torture test, so it is very likely you will not be able to get rid of stringing altogether on these tests. As an example, see the image below of my best result. There is sting quite a bit of stringing, but it is extremely fine. And I have barely encountered any stringing on non-torture test prints since I have done the calibration.
![Best Stringing Test](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/stringing-best.jpg "Best Stringing Test")
Finally let's take a look at our 3D Benchy again and we can see significant stringing before the calibration and hardly any after. And whatever stringing there is, can easily be removed as it is very fine.
Before
![Benchy Before Stringing Calibration](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/stringing-benchy-before.jpg "Benchy Before Stringing Calibration")
After
![Benchy After Stringing Calibration](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/strining-benchy-after.jpg "Benchy After Stringing Calibration")

## Improving Print Quality - Layer Height and The Magic Number
The final item to cover off is the layer height and something referred to as the magic number. The magic number refers to the optimal layer heights for printer based off a number of factors (e.g., motor step angle and leadscrew pitch). And while you can calculate all of this yourself, it is probably easier to do a search for the magic number of your printer model as someone will probably have already worked it out. If you know all the required details, you can also use the "Optimal layer height for your Z axis" [calculator](https://blog.prusaprinters.org/calculator_3416/) on the Prusa Printers website.
Once you have the magic number your optimal layer height is any that are multiple of that number.
I only came across the reference to this magic number quite late, when I was looking into removing the zits or blobs from the surface of my Benchys. These zits didn't really appear in my other prints, just on the Benchys, so I didn't put any effort into getting rid of them initially.
For my printer the magic number is 0.04mm so that means that my previous prints at 0.1mm weren't a multiple of the magic number so not ideal. Changing to 0.2mm actually improved the quality of my prints and got rid of the zits and blobs. You can really only tell the difference between the layer heights at very close range.
Benchy at 0.1mm
![0.1mm Layer Height Benchy](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/0.1mm-benchy.jpg "0.1mm Layer Height Benchy")
Benchy at 0.2mm
![0.2mm Layer Height Benchy](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/0.2mm-benchy.jpg "0.2mm Layer Height Benchy")

That about covers what I have learned in my time using my 3D printer so far. As always if you have any question or comments, feel free to reach out.
I'll leave you with my favorite printed item to date. A 700% scale LEGO minifig. Or would that be a maxifig now?
![Maxifig](/assets/img/posts/2022-01-29-3d-printer-first-year-lessons/minifig.jpg "Maxifig")