---
id: 720
title: 'Ephing Power Pool'
date: 2020-05-21
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=720
permalink: /EphingPowerPool/
categories:
    - PowerShell
---

# Introducing the Ephing Power Pool

Let's travel back in time 20 years to February 2020 in the Ephing kitchen. Ephing Momma and EphingAdmin were having a discussion:

> EphingMomma: We should get a pool this summer!
> 
> EphingAdmin: That sounds like a great idea!
>
> EphingMomma: Looks like theres a few options at Target and Lowes
>
> EphingAdmin: Yeah, let's wait until summer to get one. Not like anything big will happen between now and then to close everything down and make everyone run out and buy them.

![WahWah](https://media.giphy.com/media/w3QsOYBlbAURq/giphy.gif)

So now, it's almost summer and we start looking for a pool!

Look at Target...

![Nothing](https://media.giphy.com/media/baPIkfAo0Iv5K/giphy.gif)

Look at every Target within 100 miles (I'm in Minnesota, there's a Lot)

![Nothing](https://media.giphy.com/media/kzxOVNpKLWDyL9tTTn/giphy.gif)

Look online...

![Nothing](https://media.giphy.com/media/QZOaeparxsNOfKWbER/giphy.gif)

My wife and I start watching Target's stock, Wednesday we see limited availability at a Target 30 minutes away, I race there!

![Nothing](https://media.giphy.com/media/h8UQPAvp7LOUJJkhac/giphy.gif)

Then this morning I get a text from my wife:

"It shows in stock at Target, GO GO GO!"

I race to Target, get there, run to the Pools and...

![Nothing](https://media.giphy.com/media/JoJGxeheao5mQaSiBK/giphy.gif)

I go home, dejected, and decide to check some links I have saved from Lowes. Bam, in stock AND order online. Surely this can't fail! I click add to cart, checkout,

"There's a problem with availability, please choose a different delivery option."

Ok, no problem, I check - all delivery options are grayed out!

I go back to the product page and... All sold out!

![Damnit](https://media.giphy.com/media/3o7TKA3ypeMbOXSrp6/giphy.gif)

So, I click refresh just to see what happens, and it's in stock!

![InStock](https://media.giphy.com/media/5VKbvrjxpVJCM/giphy.gif)

I go to click add to cart, and it gets grayed out and shows out of stock!

![WhatIsGoingOn](https://media.giphy.com/media/5fcc4PADD7ax2/giphy.gif)

What - The - Hell

I hit refresh, same thing happens. Shows in stock, but then is out of stock in a few seconds.

So that got me thinking, what if they load the page and then call an API to check the stock...

To developer mode!

I go to [the Lowes pool link](https://www.lowes.com/pd/Intex-24-ft-x-12-ft-x-52-in-Rectangle-Above-Ground-Pool/1002623858) let it load, then hit F12 on my browser, go to the network tab, and hit Refresh! 

This now shows me all the extra content the page is loading in the background, which I hope shows me whatever is updating the In Stock button...

![F12](https://www.ephingadmin.com/images/2020/LowesInitialF12.jpg)

I then start scrolling through them! To help myself narrow this down, I decide to first look at lowes.com things (lowesCDN sounds like places they store images and static files, and the other locations all look like ads) AND look for json responses - like these:

![Look Promising](https://www.ephingadmin.com/images/2020/Lowes-TheseLookPromising.jpg)

If you see in the image above, I clicked on the "Response" tab on the right so I can see the JSON output. That link doesn't look great, but what about that one that says Guest?

Click on that and you see some great json!

![Lowes-GreatJSON](https://www.ephingadmin.com/images/2020/Lowes-GreatJson.jpg)

Now, this isn't exactly what we are looking for (how to tell if it's in stock) but there's a lot of product details here. If I scroll down...

![Lowes-ItemInventory](https://www.ephingadmin.com/images/2020/Lowes-ItemInventory.jpg)

A json object called ```ItemInventory``` with a ```totalAvailableQty``` property! This is exactly what we need.  If you click back to the Headers tab on the right, you can get the link that produced this JSON:

> https://www.lowes.com/pd/1002623858/productdetail/1955/Guest

Looking at this link, and the data in the JSON blob, I believe this is how the link is organized:

``` https://www.lowes.com/pd/<productId>/productdetail/<storeId>/Guest ```

So, this link is specific to my product and my store, which is really all I need.  Now, to PowerShell!

``` PowerShell
$Quantity = 0
while($Quantity -eq 0) {
    $LowesData = Invoke-RestMethod -Uri 'https://www.lowes.com/pd/1002623858/productdetail/1955/Guest' -Method Get
    $LowesData.inventory.totalAvailableQty
    $Quantity = $LowesData.inventory.totalAvailableQty
    if($Quantity -eq $null) { $Quantity = 0 }
    if($Quantity -eq 0){
        Start-Sleep -Seconds 60
    }
}

```

I wrote this simple script to check every 60 seconds if the pool is in stock. Line 4 will output the current quantity (sanity check) and Line 5 & 6 will switch Quantity to 0 if it's $null, meaning if there was no data returned.

So this script will check if the item is in stock, it's going to run on my home machine, how do I make it really let me know when it's in stock?! Since I work from home right next to this computer, I decided to simply have PowerShell tell me it's in stock!

``` PowerShell
Start 'https://www.lowes.com/pd/Intex-24-ft-x-12-ft-x-52-in-Rectangle-Above-Ground-Pool/1002623858'
while($true) {
    $null = (New-Object –ComObject SAPI.SPVoice).Speak("It is in stock")
    Start-Sleep -Seconds 5
}
```

First I have it open the Pool link in my preferred browser, then I have it yell at me every 5 seconds It is in stock.

It took about 4 hours, but eventually my computer started yelling it is in stock! I ordered the pool and well be in the new Ephing Power Pool in a week or two!
