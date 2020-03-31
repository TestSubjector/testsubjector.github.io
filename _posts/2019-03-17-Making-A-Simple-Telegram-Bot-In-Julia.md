---
layout: post
title: "Making A Simple Telegram Bot In Julia"
date: 2019-03-17
---

So finding myself with some free time on hand and a persisting interest in Telegram bots due to my daily use of the FeedReaderBot (which I have talked about in a previous post), I decided to create a bot of my own. <br/>
And well, what better language to use than Julia? Yes, Python may have a much more developed series of libraries for making Telegram bots, but where is the fun in using that? <br/>
<br/>
Therefore, here I'll be doing a quick run-through on how a simple bot called WikiRandom bot was made for Telegram. The purpose of the bot is very simple and concise i.e a user will ask the bot for a link to any random article on Wikipedia and the bot will respond with such a link.<br/>

Here is an example of the bot in action - <br/>
<img src="{{site.url}}/images/telegrambots/workinglist.png" margin: auto alt = "It's Alive!">


Firstly, let's start with a quick primer for those not familiar with the topic in hand.<br/>

**What is Telegram?**<br/>
It is a chat application, similar to Messenger or WhatsApp with the added promise of being much more secure (and a clean UI!)

**What is a [Telegram Bot](https://core.telegram.org/bots)?**<br/>
Telegram provides this nifty feature known as bots, which are in essence small pieces of code wrapped up as a dummy account. You can communicate with the bot using valid commands and the bot responds to those commands based on how it is programmed.

**What is Julia?**<br/>
It is a good language to code in. Faster than Python (usually) but with code that looks like it's a long lost brother to the Python syntax. For anything more than that, please find out by means of some healthy googling.<br/>

Now, we start looking into the actual approach to build our bot.
<br><br/>
**Creating a bot**

Before anything else, we need to have a bot to call our own. Bots in Telegram *are special accounts that do not require an additional phone number to set up* and to make these special accounts Telegram provides an ... in-built bot(!) called the [BotFather](https://telegram.me/botfather). Once you have it running, it is pretty self explanatory in the way it is supposed to be used.
<br><br/>
<img src="{{site.url}}/images/telegrambots/botfather1.png" margin: auto alt = "The Botfather">
<br><br/>
**Communicating with your bot**

When the bot has been named, its details set up and after it has been adopted as an official child (do remember that adoption is invalid if the bot token is lost, which is also a sign of bad parenting), the next thing required is a way to communicate with it. Stumped? Well, no need to worry. In this highly introverted age, humans have made a marvellous tool called the application programming interface (APIs) to remove all unnecessary friendly social communication. [Telegram's API](https://core.telegram.org/bots/api) is a party to this principle.
<br><br/>
**Wrappers and the beauty of the open-source world**

But wait, we plan to use Julia for this. How does one go about linking the language to the APIs i.e creating a wrapper library? This seems like a lot of effort. It's all very simple really, you simply don't make that effort. Someone else usually has done all that hard work already. And the person in question is [Moelf](https://github.com/Moelf) with his Julia wrapper library [Telegrambot.jl](https://github.com/Moelf/Telegrambot.jl). Huge shoutout to him. You can also find his blog on the topic [here](https://blog.jling.dev/a-telegram-bot-in-julia/).
<br><br/>
**Getting the bot commands running**

Let's start with the nitty-gritty now. First thing first is to get the [example shown in the README](https://github.com/Moelf/Telegrambot.jl/blob/master/README.md) for the Telegrambot.jl running. It's a simple way to make sure that everything has been set up properly on all sides. After that is done, we direct ourselves on how to make our so called *random_wikipedia_page_giver* bot.
<br><br/>
We would like to have a main() function, which will act as the placeholder where all the code runs -
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
Now the inner workings of the code will be understood by going through the docs but essentially the *txtCmnds* are the text commands that will be given as input to the bot and in return the bot gives the expected response. The startBot method make sures that everything is running in a neat, tight loop. For example of a text command, here is the /*repeatmessage* command in action.<br><br/>
<img src="{{site.url}}/images/telegrambots/repeatmessage.png" margin: auto alt = "Repeat Message">
<br><br/>
**Bot Commands - Minor**

The commands are functions themselves, so we can write them over in separate commands.jl file. The two simple commands are -
```julia
# This command gives a greeting to you, how polite
function welcomeMessage(incoming::AbstractString)
    return "Welcome, the most generic greeting ever"
end
# Echo back whatever has been said by the user
function echo(incoming::AbstractString)
    return incoming
end
```
It's time to digress a bit. There was an issue originally that could easily lead to using the welcomeMessage command (or some other command) to pretty quickly fill your console with warning messages - <br><br/>
<img src="{{site.url}}/images/telegrambots/allstringstarterror.png" margin: auto alt = "Warnings everywhere">
<br><br/>
The details can be learned about in this [issue](https://github.com/Moelf/Telegrambot.jl/issues/5). In brief, the way the wrapper code was set up required all commands to have a message(parameter) when sent to the bot (*/bot_command* non_empty_message). Moelf fixed the issue by himself, adding other improvements to the code (even though I said I'll do it, mid-semester exams made it impossible for me to quickly work on this). So this is a non-issue now.
<br><br/>
**Bot Command - Major**

The */randomarticle* command, which will be the crux of our code requires two things. Since we need to get articles from Wikipedia randomly, it may appear at first glance that we will have to create a database of multiple links consisting of Wikipedia pages and randomly choose one of those links when a user gives a request. There is thankfully a simpler way to do this. Wikipedia has an API(who would have known it) that will automatically give you the metadata for a random page on Wikipedia.<br><br/>
All we now require is a way to communicate with this API. We use a Julia library called [HTTP.jl](https://github.com/JuliaWeb/HTTP.jl) for this purpose. With all the key ingredients ready, the final code for the command will look something like -
```julia
using HTTP
function getRandomWikiPage(incoming::AbstractString)
    httpResponse = HTTP.request("GET", "https://en.wikipedia.org/w/api.php?action=query&list=random&format=json&rnnamespace=0&rnlimit=1"; verbose=1)
    responseData = deepcopy(String(httpResponse.body)[:])
    # print(responseData)
    regexMatch = match(r"\"id\":\d+", responseData)
    # print(regexMatch.match)
    idMatch = regexMatch.match[6:end]
    # return idMatch
    return string("https://en.wikipedia.org/?curid=", idMatch)
end
```
From the return statement you can see that this function will return a (pseudo)random article link on Wikipedia to the user.
<br><br/>
**Packing up**

A small problem that I've found people face difficulty with when using Julia is, in how to use functions across multiple files. The answer is *[modules](https://docs.julialang.org/en/v1/manual/modules/index.html)*, _a nifty yet a somewhat back and forth experience (it has both import and using, cmon!). I usually refer to the way I saw modules being used back in my GSoC days with AstroLib.jl, when Julia was still at v0.6. The code for a module, stored in the file which I'll call main_module.jl is -
```julia
__precompile__()
module main_module
    include("WikiRandom.jl") # This contains the main function
    export main
    include("commands.jl") # This contains all the commands
    export welcomeMessage, echo, getRandomWikiPage
end
```
<br><br/>
**Persistent Running**

So we have the bot, the module, the code and the commands ready. How to we get everything to run? Simple, make a small script to execute!
```julia
push!(LOAD_PATH, "/path/to/directory/containing/the/code") # A somewhat annoying workaround required when using modules in Julia
using main_module
main()
```
Perfect, now all you need to do is keep your system running 24/7 with the code executing in the background; so that the bot will be always up. Which, as you can see is a slight problem. What you need, is the cloud(or a Raspberry Pi and unlimited net, but I don't have the former). To keep the bot persistent, I created a VM instance in Google Cloud for this purpose (many other cloud free-for-small-use-case services are not available in my region or problematic to set up with Julia. If someone knows of an alternative, please do tell). <br><br/>
Shift all the necessary code to the instance and a simple *nohup /path/to/julia/executable your_script_to_run_the_code.jl &* command later, you have bot that you can access from Telegram anytime and share it with your friends and just feel happy seeing it in action. Until of course the free one-year Google subscription ends or till someone makes a half-hearted attempt at breaking the bot. <br><br/>
As of this moment(when the post appears online), the WikiRandom bot works - http://t.me/WikiRandomBot. Please have a go at using it (without trying to break it preferably). I may come back to this and improve the commands and response from the bots if I use it enough in the future. Or I could just revert the code back to when the bot was in its utopian form (shown below).
<br><br/>
<img src="{{site.url}}/images/telegrambots/allstart.png" margin: auto alt = "Perfection">
<br><br/>

Thank you for reading this roughly made article.
