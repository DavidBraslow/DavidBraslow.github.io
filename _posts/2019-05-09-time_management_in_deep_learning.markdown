---
layout: post
title:      "Time Management in Deep Learning"
date:       2019-05-09 18:39:01 -0400
permalink:  time_management_in_deep_learning
---

I think it is very easy to get stressed about time when working on machine learning problems. This can be a whole bunch of big picture reasons:

* You have a deadline
* You're stuck and not sure what to do
* You get distracted or bored
* You spend a lot of time on something that ends up not being useful
* Models take a long time to run
* It's never clear when you're "done"

These time-related issues can make a difficult machine learning task harder, as I learned with my most recent Deep NLP project. I've decided that, in a broad sense, deep learning can be formulated as a time-management problem.

## How much time do I have?

To start managing your time on a project, it first helps to figure out how much time you actually have in which to do it. For some projects this is an easy question: you have a deadline, and you have as much time as there are hours between now and then. But even then, you have other constraints. Maybe you only work from 9 to 5, and you have other competing responsibilities during that time. Maybe you work from home and have kids who might need you at any moment. For most of us, it is impossible to say "I have exactly 27 hours to work on this project". At best, you might be able to estimate something like "20-30 hours" with any confidence. 

When there is no deadline, things can quickly get out of hand. You might get lucky and finish early. But if things take a long time, they can drag on and on while you search for solutions or further improvements. At some point you have to call it, but you never know when that point is.

For my project, I had a target date by which I wanted to finish, but at the end of the day it wasn't done. I knew what the standards were for the work because of the Project Specifications, and I knew I hadn't met them yet.  Because the target date was a self-imposed deadline, it was no big deal to keep working on it, but I didn't know how much longer I should keep going.

## How long should I spend getting the data ready to model?

Without a clear sense of how long a project is going to take, it can be difficult to know how much time to spend on each part of it. At the start of a project, should I spend 1 hour cleaning my data, or 5? I don't know yet how much it will matter for the performance of my model. The cleaner the data, the easier the analysis becomes, but I don't want to spend forever getting it absolutely spotless and then run out of time to actually do the modeling. Usually I start with a quick-and-dirty data cleaning job, getting it in functional shape to start running models, and then return to it if once I have a better sense of what the model performance looks like. If I have time.

Next is data exploration, probably my favorite part. Getting my hands dirty in the data allows me to start getting a real handle on the problem. Often this surfaces issues with data cleaning that I need to circle back to. I could spend hours in this loop of finding data problems and fixing them, only to find new problems. This is particularly true for NLP projects!  But at a certain point, I had to call it and move on. That point ended up being way too late for me to meet my deadline.

## How should I spend my modeling time?

Modeling was the next phase, and this was where time really became a problem. The model I wanted to use was a Bidirectional Long Short-Term Memory (LSTM) model, and at first it crashed my Kernel when I ran it. It turned out to be a memory issue, which I was able to fix by reducing the batch size. However, this portended the phase where I had to think about time more than about building the best model.

First, I wanted to learn more about LSTM models before using it in my project, in order to make sure I was using them properly. In addition to the materials from Flatiron, I spent hours searching the internet for more information about these models. It was all very interesting, but it ultimately didn't make my project much better than if I had stuck with the Flatiron materials.

Second, I wanted to make sure I had as much processing power available as possible to run my models. My computer is only a moderately powerful PC. I decided to get my computer set up to use my graphics card (GPU) to run the model, rather than my main processor (CPU). It ended up taking a fair amount of time to get my computer set up to do this, and it ultimately only improved my computing speed marginally. While it was a valuable learning experience, it didn't help me meet my deadline any faster.

Next, I had to debug the code that ran the models. It is very disappointing to wait expectantly for your model to run, only to find that you didn't properly save the output! I decided to run quicker models, with less data and only one epoch, to help me squash the remaining bugs before specifying a model that I might actually want to submit.

Lastly, I knew that the more data and epochs I used, the better my model would be, but I couldn't just let it run for days. I worked out a compromise: I'd find a sample size and number of epochs that let me demonstrate that my model would improve if given more time and that gave me performance better than a benchmark model that I found on Kaggle. This let me feel comfortable limiting my model to 6 epochs, when deep down I wanted to see it run with 60.

## Conclusion

Managing time is both a psychological and technical problem. Thinking about machine learning as a time management problem helps me remember to engage my executive functions and make active choices about where to spend time, rather than going down every rabbit hole. Even the technical challenges can be thought of as a time management problem - with enough time, I could figure out how best to tackle them. Alas, time is finite, and it is important to pick one's battles. 

