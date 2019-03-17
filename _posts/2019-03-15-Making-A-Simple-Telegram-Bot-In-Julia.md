---
layout: post
title: "Making A Simple Telegram Bot In Julia"
date: 2019-03-15
---
So finding myself with some free time on hand and a persisting interest in Telegram bots due to my daily use of the FeedReaderBot (which I have talked about in a previous post), I decided to create a bot of my own. <br/>
And well, what better language to use than Julia? Yes, Python has a much more developed library for Telegram bots, but where is the fun in that? <br/>
<br/>
So, here I'll be doing a quick run-through on how I a simple bot was made for Telegram called WikiRandom bot. The purpose of the bot is simple and clean, a user will ask the bot to give a link to any random article on Wikipedia and the bot will give respond with such a link.<br/>

Here is an example of the bot in action - <br/>
<img src="{{site.url}}/images/telegrambots/workinglist.png" margin: auto alt = "It's Alive!">


Now, let's start with a simple primer for those not familiar with the topic in hand.<br/>

**What is Telegram?**<br/>
It is a chat application like Messenger or WhatsApp with the added promise of being much more secure (and a clean UI!)

**What is a [Telegram Bot](https://core.telegram.org/bots)?**<br/>
Telegram provides this nifty feature of bots, which are in essence small pieces of code wrapped up in accout. You communicate with the bot using valid commands and the bot responds to those commands based on how it is coded.

**What is Julia?**<br/>
It is a good language to code in. Faster than Python (usually) but with code that looks like it's a long lost brother to the Python syntax. For anything more than that, please find out by yourself.<br/>

Now, we start looking into the actual approach to build our bot.
* Firstly, we need to have a bot to call our own. Bots in Telegram *are special accounts that do not require an additional phone number to set up* and to make these special accounts, Telegram provides an ... in-built bot(!) called the [BotFather](https://telegram.me/botfather). Once you have it running, it is pretty self explainatory in the way it is supposed to be used.
<br><br/>
<img src="{{site.url}}/images/telegrambots/botfather1.png" margin: auto alt = "The Botfather">
<br><br/>
* After the bot has been named, its details set up and after it has been adopted as an official child (do remember that adoption is invalid if the bot token is lost, which is also a sign of bad parenting), the next thing required is a way to communicate with it. Stumped? Well, no need to worry. In this highly introverted age, humans have made a marvellous tool called the application programming interface (APIs) to remove all unnecessary friendly social communication. [Telegram's API](https://core.telegram.org/bots/api) is a party to this principle.
<br><br/>
* But wait, we plan to use Julia for this. How does one go about linking the language to the APIs i.e creating a wrapper library? Seems like a lot of effort. It's simple really, you simply don't make that effort. Someone else usually has done it already. And the person in question is [Moelf](https://github.com/Moelf) with his Julia wrapper library [Telegrambot.jl](https://github.com/Moelf/Telegrambot.jl). Huge thanks to him for that.
<br><br/>
* Now, first thing first is to get the example shown in the README for the Telegrambot.jl running. After that is done, we diverge on how to make our *random_wikipedia_page_giver* bot.
<br><br/>
* We would like to have a main() function which will act as the place where all the code runs, something like this -
```julia
using Telegrambot
function main()
    botApi = <"get your own token, shoo!">
    txtCmds = Dict()
    txtCmds["start"] = welcomeMessage # this will respond to '/start'
    txtCmds["repeatmessage"] = echo # this will respond to '/repeatmessage <any thing>'
    txtCmds["randomarticle"] = getRandomWikiPage # this will give the random article we require
    inlineOpts = Dict() #Title, result pair
    startBot(botApi; textHandle = txtCmds, inlineQueryHandle = inlineOpts)
end
```
* The txtCmnds are the text commands that we will give as input to the bot, and get the expected response in result. For example, here is the /*repeatmessage* command in action.<br><br/>
<img src="{{site.url}}/images/telegrambots/repeatmessage.png" margin: auto alt = "Repeat Message">