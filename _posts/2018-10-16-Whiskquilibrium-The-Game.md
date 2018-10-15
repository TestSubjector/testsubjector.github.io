---
layout: post
title: "Whiskquilibrium The Game"
date: 2018-10-16
---

On 29th and 30th September, I had the chance to participate in the [Megathon Hackathon at IIIT Hyderabad](https://megathon.in/).  
Hackathons I fell are a bit of a strange thing, the novelty of cooking something up in a day or two versus the knowledge that most of what will be churned out in such events will be empty air in mere hours.  
  
Still, it's a good thing to try out at least once and/or if you're in a coding slump.  
  
With a team consisting of me, [Kushal Agarwal](https://github.com/legobridge), [Arnav Dhamija](https://github.com/shortstheory) and [Nischay Ram](https://github.com/Nischay-Pro) (appropriately called Team "CanWeGetBackToYouLater"), we devised and developed **Whiskquilibrium**. A 2D platformer game in the Godot game engine.  
  
Here's the PR promoting synopsis - *It involves a cat named "Minto" trapped in a quantum world. She has the ability to shift her state between being alive and a ghostly dead form. Her presence itself changes the nature of the world around her. Can you solve the obstacles in her path that prevent her from escaping the mad world she is in?*  
  
* Essentially, the game consisted of a cat character that has the power to switch colors between white and black and has to clear several levels.
* The blocks themselves in these levels are white, black or gray(level borders). The game mechanic is developed in the form of the cat being able to traverse through blocks of the same colour while being able to *paint* the blocks of the opposite colour, when *you stop touching that block*.  
* The cat can jump in its default black colour
* The cat cannot jump when it switches to white(dead) state but has limited ability to fly which can be refilled by colouring up black blocks to white.
* The number of switches possible in a level is limited.
* The level can be reset at anytime (this is not a roguelike game *phew*)
  
Here are the sketches of the above mechanics, that we developed during the hackathon.

<img src="https://media.giphy.com/media/8AlZv8xCgjfwt2wv5z/giphy.gif" width="400" height="300" alt = "Colour/ Switch">
<img src="https://media.giphy.com/media/j9O9cCf7lmJzql78oc/giphy.gif" width="400" height="300" alt = "Fly">

The judging of the Game Dev section of the Hackathon was done by professionals from EA India (*insert jokes about DLCs here*).  
Suffice to say we did not win the 1st (given to a team that used RPG Maker to make more than 20 levels of content) or 2nd prize given to a team who make a game using their on OpenGL game engine (which they had been working on more than a few months, which highlights another problem of hackathons).  

Thankfully, we had a spot in the next 3 ranks (we were lazily, not told the exact rank since the **prizes** for these 3 spots were the same).  
<img src="https://i.imgur.com/Jn5q0K1.jpg" width="400" height="300" alt = "The Prize">

I was actually very happy at the way the judges had given scores to be honest. They had instantly picked out people who had cut and pasted code and assests from the net (essentially cheating) quite quickly. It was satisfying to watch them (such participants) being tongue-tied when the difficult questions where asked, I'll admit.  
  
*This hackathon actually helped me fulfill one my college goals, which was to make a proper effort at a developing a game. Who knows? If we are able to actually gather willpower, maybe we'll polish this and release this on Steam.*
  
Game development, in itself...is a pain. We had 4 people in our team and around 18 hours of time to understand Godot and make something. I can remember at least 5 instances, where at least one of us was on the verge of screaming, killing the other three and dancing on their corpses. The essence and the main difficulty with game development that I felt was that, everything in games are linked. It's like how a real world behaves.  
You want to change one move? Change everything that is dependent on that move.  
You want to add visual indicators? Make sure that they - 
* follow the camera 
* are resized properly when the camera is zoomed in/out
* do not lag behing key presses
* look good
* and work as they are supposed too

Now when you are novice (like we where), where and how to actually implement the above in a game engine(all of whom a bazillion icons to chose from), is more than a bit of a pain.
  
  
To conclude,  
the most amazing yet sad thing is, at the end of it all, we were more than happy at what we had been able to accomplish at the hackathon. We were sleep deprived of course, but seeing that the game actually came to life brought that small-persistent-warm-giddy feeling to our hearts (We had implemented sound effects. Sound effects!).  
  
The reason I was feeling sad though, was because of one of the conclusions I had reached. That was, while being an indie game developer will definetly be a fun and interesting job, you really need to be a bit of a masochist to be able to qualify for such a job.
  