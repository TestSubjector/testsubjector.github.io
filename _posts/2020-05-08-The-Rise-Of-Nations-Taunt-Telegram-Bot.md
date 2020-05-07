---
layout: post
title: "The Rise Of Nations Taunt Telegram Bot"
date: 2020-05-08
mathjax: true
---
----------------
My adventures with bots in the Telegram app continues.

I have a group chat on Telegram with my college wingmates, folks that have lived adjacent to
for almost four years of my undergraduate hostel life. A few days back, one of said wingmates went and
sent forth on the group the message *Taunt 3* in response to some discussion.

Now for a bit of a background, this message in question is a command found in the video game [Rise of Nations](https://en.wikipedia.org/wiki/Rise_of_Nations) or
RoN, by Microsoft.
It is one of the many taunt commands that essentially plays a short voice clip in game when used. Examples of 
such voice lines include *Taunt 8 - (I need Oil)* or *Taunt 60 - (Random! Random!)* [^1]. *Taunt 3 was for (Maybe)*, and while the actual voice clip did not play in the chat, everyone understood what that specific friend wanted to convey through.

It was at that point that I questioned myself that what was stopping us from having the actual voice clips play, when a message matching the RoN taunt command was sent to the group.
And what better way to do that then to use a Telegram bot to accomplish this task.

Now I've used the [Python Telegram library](https://github.com/python-telegram-bot) for this specific bot. The reason being that it has a much better developed API interface and is a much more mature library compared to any other library out there. The bot is persistent and in every execution cycle it parses every message that is passed in a group it is a part of. To allow a Telegram bot to have this access
to the messages of a group, you have to change the default privacy mode of the newly created bot using the [BotFather](https://core.telegram.org/bots#6-botfather).

The bot will discard any message greater than a specific to avoid too much parsing of the message and decrease the burden on the bot.
The bot will essentially try to match the given message to a *Taunt X* type format. The word taunt followed by a integer ranging from one to hundred, which encompasses all the default taunts in RoN.

The most time consuming section of making this bot was the part where every Taunt had to be matched to a specific audio file. These matches were found via
creating a dictionary that matches every number to its specific audio file. And sadly, this had to be done manually as the audio file names did not correspond with their numbers. By this I mean that for say *Taunt 35*, the relevant audio file is not called something like *taunt35.wav* but instead *ships_ahoy-3.wav*.

Please [click here](https://github.com/TestSubjector/RONTauntTelegramBot) to go to the Github repository with the actual code of the bot[^2].

A friend of mine, [Nischay Ram](https://github.com/Nischay-Pro) improved the bot code by adding queues for [flood prevention](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Avoiding-flood-limits), created the ability for multi-taunts in one single message like *Taunt 33 84 1* to be possible and even added the Age Of Empires's (another videogame) taunt lines to the bots dictionary.

In conclusion, I would like to inform the reader that the bot was a roaring success in our Telegram group. At least for some time, and then complaints against the spamming of the bot and of the barrage of large number of notifications meant that it is used much more sparingly now.

**References:**

[^1]: [Rise of Nation Taunt List](https://steamcommunity.com/app/287450/discussions/0/540744474731911106/)
[^2]: [Github Repository](https://github.com/TestSubjector/RONTauntTelegramBot)