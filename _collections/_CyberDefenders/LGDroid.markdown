---
layout: write_up
title: LGDroid
---

## Introduction

With this post we will be discussing the solutions for the challenge [LGDroid](https://cyberdefenders.org/blueteam-ctf-challenges/69) on Cyberdefenders.

<h4 style="padding-left:18px; padding-top:12px"> Tools </h4>
<div style="padding-left:18px;" markdown="1" id="tools-list">
Suggested by the platform:

* [SQLite Browser](https://sqlitebrowser.org/)
* [EpochConverter](https://www.epochconverter.com/) (online tool)

Suggested by me:

* [SSIM-PyTorch](https://github.com/pranjaldatta/SSIM-PyTorch/blob/master/SSIM_notebook.ipynb)

</div>

<br>

## Guide

Nothing interesting to know with the given scenario, we have a disk dump of an Android phone to analyze.

First question:

    What is the email address of Zoe Washburne?

We have to find the email address of, presumably, one of the owner's contacts -- exploring the image dump we can find the `contacts3.db` file inside the `Agent Data` folder: open it with **SQLite Browser** and within the table `acquired_contacts` you'll find the flag which is <code class="flag">zoewash@0x42.null</code>.

<br>

    What was the device time in UTC at the time of acquisition? (hh:mm:ss)

Inside `Live Data` folder there is a file called `device_datetime_utc.txt`: there you can read the device time, <code class="flag">18:17:56</code>, which is already in `UTC` as per filename.

<br>

    What time was Tor Browser downloaded in UTC? (hh:mm:ss)

We have to look for a database of the downloads. Again, under the folder `Agent Data` you can find a file called `downloads.db`: open it with **SQLite Browser** and within the table `downloads` you'll find the timestamp `1619725346000` -- use **EpochConverter** to convert it to UTC and you'll get the flag <code class="flag">19:42:26</code>.

<br>

    What time did the phone charge to 100% after the last reset? (hh:mm:ss)

We have to look for `batterystats` -- on linux I used the command `find . -name "battery*"` which outputted me the path `Live Data/Dumpsys Data`. Open the `.txt` file within you'll find the following line:

```txt
          +5m01s459ms (3) 100 status=full charge=2665
```
In order to calculate the time, we need to know the starting point of the `+5m01s459ms` offset; if we look a few lines before we can read:

```txt
                    0 (10) RESET:TIME: 2021-05-21-13-12-19
```
Thus, `13:12:19 + 00:05:01` gives us the flag <code class="flag">13:17:20</code>. Next question!

<br>

    What is the password for the most recently connected WIFI access point?

The settings configuration is stored under `com.android.providers.settings`; however, in order to obtain access to this folder, we have to extract data from `adb-data.tar`. Under the `apps/com.android.providers.settings` there is a  file called `com.android.providers.settings.data` -- my editors had a few problems opening it, so, on linux I just opened the terminal, typed `cat com.android.providers.settings.data`, and found the following line:

```txt
<string name="PreSharedKey">&quot;ThinkingForest!&quot;</string>
```
Thus, the flag is <code class="flag">ThinkingForest!</code>.

<br>
    
    What app was the user focused on at 2021-05-20 14:13:27?

We have to look for usage stats. Under `Live Data` folder there is a file called `usage_stats.txt` where we can find:

```txt
    time="2021-05-20 14:13:27" type=MOVE_TO_FOREGROUND package=com.google.android.youtube class=com.google.android.apps.youtube.app.application.Shell_HomeActivity flags=0x0 
```
As we can read, <code class="flag">YouTube</code> was moved to foreground and was the app the user was focused on at that time.

<br>

    How much time did the suspect watch Youtube on 2021-05-20? (hh:mm:ss)

Using the previous file, we can check the time youtube was moved to background:

```txt
    time="2021-05-20 22:47:57" type=MOVE_TO_BACKGROUND package=com.google.android.youtube class=com.google.android.apps.youtube.app.watchwhile.WatchWhileActivity flags=0x0 
```
Thus, `22:47:57 - 14:13:27` gives us the flag, <code class="flag">08:34:30</code>. Now last question!

<br>

    "suspicious.jpg: What is the structural similarity metric for this image compared to a visually similar image taken with the mobile phone? (#.##).

If you don't know what is this question talking about, [here](https://en.wikipedia.org/wiki/Structural_similarity) you can find more info about structural similiarity metric -- you can decider either to implement the algorithm by yourself, or use to the tool [SSIM-PyTorch](https://github.com/pranjaldatta/SSIM-PyTorch/blob/master/SSIM_notebook.ipynb) (more explanation about it in the author's [article](https://medium.com/srm-mic/all-about-structural-similarity-index-ssim-theory-code-in-pytorch-6551b455541e)) that I found on internet!
Now let's continue with this question. If you noticed, within the downloaded zip there was a photo named `suspicious.jpg` -- we have to find a similiar photo inside the phone! Given that the question says that the photo was *taken with the mobile phone*, we will be checking `shared/0/DCIM/Camera` (remember that `shared` folder was extracted from `adb-data.tar`) where, in fact, we can find the file `20210429_151535.jpg` which is visually similiar. With the tool that I linked before you will be outputted `0.9973`, thus the final flag to submit is <code class="flag">0.99</code>!

<br>
<br>

We've reached the end of this challenge, and, consequently, of this guide as well! I hope it was useful and that you enjoyed it!