---
layout: post
title: Adding New Functionalities To FamilyBot
date: 2024-07-28 18:00:00
categories: ["Dev","Python","Steam","Discord","EpicGame"]
---

## 📜Context
In my previous post, I described how I made a bot to get notifications when one member of our Steam family purchases a new shared game. With this working fine (with a bit of tweaking), I was asked to add two functionalities:  
  - A command which returns the multiplayer games with a given number of copies in the shared library, which I chose to name !coop.
  - Notify the users of the new free Epic Game Store games.

## ➕ Adding New Commands
Previously, I had already added some commands for debug purposes which I send to the bot using DM. One to force the check if new games were added and another one to restart the web socket server since it was crashing and stopped renewing the token (it never crashed again since I added the command).  
The commands were simply implemented with the `on_message` method, but by trying to use it to add the !coop one, I discovered ***[discord.ext.commands](https://discordpy.readthedocs.io/en/stable/ext/commands/index.html)*** which is way easier and more reliable to use.  
After 2 hours without figuring out why it didn't work when I sent the command in a channel and it worked in DM, I double-checked the tutorial I used in the first place to set up my bot and figured out that I forgot to add the permission to read messages on servers... 

Now I've got a function that's called when the command is used.

## 👥 Adding the !coop command
With the first functionality of the bot, I already know that an array with the owners' IDs is in the answer of the Steam API request to get the family library. So, to get only the games with enough copies is pretty straightforward.  
Getting only the multiplayer games, on the other hand, is painful 🥲  
After checking the result of the API, there is no field that I can use to know if the game is multiplayer or not. So, to get it, I'm forced to use another API request, the one to get the Steam store page data and get these tags:   
 ![Steam store Tags](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/SteamTags.png?raw=true)  
Unfortunately, there are multiple tags that can be used, but after some trials, I found the ones I need, and it returns the correct game list. 

The last issue that I encountered is that, for example, when I used the command ***!coop 4*** it returns Portal 2, which is a 2-player co-op game (after some research, it can be played by up to 33 players with mods). So, I wanted to only return the games that can be played by the entered number of players/copies. Unfortunately, I didn't find a way to get the max amount of players for games. The only solution I found was to use ChatGPT, but I quickly dismissed this option because it was a paid one, and even if it was pretty accurate for some indie games, the results were incorrect.

## 🎁 Getting the Weekly Free Games from The Epic Game Store
Getting the free games is easy, and there is even a Python library to get them ***[epic-games-free](https://pypi.org/project/epic-games-free/)***. Unfortunately, the functionalities are pretty limited, and it returns all the free games and the games that will be free next week, but without filtering possibilities except the *"offerType"* to filter if this is a game, a DLC, or another offer.

So, I checked the sources of the library and found the API call that it does and used it to get the full answer and filter it myself to only get the offers that are currently active and that are games. Then, the function sends the URL to a specific channel on my Discord server.

```python
async def sendEpicFreeGames()
    epicUrl = "https://store-site-backend-static-ipv4.ak.epicgames.com/freeGamesPromotions?locale=fr&country=FR&allowCountries=FR" #the get parameters can be customized so the information gathered are in your language/currency
    baseShopURL = "https://store.epicgames.com/fr/p/"
        
    answer = requests.get(epicUrl)
    
    jsonanswer = json.loads(answer.text)
    
    gamelist = jsonanswer["data"]["Catalog"]["searchStore"]["elements"]
    for game in gamelist:
        if game["offerType"] == 'BASE_GAME' and game["price"]["totalPrice"]["discountPrice"] == 0:
            await epicGamesChannel.send(baseShopURL + game["urlSlug"])
```

The hardest part (before finding the easy solution) was to run this function every Thursday at 18:00 when the new offers are available without blocking the main script. After trying to use the ***schedule*** library and multithreading, I nearly gave up. Then, I remembered that when I was trying to add new commands, the discord.py library already had prebuilt functions to make it easier. So, I checked the documentation to see if there was something to schedule tasks, and there was exactly what I wanted **[discord.ext.tasks](https://discordpy.readthedocs.io/en/stable/ext/tasks/index.html)**. At this moment, I was about to explode (I lost 4 hours).

The implementation of this is really easy and looks like this for the `sendEpicFreeGames` function:

```python
@tasks.loop(seconds=60)
async def sendEpicFreeGames() -> None:
    now = datetime.now()
    if now.weekday() == 3 and now.hour == 18 and now.minute == 00:
        epicGamesChannel = client.get_channel(mydiscordChannelid)
        epicUrl = "https://store-site-backend-static-ipv4.ak.epicgames.com/freeGamesPromotions?locale=fr&country=FR&allowCountries=FR"
        baseShopURL = "https://store.epicgames.com/fr/p/"
            
        answer = requests.get(epicUrl)

        jsonanswer = json.loads(answer.text)

        gamelist = jsonanswer["data"]["Catalog"]["searchStore"]["elements"]
        await epicGamesChannel.send("les jeux Epic de la semaine sont:")
        for game in gamelist:
            if game["offerType"] == 'BASE_GAME' and game["price"]["totalPrice"]["discountPrice"] == 0:
                await epicGamesChannel.send(baseShopURL + game["urlSlug"])
```

Then in the `on_ready` I call `sendEpicFreeGames.start()` to start the loop.

Since I was using a `while True` loop for the main functionality, I just added `@tasks.loop(minutes=30.0)` before the declaration and a `sendNewGame.start()` in the `on_ready` event, and it makes it way cleaner.