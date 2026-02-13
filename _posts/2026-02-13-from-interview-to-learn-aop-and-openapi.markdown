---
layout: post
title: "From interview to learn AOP and OpenAPI"
date: "2026-02-13 12:38:00 +0300"
permalink: /:title
---

## Preface

The idea behind this post is to demonstrate how I learn and why one shouldn't make too hasty decisions what can and cannot do. I have a lot of confidence on my own skills to learn, but it works sometimes in a bit weird way. There is quite a lot of humor in this post, at least I laugh at myself and many times to what I'm learning. The humor makes learning easier and more memorable. Also it is possible to see that I don't really like to read the docs and I get sometimes a bit frustrated to just trying to undestand plain text. I learn by doing and eventually the acronyms and ideas behind the tech thing I'm learning about will click.

And there is no salt in this post at all.

## Why I wanted to learn AOP

I'm a consultant and I was in an interview with a potential fintech client. They asked me about AOP and how have I've used it in the past and what do I know about it. I didn't really know what AOP is and how to use it. Many times I've used something without really knowing the real terms or deeper ideas, but it later turned out that I didn't know anything about AOP and I had not used it anywhere. Not knowing in interviews hits me hard. I know that nobody knows everything, but in this context not knowing might eventually give me a lot of headache in the form of not employing myself. In retrospective, I figured out that AOP is something important in this position, because there were not many technical questions and the questions were broader than just some random technical details. This also puts more weight to know this kind of acronym later, if asked. You shouldn't make the same mistake twice.

I'm very curious person. Almost always when I stumble into something that I don't know or understand, I dig deeper into that and try to learn it. In other contexts than tech the learning might take years, but tech or software related stuff are eventually quite easy and logical. I learn mainly by doing and thinking, so let's do and think in the form of blog post.

## AOP itself

Let's see what the documentation says

