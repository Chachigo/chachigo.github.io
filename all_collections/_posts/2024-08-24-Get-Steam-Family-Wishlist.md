---
layout: post
title: Get Steam Family Wishlist
date: 2024-08-24 21:30:00
categories: ["Dev","Python","Steam","Discord"]
---

## 🎯 Goal
the feature needs to display the game that are common to multiple family member's wishlists.  
the information to be listed are as follows:
- username of the users that have the game in common
- game name
- price
    - price per user
    - historical lowest price
- tell if the game is on sale

## ⚙️Process
 - ### Getting the wishlists
     To get the wish list unfortunately the Steam API documentation doesn't provide information to get the users wishlist since the extenssion [AugmentedSteam](https://augmentedsteam.com/) allows you to export a wishlist in JSON so it was obvious that it was possible using an http request so i've check in the debug information of the wishlist page and found i can use this: 
    > https://store.steampowered.com/wishlist/profiles/{SteamID}/wishlistdata/  
    
    the only small issue i encounter is that in order for this to work the game list of the users needs to be public.

    to get the common game beetween users i store the game id in a 2d python list
    formated like this : 
    > [[SteamAppId,[userId1,userId2]],  
    [SteamAppId2,[userId1,userId2]]]
    
 - ### Filter the "Global" Wishlist
    Since i only whant the game listed by multiple users i check the size of the user list for each gameID then for the information to be useful, the game must have the **"Family Sharing"** option so, for each game i'll need to check if it's the case using th Steam Store API like i've done to get the Family library. 
 
    Something that i have a bit of difficulties to do is only get the buyable games since my brother and i have a lot of unrelease indie game in our wishlist to be notify when the game is released and the only thing i found to filter these kind of game is the number of steam review since it's not possible to review an unreleased game.
 
 - ### Getting the Prices
    while i check if the game have the family sharing option and if it buyable i gather the price and divide it by the number of users that wants (and round it) then to get the lowest price of the game on steam, since it's not available throug the Steam API i uses the [IsThereAnyDeal API](https://docs.isthereanydeal.com/) 
    
    here is how i use it :  
        1. get the IsThereAnyDeal id from the steam appId using the **Game Lookup** method  
        2. get the lowest price using the **Store Low** method which return the lowest price with the IsThereAnyDeal id, a country (to get the price in the local curensy) and a store ID wich is 61  
        3. return the price as a string
    
    here is the python function i've made:

    ```python
    def get_lowest_price(steamid:int,) -> str:
        url = "https://api.isthereanydeal.com/games/lookup/v1?key="+api_key+"&appid="+str(steamid)
        answer = requests.get(url).json()
    
        gameid = answer["game"]["id"]
    
        data = [gameid]
        url = "https://api.isthereanydeal.com/games/storelow/v2?key="+api_key+"&country=FR&shops=61"
        answer = requests.post(url, json=data).json()
        return str(answer[0]["lows"][0]["price"]["amount"])
    ```
 
## ➕ Implementation of the feature to the bot
i've made another function to format the data from the API to a sendable discord message, After this the implementation is pretty straight forward since i've used the event functionality previously i just create an event function to make the request and format it which will run once a day.  
I don't like being notify randomly on discord so i've made the bot edit its message instead of sending a new one and the message look like this:

![discord Message](ttps://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/wishlistMessage.png?raw=true)
    