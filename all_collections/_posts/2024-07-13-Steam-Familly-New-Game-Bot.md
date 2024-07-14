---
layout: post
title: Steam Familly New Game Bot
date: 2024-07-13 21:00:00
categories: ["Dev"]
---

## 📜Context
Since March 2024, Steam has released its new family functionality, which allows multiple users to share their game library. Unlike the first family mode they released, the new one allows users to play shared games even when the game owner is connected.

## 🔍 My requirement
With my family, we now have hundreds of games, and it's difficult to know when someone buys a new one. Therefore, I would like to be notified when a new game is added to the game library. I spoke to other members, and it's something they would also appreciate.

## 🚀 Start of the Project
Since it needs to notify all family members, a Discord bot seemed obvious to me. However, I knew this wasn't the difficult part. First, I needed to gather the shared library using an API or similar method.

- ### 📊 Getting the Shared Library
  First, I tried to find an API call on the Steam family page using debug mode, but it was too messy, so I gave up quickly.

  I looked up the [Steam API official documentation](https://developer.valvesoftware.com/wiki/Steam_Web_API), but there was nothing related to family sharing. Since this feature is still in beta, this is understandable. I also considered privacy concerns; it seems reasonable that not anyone can access family data.

  Next, I found [this API documentation](https://steamapi.xpaw.me/) by the person behind [Steam DB](https://steamdb.info/). I tried it with the necessary token, and the request [IFamilyGroupsService/GetSharedLibraryApps](https://steamapi.xpaw.me/#IFamilyGroupsService/GetSharedLibraryApps) was exactly what I needed.

  The required information includes the family_groupid, easily obtainable using the [GetFamilyGroupForUser](https://steamapi.xpaw.me/#IFamilyGroupsService/GetFamilyGroupForUser) request, and an Access_Key, which you can find by visiting this link <https://store.steampowered.com/pointssummary/ajaxgetasyncconfig> with a browser already connected to Steam. However, this key changes frequently.

  At this point, obtaining the access key was not my priority, so I focused on testing if the project was feasible by retrieving the games and detecting new additions.
  
- ### 🎮 Getting the New Games
  Initially, I considered creating a database to store the games but opted for simplicity by using a text file. Each time I retrieve the games, I compare them with those in the file to find any new additions. Since I mainly work with Python (still a beginner), here's how I did it:
  
  ```python
  gamelistFile = open("/file/", "r")
  gameList = gamelistFile.readlines()
  if len(gameList) < len(APIGameList):
      newGame = set(APIGameList) - set(gameList)
      print(newGame)
  else:
      print("No New Game")
  ```
  
  Since the goal is to create a Discord bot, I also collect the SteamID of the game owner to thank them in the Discord notification.

- ### 🤖 Setting Up the Discord Bot
  Setting up a Discord bot for the first time, I followed [this tutorial](https://www.docstring.fr/blog/creer-un-bot-discord-avec-python/). I connected it to my Discord server and made it send a simple message with a thank-you note and a Steam shop link, similar to this:
  
  ![Discord Message Example](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/DiscordMessage.png?raw=true)

## 🔑 Getting the Access Token
This was the most challenging part of the project. I tried multiple Python libraries but found none with this functionality that could log in with my account each time due to my Steam Guard setup.

Ultimately, I used Selenium to run a Firefox profile already connected to my Steam account to access <https://store.steampowered.com/pointssummary/ajaxgetasyncconfig> and obtain the token.

I wanted to run the bot on my NAS server, which lacks a browser. Hence, I realized it wouldn't work solely on my NAS. Therefore, the part running on my computer needs to send the token to the script on my NAS.

- ### ✉️ Sending the Access Token
  To send the token to the server, I used WebSockets. Although I was not familiar with it, it was the first solution that came to mind to solve this problem. I considered MQTT later but couldn't get my test scripts to work, so I stuck with WebSockets.

  The main issue was that the WebSocket server was blocking. After trying various methods (like multithreading), I implemented a quick and dirty solution: the Discord bot starts and runs another Python script in parallel with the WebSocket server. The WebSocket server writes the token to a file when received. Therefore, when the bot retrieves the Steam library, it reads its token value from the file and makes the request.

- ### 🕒 Get the Token When It Expires
  Initially, I assumed the token had a 24-hour lifetime. Since I use my computer daily, I only needed to obtain a new token after my computer started. However, it's not always 24 hours, and once generated, it remains the same until it expires.

  To regenerate the token, I needed its expiration date. On <https://steamapi.xpaw.me/>, this information is displayed. Initially, I tried an API call named **GetTokenDetails**, which requires only the token and consistently returned "Access Denied."

  I looked at how xPaw displayed it and found that the token (mostly) was a JSON in base64. Here's the TypeScript code managing the access token, which I converted to Python to get the expiration timestamp:
  
  ```python
  while True:
      exptimeFile = open("./tokenExp", "r")
      exptime = exptimeFile.readline()
      exptimeFile.close()

      runtime = datetime.fromtimestamp(float(exptime) + 60)
      print(runtime)
      now = datetime.now()
      if now > runtime:
          print("Target time has already passed. Running immediately...")
          asyncio.run(getToken())
      else:
          wait_seconds = (runtime - now).total_seconds()
          print(f"Waiting until {runtime.strftime('%H:%M')} to start. That's {wait_seconds} seconds.")
          time.sleep(wait_seconds)
          asyncio.run(getToken())
  ```

## ✨ Perfecting the Script
To make troubleshooting easier, I added logs using a logger. It was my first time using it, and it's straightforward.  
I also added the feature of being warned by a private message when the bot tries to retrieve games when the token has expired.  
Since in the request URL the **_include_free=false_** was not doing a great job to remove the free games, I found out that in the answer there was the field **_exclude_reason_** that equals 3 when the game is free, so I didn't add those games to the list.

## 🔧 Setting Up
Since my NAS is a Synology, I needed to install Python and pip on it, which was a bit more challenging than expected but doable. Then, I installed all the required libraries. The final step was to create a scheduled task to start the Discord bot on startup.

On my PC side, I also set up a scheduled task to run the script to obtain the token on startup, and voila!