> Aspect-oriented Programming ([AOP](https://docs.spring.io/spring-framework/reference/core/aop.html)) complements Object-oriented Programming (OOP) by providing another way of thinking about program structure. The key unit of modularity in OOP is the class, whereas in AOP the unit of modularity is the aspect. Aspects enable the modularization of concerns (such as transaction management) that cut across multiple types and objects. (Such concerns are often termed "crosscutting" concerns in AOP literature.)

What a bunch of mumbo jambo that tells me nothing I can grasp on. I use Merriam Webster and other dictionaries to understand words and take a look what it says about [aspect](https://www.merriam-webster.com/dictionary/aspect). I know the word 'aspect', but in this context from that explanation the documentation gives, I have no idea.

The page says

> AOP is used in the Spring Framework to: Provide declarative enterprise services. The most important such service is declarative transaction management.

My brains pick that word *enterprise* and of course *the most important*.

I click the link and open the page for [declarative transaction management](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative.html) thinking *wtf is this mumbo jumbo again*. I'm already feeling my action oriented *aspect* (pun intented) screaming for action instead of reading and then it hits me...this page tells *"AOP concepts do not generally have to be understood to make effective use of this code."* 

I laugh out loud.

A quick scan to the rest of the mumbo and I click back and go for the next page of the documentation.

The next page of the docs explains [key consepts of AOP](https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html). This seems to be a bit more readable page and maybe I can come back to this later. I quickly glance couple more pages, but there is too much text for now and too little use cases I understand and know where to use this. It is time to read some code and examples since I'm done with aspects and advices for now.

## Breaktrough

I search for "AOP examples" and get the Spring doc examples. Seems cool, but doesn't yet connect in my mind. I continue searching for something that breaks the ice. Software is not rocket science when the problems are broken down to simple pieces. Then I land on a [Medium post about AOP](https://medium.com/@sharmapraveen91/mastering-spring-aop-the-ultimate-guide-for-2025-55a146c8204c).

And there it is...**Use cases for AOP**

This mofo can be used to hook something and print something to log!

Now we are talking. Finally something real I can build on. Then I find out `@Before` so there must be at least `@After` and eventually I find out `@Around`. At the same time my brain is going full speed with infinite threads running and the rest of the post happens very quickly.

From some place I learn that AOP can be used to make prints to logs without touching the contents of the method. Then I remember that this was said somewhere in the mumbo. The original pattern - how to calculate how long it takes to execute a function/method - has been there for ages and in my books that is called *Just save current time in the beginning of the execution and decrease that in the end of the execution* -pattern. Maybe there is a cool acronym for this as well, but I'm this kind of pattern guy.

The dots are connecting. They talked about microservices in the interview and it is fintech, so most likely it is god damn crucial to know how long it takes something to execute. And because it is a microservice, the logs might not be in the same place. And because the fintech is complex there can be quite a many dependencies, calculations and points to monitor.

Now it is time to build something with this. I want to replicate similar environment. I don't care where else AOP can be used for now.

...*few moments later*...

The project is in [Github](https://github.com/spedepekka/soon-i-know-aop).

Couple of notes I want to write here for similar people to me:

* To track the times across multiple platforms the word is actually *trace*
* To trace something between multiple places or services one needs something to trace
* That something is just something unique like UUID...who would have thought :D
* That something is called sometimes correlation ID
* To get that UUID across services it must be attached to system and whatever call go between the services
* Correlation ID is usually transferred from place A to place B via HTTP headers with something like X-Correlation-ID
* If some sort of queue mechanism is used, then the correlation ID must be in the meta data

And now I have the basics for the logging thingamagick ready and it is just building from this moment on. It depends on the system how to calculate the times and there is no point to go there in this context. I'll just say that the timings can be written from the code itself or they can be calculated with external tools that scoop up the logs and visualise stuff on Grafana or whatever. When the values are there, it is easy to create sort of alarm mechanism to watch how the system operates.

Alright...now it is easier to learn what else can AOP do. Apparently it is used in transactional code as well like:

```java
@Around("execution(* com.example.aopDemo.service.*.*(..))")
public Object manageTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
    try {
        beginTransaction();
        Object result = joinPoint.proceed();  // Proceed with the method call
        commitTransaction();
        return result;
    } catch (Exception e) {
        rollbackTransaction();
        throw e;
    }
}
```

Now this is easy and readable. I think I cracked the gibberish and I don't need a dictionary to realize 'around' means 'wrapping around the method', instead 'somewhere around there'. This thing just pauses the execution of the method and one can continue executing it with `joinPoint.proceed()`, which I mistakenly read at some point as 'jointPoint' (that actually is quite close, but I had my knee joints in my mind). Even the 'join point' means the 'point to join "back" to original code'... like why was it so hard again to tell it simply in the beginning?

But seriously, it is really hard to explain things to other people when the concept is something abstract and the reader doesn't have anything real where to connect. So I'm not really saying that write better docs, because I don't know how anybody else could understand the way I learn. So maybe it is better to try to write the mumbo jumbo, because after all it will make much more sense - even for me.

Back to the example...there is some transaction going on, which can be anything. This example must be also quite used in fintech, because there are money transactions and most like a thousand other transactions going all over the place. Not to forget that in Spring there can be SQL-related transactions and of course these can overlap more or less in fintech context.

Oh yeah...I forgot to mention that AOP can keep the business logic clean, which should mean less bugs. One doesn't need to print timestamps to logs in the middle of some fancy state machine or similar full blown logical master piece. This might be actually one of the key motivations to create AOP, but I'm too lazy to check that from the docs.

## OpenAPI

There was a quick question about OpenAPI. In retrospective - All I can do is to laugh at my "I don't know what that is" answer. The truth is that it didn't ring any bells, but eventually it took a few seconds to figure out, because OpenAPI stuff is more basic stuff than AOP. I just have worked in projects where standards and definitions haven't been so high priority. So, to be honest...not knowing OpenAPI might tell more about the past projects than me.

I also said at some point that I would set up Swagger before doing any serious API-stuff. Swagger is not the same as OpenAPI, but they are somewhat related.

By basic stuff in this context I mean: OpenAPI makes sense and one can do many of the aspects of OpenAPI without realising the thing is part of OpenAPI ideology. For example one can write APIs in similar manner related to others and not scribble any word that comes to mind when writing the code or designing the API. Another thing is to make sure two components speak the same language with each other.

So I would define OpenAPI "Yet another tool to describe something (in this case HTTP API) to avoid broken connections and misconfigurations and providing easier testing". If we use that definition and ask the same question again, the answer would be something like "There have been numerous cases where I've configured/coded/defined a step in the pipeline or made programmatically sure something is like it should be."

So I kinda can say I've almost used OpenAPI, because it is so simple and straightforward, but I didn't know what OpenAPI stands for at the time.

## Afterwords

When I was writing this texts I figured out that here is a story to tell if I don't know something in the future in interviews. This post hopefully proves that I can learn something quickly and I'm good at connecting the dots, when there is something to connect to. Maybe people can get I bit better grasp who I am and who they are paying to do stuff.

There are other things one can do with AOP and my Github code for this project doesn't really cover much yet, but for this Friday this is enough.

It is hard to know as an interviewer who can deliver and who cannot, who fits the team and who doesn't. I don't know how it goes, but I know - I can deliver and fit.
