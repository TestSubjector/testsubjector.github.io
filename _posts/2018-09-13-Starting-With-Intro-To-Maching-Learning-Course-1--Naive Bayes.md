---
layout: post
title: "Starting With Intro To Maching Learning Course 1- Naive Bayes"
date: 2018-09-13
---

In essence, now that I'm in my 4th year, I need to think about my future. Unfortunately, as my academics can be currently be very aptly termed as a **Disaster Zone**, I have to finally pick up the slack all-around. This is to make me capable of being able to bring food to my table, in the near future.  
That, and I'm now really tired(read envious) of seeing other people achieve stuff and play games and have fun without a care in the world.
  
Now, AI was something I wanted to get into as kid(and seeing everyone else suddenly get hyped about the same, in these last few years has been damn frustrating (The competition!)). 
Anyway, I can rant a lot about this to be sure; but let's get on with the main topic. 
   
I've started with the basic of the basics, the Udacity course Introduction To Machine Learning. I'll try to put up updates about this course as I go deeper into it. 
Maybe I'll also put up some summary notes here in the future? Let's try with a brief one.  
  
The first main topic, Naive Bayes can be (on the code side) be summed up by the following few lines - 

```python
from sklearn.naive_bayes import GaussianNB
clf = GaussianNB()
clf.fit(features_train, labels_train)
pred = clf.predict(features_test)
accuracy = clf.score(features_test, labels_test)
```
  
Shoutout to my [friend](http://arnavdhamija.com/) for prodding me to *attempt* to blog again.
