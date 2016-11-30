---
layout: post
title: Enhance Your iOS Bug Report With a Video
---

Your product has an iOS component and you found a bug. You're going to file it in your team's bug tracker so it gets fixed.

But it's a bit tricky to describe all the steps necessary to reproduce the bug -- what do you do? You could take screenshots at each step and include that in your writeup. That's a bit cumbersome and takes a while. A video would be easier. Here's how you do it:

On a Mac, you can [record the screen of your iDevice with QuickTime][ioshacker]. Here are the steps:

[ioshacker]: http://ioshacker.com/how-to/use-quicktime-record-screen-iphone-ipad-ipod-touch-running-ios-8

1. Connect the iDevice to your Mac with a Lightning cable, make sure the device is unlocked
2. Open QuickTime Player and select File -> New Movie Recording (or mash ⌥⌘N)
3. Next to the record button is a drop down icon, open it
4. In the menu, select your iDevice as the camera and the mic
5. Hit record and reproduce the bug on the iDevice

(Check out [iOSHacker][ioshacker] for more details and screenshots.)

The resulting movie file will probably be large. You can use QuickTime to make it smaller (File -> Export...).

You may also want to strip out the audio track. Now, you *could* record yourself narrating what you're doing, or perhaps elaborating in certain steps. In that case, leave the audio, and make sure to indicate in the bug report that the video has important details in the audio (I usually have my laptop muted).

But if the audio isn't important, stripping out will help decrease the file size. I couldn't find how to do this with QuickTime, but your Mac ships with a useful command-line utility which can do the job: [`avconvert`][avconvert]. Here's a handy command to shrink your video and strip out the audio track:

[avconvert]: https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/avconvert.1.html

```
avconvert \
    --preset PresetAppleM4V720pHD \
    --source /path/to/source.mov \
    --output /path/to/converted.mov \
    -ot audioTrack
```

This command uses the `PresetAppleM4V720pHD` preset -- you can list the available presets with `avconvert --listPresets`.

And you're done! You have a nice video demonstrating the bug to go along with your report.
