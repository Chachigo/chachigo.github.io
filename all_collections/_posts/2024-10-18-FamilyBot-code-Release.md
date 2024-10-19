---
layout: post
title: FamilyBot Code Release
date: 2024-10-19 15:30:00
categories: ["Dev","Python","bot","Discord"]
---

# 🚀 FamilyBot: The Code Release

After months, I'm thrilled to announce that FamilyBot is finally released.

## 🔧 Enhancements Overhaul

One of the most significant additions in FamilyBot is the implementation of plugins. This feature allows developers to extend the bot's functionality without modifying the main script.

To take advantage of this feature, I've reorganized our code structure into a more modular and maintainable format. The new file structure includes separate libraries for game management, utility functions, token management, and plugin development.

## 💻 Key Features

- **Steam Family**
This plugin includes all the features related to Steam Family.
The current features are:
    - Send a notification when a new game is added to the Family library
    - Compare wishlists to get common games to share prices between multiple users who want the same game
    - ``!coop <number>`` command that returns all multiplayer games in the Family library with at least the specified number of copies

- **Free Epic Games**
Send a notification in a given channel about new weekly free games on the Epic Games Store

- **Common Games**
Add the following commands:
    - ``!register <SteamId>``: Link Discord ID to Steam ID
    - ``!common_games @user1 @user2`` Get multiplayer games common to sender's Steam library and tagged users
    - ``!list_users`` Get list of users who have linked their SteamID with their DiscordID using the ``!register`` command

- **Help Message**
Dynamically generate a help message for plugin commands

## 👥 Join the FamilyBot Community!

Feel free to share your thoughts, suggestions, and feedback with me, and help enhance FamilyBot.
Find the source code for [FamilyBot on GitHub](https://github.com/Chachigo/FamilyBot).

## ❓ What's Next?
Here are some features I've thought about:
- Notify users who have wishlist games in an upcoming Humble Bundle