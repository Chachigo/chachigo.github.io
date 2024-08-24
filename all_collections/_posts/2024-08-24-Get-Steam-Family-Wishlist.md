---
layout: post
title: Get Steam Family Wishlist
date: 2024-08-24 21:30:00
categories: ["Dev","Python","Steam","Discord"]
---

## 🎯 Goal
The feature needs to display the games that are common to multiple family members' wishlists.  
The information to be listed is as follows:
- Username of the users that have the game in common
- Game name
- Price
    - Price per user
    - Historical lowest price
- Indicate if the game is on sale

## ⚙️Process
 - ### Getting the wishlists
     To retrieve the wishlists, unfortunately, the Steam API documentation doesn't provide information to get a user's wishlist. However, since the extension [AugmentedSteam](https://augmentedsteam.com/) allows you to export a wishlist in JSON, it was evident that it was possible using an HTTP request. I checked the debug information on the wishlist page and found that I could use this URL: 
    > https://store.steampowered.com/wishlist/profiles/{SteamID}/wishlistdata/  
    
    The only minor issue I encountered is that for this to work, the user's game list needs to be public.

    To get the common games between users, I store the game ID in a 2D Python list
    formatted like this: 
    > [[SteamAppId,[userId1,userId2]],  
    [SteamAppId2,[userId1,userId2]]]
    
 - ### Filter the "Global" Wishlist
    Since I only want the games listed by multiple users, I check the size of the user list for each game ID. Then, for the information to be useful, the game must have the **"Family Sharing"** option. So, for each game, I'll need to check if it's the case using the Steam Store API like I've done to get the Family library. 
 
    One difficulty I encountered is filtering for only buyable games since my brother and I have a lot of unreleased indie games on our wishlists to be notified when they are released. The only method I found to filter these types of games is by the number of Steam reviews since it's not possible to review an unreleased game.
 
 - ### Getting the Prices
    While I check if the game has the Family Sharing option and if it's buyable, I gather the price and divide it by the number of users who want it (and round it). To get the lowest price of the game on Steam, since it's not available through the Steam API, I use the [IsThereAnyDeal API](https://docs.isthereanydeal.com/).
    
    Here is how I use it:  
        1. Get the IsThereAnyDeal ID from the Steam App ID using the **Game Lookup** method  
        2. Get the lowest price using the **Store Low** method, which returns the lowest price with the IsThereAnyDeal ID, a country (to get the price in the local currency), and a store ID, which is 61  
        3. Return the price as a string
    
    Here is the Python function I've created:

    ```python
    def get_lowest_price(steamid: int) -> str:
        url = "https://api.isthereanydeal.com/games/lookup/v1?key="+api_key+"&appid="+str(steamid)
        answer = requests.get(url).json()
    
        gameid = answer["game"]["id"]
    
        data = [gameid]
        url = "https://api.isthereanydeal.com/games/storelow/v2?key="+api_key+"&country=FR&shops=61"
        answer = requests.post(url, json=data).json()
        return str(answer[0]["lows"][0]["price"]["amount"])
    ```
 
## ➕ Implementation of the feature in the bot
I've created another function to format the data from the API into a sendable Discord message. After this, the implementation is pretty straightforward. Since I've used the event functionality previously, I just created an event function to make the request and format it, which will run once a day.  
I don't like being notified randomly on Discord, so I've made the bot edit its message instead of sending a new one, and the message looks like this:

![discord Message](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/wishlistMessage.png?raw=true)
