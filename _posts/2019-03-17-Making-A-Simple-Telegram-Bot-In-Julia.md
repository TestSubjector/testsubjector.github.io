---
layout: post
title: "Making A Simple Telegram Bot In Julia"
date: 2019-03-17
---
Note - Not proofread yet
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
**Creating a bot**
* Firstly, we need to have a bot to call our own. Bots in Telegram *are special accounts that do not require an additional phone number to set up* and to make these special accounts, Telegram provides an ... in-built bot(!) called the [BotFather](https://telegram.me/botfather). Once you have it running, it is pretty self explainatory in the way it is supposed to be used.
<br><br/>
<img src="{{site.url}}/images/telegrambots/botfather1.png" margin: auto alt = "The Botfather">
<br><br/>
**Communicating with your bot**
* After the bot has been named, its details set up and after it has been adopted as an official child (do remember that adoption is invalid if the bot token is lost, which is also a sign of bad parenting), the next thing required is a way to communicate with it. Stumped? Well, no need to worry. In this highly introverted age, humans have made a marvellous tool called the application programming interface (APIs) to remove all unnecessary friendly social communication. [Telegram's API](https://core.telegram.org/bots/api) is a party to this principle.
<br><br/>
**Wrappers and the beauty of the open-source world**
* But wait, we plan to use Julia for this. How does one go about linking the language to the APIs i.e creating a wrapper library? Seems like a lot of effort. It's simple really, you simply don't make that effort. Someone else usually has done it already. And the person in question is [Moelf](https://github.com/Moelf) with his Julia wrapper library [Telegrambot.jl](https://github.com/Moelf/Telegrambot.jl). Huge thanks to him for that. You can also find his blog on the topic [here](https://blog.jling.dev/a-telegram-bot-in-julia/).
<br><br/>
**Getting the bot commands running**
* Now, first thing first is to get the [example shown in the README](https://github.com/Moelf/Telegrambot.jl/blob/master/README.md) for the Telegrambot.jl running. After that is done, we diverge on how to make our *random_wikipedia_page_giver* bot.
<br><br/>
* We would like to have a main() function, which will act as the placeholder where all the code runs -
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
* Now the inner workings of the code will be understood by going through the docs but essentially the *txtCmnds* are the text commands that we will give as input to the bot and get the expected response in return. The startBot method make sures that everything is running in a neat, tight loop. For example, here is the /*repeatmessage* command in action.<br><br/>
<img src="{{site.url}}/images/telegrambots/repeatmessage.png" margin: auto alt = "Repeat Message">
<br><br/>
**Bot Commands - Minor**
* The commands are functions themselves, so we write them over in seperate commands.jl file. The two simpler commands are -
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
* Its time to digress a bit. The thing was that originally using the welcomeMessage command will pretty quickly fill your console with warning messages quickly - <br><br/>
<img src="{{site.url}}/images/telegrambots/allstringstarterror.png" margin: auto alt = "Warnings everywhere">
<br><br/>
* The details can be learned about in this [issue](https://github.com/Moelf/Telegrambot.jl/issues/5). In brief, the way the wrapper code was set up required all comands to have a message after command (*/bot_command* non_empty_message). Moelf fixed the issue by himself (even though I said I'll do it, mid-semester exams made it impossible for me to quickly work on this). So this is a non-issue now.
<br><br/>
**Bot Command - Major**
* The */randomarticle* command, the crux of our code requires two things. Since we need to get articles from wikipedia randomly, it may seem like we will have to create a repository of multiple links on Wikipedia and randomly choose one of those links when a user gives a request. There is a simple way. Wikipedia has an API(who would have know it) that will automatically give you the metadata for a random page on Wikipedia.<br><br/>
* All we now require is a way to communicate with this API. We use a Julia library called [HTTP.jl](https://github.com/JuliaWeb/HTTP.jl) for this purpose. With all the key ingredients ready, the final code for the function will loke somewhat like -
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
* From the return statement you can see that this function will return a (pseudo)random article link on Wikipedia to the user.
<br><br/>
**Packing up**
* A small problem I've found people face difficulty with, when using Julia is in how to use functions across multiple files. The answer is *[modules](https://docs.julialang.org/en/v1/manual/modules/index.html)*, _a nifty yet a somewhat back and forth experience (it has both import and using, cmon!). I usually refer to the way I saw modules being used back in my GSoC days with AstroLib.jl, when Julia was still at v0.6. The code for a module, stored in the file which I'll call main_module.jl is -
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
* So we have the bot, the module, the code and the commands ready. How to run everything? Simple, you make small script to execute!
```julia
push!(LOAD_PATH, "/path/to/directory/containing/the/code")
using main_module
main()
```
* Perfect, all you need to do is keep your system running 24/7 with the code executing in the background for the bot to be always up. Which, as you can see is a slight problem. What you need, is the cloud(or a Raspberry Pi and unlimited net, but I don't have the former). To keep the bot persistent, I created a VM instance in Google Cloud (many other cloud free-for-small-use-case services are not available in my region or problematic to set up with Julia. If someone knows of an alternative, please do tell). <br><br/>
* Shift all the necessary code to the instance and a simple *nohup /path/to/julia/execuable your_script_to_run_the_code.jl &* command later, you have bot that you can access from Telegram anytime and share it with your friends and just feel happy looking at it working. Until your free one-year Google subscription ends or until someone makes a half-hearted attempt at breaking the bot. <br><br/>
* As of this moment(when the post appears online), the WikiRandom bot works - http://t.me/WikiRandomBot. Please have a go at using it (without trying to break it preferably). I may come back to this and improve the commands and response from the bots if I use it enough. Or I could just revert the code back to when the bot was its utopian form.
<br><br/>
<img src="{{site.url}}/images/telegrambots/allstart.png" margin: auto alt = "Perfection">
<br><br/>

Thank you for reading this roughly made article.
