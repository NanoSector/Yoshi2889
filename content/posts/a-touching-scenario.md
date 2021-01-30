---
title: "A Touching Scenario"
date: 2021-01-30T15:18:57+01:00
tags: ["ramblings", "project"]
draft: false
showtoc: true
description: "Mac touchpads are weird and cause issues when missing."
---

Recently I purchased a supposedly faulty A1342 MacBook6,1 ('09 13" white MacBook). The listing mentioned it "would no longer turn on" and the seller tried various things to get it back to life, to no avail. Let's diagnose and fix it!

# Diagnosis: First boot
Unboxing the laptop from its original box (what a find!), it indeed did not turn on. The SIL (Sleep Indicator Light)
would flash a few times, which [according to Apple's website](https://support.apple.com/en-us/HT203576) meant the battery 
was faulty. However, a faulty battery should normally not cause the machine to not turn on at all when plugged in to a
power source.

Very often, when computer don't pass POST (Power-On Self Test) and there is no sign of life beyond a flashing light, 
the RAM has gone bad. Such was the case in this laptop and removing one of the sticks caused it to turn on!

![First sign of life!](/img/slackbook/first-sign-of-life.jpg)

But this was only just the beginning.

## Diagnosis: Boot failures
While the machine looked like it was booting, it would not boot all the way to macOS and shut down about halfway through.

Looking around for potential reasons for boot failures I stumbled upon [this video by Louis Rossman talking about a 
malfunctioning touchpad disabling the screen](https://youtu.be/slJzVvJNxZs).

One-by-one I started unplugging various components inside the machine, starting with the touchpad, while booting it back
up after every change. I noticed the first change happened after unplugging the touchpad, verified by mixing the various
components:

- The fan would now spin at full speed. And it was _loud_.
- The machine no longer booted past a white screen without Apple logo. No matter the key combinations.

However, after also unplugging the wireless module, the machine would boot up fine, and even **fully boot to macOS**.

The following table summarises the various combinations:

| Combination         | Mac OS X 10.6 (DVD) | Fedora 32 (USB) | Windows 7 (DVD) |
|         ---         |        :---:        |      :---:      |      :---:      |
| Everything          |          x          |        ✓        |        x        |
| No touchpad         |          x          |        x        |        x        |
| No touchpad or wifi |          ✓          |        ✓        |        ✓        |

## Diagnosis: The birth of SlackBook
It was **slow**. Not to mention noisy. The installation of macOS took literal hours onto an SSD.
Booting the fresh installation took minutes.

In fact, it was so slow, that this machine was dubbed SlackBook, because it really was slacking.
At first, I blamed the DVD drive, however after installation it became apparent that the SSD was not much faster.
Mac OS X 10.6 should not take minutes to boot, especially not from an SSD and on a fresh installation.

Let's open Activity Monitor.

![Eating them cores](/img/slackbook/kernel-high-cpu.jpg)

`kernel_task` is a core process of Mac OS X that manages a lot of low-level tasks on behalf of the operating system's kernel.
[According to Apple](https://support.apple.com/en-us/HT207359) high `kernel_task` CPU usage might be caused due to high temperatures.
It simply prevents other processes from using this CPU power. 

On top of that, the CPU does run anywhere beyond 800 MHz. Combine this with practically cutting your CPU power in half,
and the slowness starts to make sense.

Why would this happen though? The system did not feel hot or even warm. The fan was spinning at full speed, so the metal
cover even felt cold to the touch.

Using [Macs Fan Control](https://crystalidea.com/macs-fan-control) we can get a look at the thermal sensors inside SlackBook.
These screenshots are taken after the machine has been fixed.

![Macs Fan Control before](/img/slackbook/mfc-before.jpeg)

Notice anything missing? There is no `Palm Rest` sensor. The logical conclusion would be that the touchpad provides this
sensor, which would make sense considering its location. The kernel likely assumes the machine is overheating whenever
one or more sensors are missing.

## Diagnosis: Unhappy Kernel
On top of the touchpad issues, the keyboard was intermittently working. The kernel is really unhappy about a missing
touchpad and seems to be resetting the driver responsible for it every second.

![The kernel really is not happy...](/img/slackbook/kernel-unhappy-touchpad.jpg)

Louis makes a point in the aformentioned video that the keyboard traffic is routed through the touchpad. Seeing as the
touchpad and keyboard are listed as `Apple Internal Keyboard / Trackpad` in System Information, but the keyboard is
intermittently working, I think this is not the case for this machine, but rather the keyboard is affected by the driver
resets as they share the same USB device. This is purely hypothetical though.

# The fix
The fix is simple. Replace the touchpad.

These touchpads are set up in a way that the surface and the actual circuitry behind it can be separated.
Therefore, the surface could potentially be reused if the circuitry in the cable went bad.
[In a guide by iFixit](https://www.ifixit.com/Guide/MacBook+Unibody+Model+A1342+Trackpad+Replacement/6338) you can see how
this works. Booting the machine with just the cable but no surface attached works fine.

Replacing the touchpad in this machine made the `Palm Rest` sensor show up again and the machine booted fine and was no longer
throttling.

![I can rest my palms again](/img/slackbook/mfc-after.jpeg)

For this machine the fix was a bit more nuanced though. Besides the touchpad issues, the machine also had blown speakers and
showed no audio output in any operating system. The logic board has some damage and a missing component around the audio circuitry,
likely caused by the bottom case scraping against the board while reassembling the system. One of the metal retention clips
on the bottom case is located nearby this section and could have caused this damage by not properly lining up the bottom case.
This likely caused a short or peak in voltage which in turn damaged the speakers.

![Damage to audio circuitry](/img/slackbook/board-audio-damage.jpg)

Moreover, the top case was shattered around the MagSafe port. The only solution here is to buy a new top case. The first
one I received had an outright broken keyboard, but the seller was extremely generous and shipped a replacement the day after,
which worked perfectly fine and even included a touchpad.

In the end, I replaced the following components:

- Top case with a keyboard and upward-facing speakers,
- Touchpad,
- Logic board,
- Battery,
- Rear main speaker.

![Parts](/img/slackbook/parts.jpg)

Needless to say, I paid more in parts than this machine would sell for. However, it was a really fun and interesting experience.

# The verdict
Is this caused by bad design? I don't think it is. This is the result of the machine detecting a fault and trying to protect itself
from further damage. The machine definitely _did not necessarily have_ to throttle the way it did, nor did the CPU protections in
`kernel_task` have to kick in while not having a touchpad.

The only point of concern here is the coupling of touchpad and keyboard into a single USB device. While untested, killing
the keyboard likely results in issues with the touchpad.

All in all, your machine has a fault that needs to be fixed for proper operation. Having protections in place is a good thing,
however ridiculous these might seem at a first glance.

![SlackBook in its final form](/img/slackbook/slackbook.jpg)