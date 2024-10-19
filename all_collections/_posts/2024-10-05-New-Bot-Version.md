---
layout: post
title: New Bot Version
date: 2024-10-05 13:00:00
categories: ["Dev","Python","bot","Discord"]
---

## 💡Introduction
From the start, I considered using "plugins" to add features instead of modifying the main file. However, after trying multiple approaches without success, I realized that making plugins automatically load at script startup was a challenging task. Recently, my brother asked me to add a new feature, and I selfishly decided not to do it until I'd implemented a plugin feature.

## 💻 Adding Scalability

I found the "**discord-py-interactions**" library, which is built on top of "**discord.py**", offering more functionalities and an extension functionality that exactly met my needs. However, I had to adapt my code to work with this library.

To make development easier, I separated my code into multiple files based on their functions. This made it simpler to implement plugins. Additionally, I created a configuration file (config.yml) to store settings like channel IDs and user IDs in one place.

### Config file:

In the past, I used `configparser`, but found it too heavy for my use case. My dad, also a programmer, recommended using **YAML** as an alternative. The main advantages of YAML are: it's lightweight, easily readable and comprehensive, comment-compatible, and easy to implement.

To make config file usage even easier, I created a `config.py` file that reads the configuration file and sets constants for each configuration value.

### Plugins:

With "**discord-py-interactions**", adding extensions is relatively straightforward. I modified my main code to act as a Discord server that loads all functionalities as plugins. To take advantage of this feature, I organized my functions into custom libraries and created a new file structure for the bot:
```
FamilyBotV2/
├── lib/
│   ├── familly_game_manager.py
│   ├── family_utils.py
│   ├── token_manager.py
│   └── utils.py
├── plugins/
│   ├── common_game.py
│   ├── free_epicgames.py
│   ├── help_message.py
│   └── steam_family.py
├── config.py
├── config.yml
└── FamilyBot.py
```

## ✨ New Feature
### Common Games
My brother requested a feature to get all common multiplayer Steam games among multiple users. The twist was that he wanted to use Discord @ tags instead of Steam IDs, making it more convenient.

To achieve this, I created the `!register` command, which uses the syntax `!register <SteamID>`. This command saves the correspondence between a Discord @ tag and a Steam ID in a CSV file. Since only two columns are involved, CSV was an ideal choice for readability and writeability.

Next, I developed the `!common_games` command, used as `!common_games @user1 @user2`, which fetches the Steam IDs of the sender and listed users. Using the Steam API, it retrieves each user's game library and identifies common games among them. After some tweaking with AI, I successfully implemented a function to compare game libraries:

```python
def find_common_elements_in_lists(list_of_lists):
    # Initialize an empty set to store the common elements found so far
    common_element = set()
    
    # Iterate over each pair of lists in the list of lists
    for i in range(len(list_of_lists)):
        for j in range(i+1, len(list_of_lists)):
            # Find the intersection of the current two lists and update the common_element set
            common_element = set(list_of_lists[i]).intersection(set(list_of_lists[j]))
            
            # Intersect the common_element with the first list to ensure uniqueness across all lists
            common_element = common_element.intersection(set(list_of_lists[0]))
    
    # Return a sorted list of unique common elements found in all lists
    return sorted(list(common_element))
```

To handle cases where an @ tag isn't registered, I created the `!list_user` command to simply respond with the list of registered users.

### Help Message
In response to my brother's comment about needing documentation for all commands, I developed a plugin that extracts specially formatted comments from the plugin Python file and generates a Discord message containing all commands per plugin.

The plugin updates the bot's welcome message at startup. Here is the current help message:  
![alt text](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/help_message.png?raw=true)

## Additional Comments
I'll be publishing the code soon, but first, I'd like to thoroughly test it, address any issues, and create comprehensive documentation on installation and usage.

To ensure my coding is error-free and polished, I'm now utilizing LLaMA 3.2, which runs locally, providing a convenient and efficient means of spell-checking articles and offering guidance during the development process.