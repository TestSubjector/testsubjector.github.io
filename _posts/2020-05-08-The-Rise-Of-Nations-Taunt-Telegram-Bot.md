---
layout: post
title: "The Rise Of Nations Taunt Telegram Bot"
date: 2020-05-08
mathjax: true
---
----------------
My adventures with bots in the Telegram app continues in this blog post.

I have a group chat on Telegram that includes my college wingmates. A place to stay connected with the folks that have lived adjacent to me
for almost four years of my undergrad hostel life. A few days ago in the chat group, one of said wingmates went and
sent forth the message *Taunt 3*, in response to some discussion.

Now for a bit of a background. This message's content is a command found in the video game [Rise of Nations](https://en.wikipedia.org/wiki/Rise_of_Nations) or RoN, by Microsoft.
It is one of the many taunt commands found in the game that essentially play a short audio file, when used in game. Some examples of 
such voice lines include *Taunt 8 - (I need Oil)* or *Taunt 60 - (Random! Random!)* [^1]. *Taunt 3 was for (Maybe)*. While the actual audio file did not play in the chat, everyone understood what that specific friend wanted to convey through the message (that he was saying maybe).

It was at that point, when seeing the message, that I asked myself. What was stopping us from having the actual audio files play? When a message matching the RoN taunt command was sent to the group, the relevant audio file could be sent to the group chat. And what better way to do this, than to use a Telegram bot for such an endeavour?

Do note that I've used the [Python Telegram library](https://github.com/python-telegram-bot) for this specific bot. The reasoning behind this being that it has a better developed API interface and in general, is a much more mature library compared to any other similar library out there. The bot, in every execution cycle. parses every message that is passed in a group of which it is a part. To allow a Telegram bot to have this level of unrestricted access
to the messages of a group, you have to change the default privacy mode of any such bot using the options given by the [BotFather](https://core.telegram.org/bots#6-botfather).

Continuing on, the bot will discard any message greater than a specific length to avoid too much parsing of the message. This also decreases the burden on the bot. The primary task of the bot is to match every valid message format of *Taunt X*, X is an integer ranging from one to hundred. This encompasses all the default taunts in RoN.

The most time-consuming portion while making this bot was when every Taunt had to be matched to a specific audio file. These matches were found by
creating a dictionary that matches every number to its specific audio file. This was done so that the bot could match the given command to the correct audio file quickly which had to be done manually as the audio file names did not correspond to their numbers. By this I mean that for say *Taunt 35*, the relevant audio file is not called something like *taunt35.wav* but instead is called *ships_ahoy-3.wav*.

With all these features in place, the bot is now complete and can be deployed.

Please [click here](https://github.com/TestSubjector/RONTauntTelegramBot) to go to the Github repository with the actual code of the bot[^2].

A friend of mine, [Nischay Ram](https://github.com/Nischay-Pro) improved the bot code by adding queues for [flood prevention](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Avoiding-flood-limits). He created the ability for multi-taunts in one single message like *Taunt 33 84 1* to be possible and even added the Age Of Empires' (another videogame) taunt lines to the bots dictionary.

In conclusion, I would like to inform the reader that the bot was a roaring success in our Telegram group chat. At least for some time. Then the complaints against the spamming of the bot and the barrage of notifications it brought meant that it is now used much more sparingly.

**References:**

[^1]: [Rise of Nation Taunt List](https://steamcommunity.com/app/287450/discussions/0/540744474731911106/)
[^2]: [Github Repository](https://github.com/TestSubjector/RONTauntTelegramBot)
