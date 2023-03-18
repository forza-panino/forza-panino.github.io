---
layout: write_up
title: Jailbroken
---

## Introduction

With this post we will be discussing the solutions for the challenge [Jailbroken](https://cyberdefenders.org/blueteam-ctf-challenges/35) on Cyberdefenders.

<h4 style="padding-left:18px; padding-top:12px"> Tools </h4>
<div style="padding-left:18px;" markdown="1" id="tools-list">
Suggested by the platform:

* [iLEAPP](https://github.com/abrignoni/iLEAPP)
* [Autopsy](https://www.sleuthkit.org/autopsy/)
* [mac_apt](https://github.com/ydkhatri/mac_apt)
* [SQLite Browser](https://sqlitebrowser.org/)

Suggested by me:

* [EpochConverter](https://www.epochconverter.com/) (online tool)

</div>

<br>

## Guide

Nothing interesting to know with the given scenario, we have a disk dump of an IOS phone to analyze.

First question:

    What is the IOS version of this device?

Open the output of **iLEAPP**, go to `Build Information` and you'll find out the IOS version to be <code class="flag">9.3.5</code>.

<br>

    Who is using the iPad? Include their first and last name. (Two words)

Open the output of **iLEAPP**, go to `Identity` and you'll find out the flag to be <code class="flag">Tim Apple</code>.

<br>

    When was the last time this device was 100% charged? Format: 01/01/2000 01:01:01 PM

In linux, inside the root folder of the extracted image run `find . -iname *battery*` and we can find the following path to `BatteryLife`: `./private/var/containers/Shared/SystemGroup/4212B332-3DD8-449B-81B8-DBB62BCD3423/Library/BatteryLife` -- inside we can find the file `CurrentPowerlog.PLSQL` that I opened with **SQLite Browser** (when you are inside SQLite Browser's file browser, remember to make it show **all** files). Select the table `PLBatteryAgent_EventBackward_Battery`, sort by descending `timestamp` and find the first time `level` equals 100, which is at timestamp `1586976031.21529`, copy-paste it to **EpochConverter** which will give you the flag: <code class="flag">04/15/2020 06:40:31 PM</code>.

<br>

    What is the title of the webpage that was viewed the most? (Three words)

Open the output of **iLEAPP**, go to `Safari Broswer - History`, sort by descending `Visit Count` and you'll find out the flag to be <code class="flag">kirby with legs</code>.

<br>

    What is the title of the first podcast that was downloaded?

Let's search for medias inside the phone. Go to `private/var/mobile/Media` and you'll find the `Podcasts` folder: inside there is a file called `StorePurchasesInfo.plist` -- I extracted some data on Linux with `cat StorePurchasesInfo.plist` to find out that the first listed podcast is `5782920041593012970.mp3`, I opened it and the title was <code class="flag">WHERE ARE WE?</code>.

<br>

    What is the name of the WiFi network this device connected to? (Two words)

Open the output of **iLEAPP**, go to `Network Usage (netusage) - Connections`, and you'll find out the flag to be <code class="flag">black lab</code>.

<br>

    What is the name of the skin/color scheme used for the game emulator? This should be a filename.

First, we have to understand what game emulator did the user download: open the output of **iLEAPP**, go to `Search Terms`, and after a little of scrolling you'll find out that the user downloaded `gba4ios`.
Now we have to find the app's folder. Again, within **iLEAPP**, go to `Application State DB` and search for `gba4ios`: you'll find out the bundle path of the app to be located at `/Applications/GBA4iOS.app` -- go there and you'll find the flag <code class="flag">Default.gbaskin</code>.

<br>

    How long did the News App run in the background?

Open again with **SQLite Broswer** `CurrentPowerlog.PLSQL`, but this time we will be looking at `PLAppTimeService_Aggregate_AppRunTime` table -- filter `BundleID` with the value `news` and you'll find the flag under `BackgroundTime`: <code class="flag">197.810275</code>.

<br>

    What was the first app download from AppStore? (Two words)

Open the output of **iLEAPP**, go to `Apps - Itunes Metadata`, and you'll find out the flag to be <code class="flag">Cookie run</code>.

<br>

    What app was used to jailbreak this device?

Open the output of **iLEAPP**, go to `Application State DB`, and after a little of scrolling you'll find out the flag to be <code class="flag">phoenix</code>.

<br>

    How many applications were installed from the app store?

Open the output of **iLEAPP**, go to `Apps - Itunes Metadata`, you can count <code class="flag">2</code> apps.

<br>

    How many save states were made for the emulator game that was most recently obtained?

First we have to know which is the most recently obtained game: open the output of **iLEAPP**, go to `Safari Broswer - History`, sort by descending `Visit Timestamp`, and you'll find out that game to be `Legend Of Zelda` -- now time to check how many save states were made.
In `private/var/mobile/Documents` we can find the folder `Save States`: open it, then go to the subfolder dedicated to the `Zelda` game and you can count <code class="flag">1</code> save state.

<br>

    What language is the user trying to learn?

Browser history did not help so I had to ask for a hint which suggested to "check podcast information" -- I opened the other podcast (`3408166032160890657.mp3`) inside `private/var/mobile/Media/Podcasts` and found out the user was listening to a podcast to learn the <code class="flag">Spanish</code> language.

<br>

    The user was reading a book in real life but used their IPad to record the page that they had left off on. What number was it?

We have to look for the camera's recordings. Go to `private/var/mobile/Media/DCIM/100APPLE` where you can find `IMG_0008.MOV`: open it to obtain the flag -- <code class="flag">85</code>.

<br>

    If you found me, what should I buy?

Again, had no idea what to do and asked for a hint which suggested to "check User notes" -- so the first step is to find the notes app's folder.
Open the output of **iLEAPP**, go to `Bundle ID by AppGroup & PluginKit IDs report`, and search for `note`: the path to check is `private/var/mobile/Containers/Shared/AppGroup/4466A521-8AF9-4E09-800B-C3203BB70E0E` (I just appended the `Directory GUID` to the given `Path`).
Inside that path there is `NoteStore.sqlite`: I open it with **SQLite Broswer**, but still, I couldn't obtain any useful data -- asked for another hint which suggested me to analyse the file with `mac_apt_artifacts_only.exe`.
I just executed `python mac_apt_artifact_only.py -i path/to/NoteStore.sqlite -o out/ NOTES` and opened the output with **SQLite Browser** within you can find the following entry:
```txt
Did you find me? Then you should Buy Crash Bandicoot Nitro-Fueled…
```
Thus, the flag to submit is <code class="flag">Crash Bandicoot Nitro-Fueled Racing</code>.

<br>

    There was an SMS app on this device's dock. Provide the name in bundle format: com.provider.appname

Open the output of **iLEAPP**, go to `Bundle ID by AppGroup & PluginKit IDs report`, and search for `sms`: the flag is under `Bundle ID` and it is <code class="flag">com.apple.MobileSMS</code>. Now last question!

<br>

    A reminder was made to get something, what was it?

It is clear that we have to check for some `calendar`/`reminding` app's data. Open **iLEAPP**, go to `Items` (under `Calendar`) and there you can find our final flag: <code class="flag">milk</code>.

<br>
<br>

We've reached the end of this challenge, and, consequently, of this guide as well! I hope it was useful and that you enjoyed it!