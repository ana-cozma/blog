---
title: "Book Review: Software Architecture Patterns (Report)"
date: 2023-10-04T14:19:55+02:00
draft: false
tags: ["coffee-reads","book-review"]
---

![Software Architecture Patterns (Report)](cover.png)

For this *coffee read*, I will be reviewing the book [Software Architecture Patterns (Report) by Mark Richards](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/).

I want to start with a disclaimer on this one. I have not worked as a software architect. So this book review will be from the perspective of someone that has worked as a DevOps engineer (more of a Cloud Engineer if we have to label it), and that has been involved in the design of systems and the architecture of systems. My personal belief is you need to have a good high-level view of the system you are working on, regardless of the project or your skillset (or job title), in order to be able to design or test solutions that make it scalable, reliable, and maintainable. 

My hopes, from reading this book, were to get a better understanding of the different architecture patterns that are out there, how they can be used to design systems, what are some things to look out for when choosing one or the other or a combination of them and what would be some good questions to ask yourself when opting between them.

## Software Architecture Patterns (Report)

The book, in its own description, emphasizes the importance of having a clear and well-defined architecture in software development. Without it, developers tend to default to traditional layered patterns, which can result in disorganized code known as the "big ball of mud" anti-pattern. Applications lacking a formal architecture are typically tightly coupled, difficult to change, and lack clarity in terms of their characteristics.

Having a defined architecture pattern helps establish the fundamental characteristics and behavior of an application. Different architecture patterns are suitable for different needs, such as scalability or agility. Understanding the strengths and weaknesses of each pattern is essential to choosing the one that aligns with specific business requirements.

The goal of the book, as mentioned by the author, is to provide architects with the information needed to make and justify these decisions effectively (hence my disclaimer).

## Thoughts on the book

The book is broken down into the following chapters: 
- _Layered Architecture_
- _Event-Driven Architecture_
- _Microkernel Architecture_
- _Microservices Architecture_ 
- _Space-Based Architecture_

Each type is nicely broken down into sections that explain the pattern description, key concepts, pattern examples, consideration, and pattern analysis.

The book also provides a summary of the different patterns in terms of their characteristics, strengths, weaknesses, and when to use them in real-life scenarios.

Each chapter starts with a summary of what the architectural pattern is and what are the key concepts behind it. It then goes into detail about the pattern (including design samples) and how it can be used by providing examples of real-life scenarios. For example, for the Event-Driven Architecture pattern, the author explains the mediator and broker topologies and how they can be used to implement the pattern before delving into the details of the pattern itself.

Next, it highlights the considerations that need to be taken into account when choosing that particular architecture. For example, in the Event-Driven Architecture pattern, the author explains that one of the challenges of this architecture pattern is the lack of atomic transactions for a single business process. And this is the reason why when using this pattern you need to continuously think about which events can and cannot run independently and plan the granularity of your event processors accordingly.

Finally, it provides an analysis of the pattern in terms of **Overall agility**, **Ease of deployment**, **Testability**, **Performance**, **Scalability**, and **Ease of development**. The author gives a rating from Low to High on each of these attributes when considering the architecture pattern and an analysis as to why it scored that way. This really made me understand why one pattern would be better than another in a specific scenario. Or at least what issues you can run into when choosing it. For example, Microservices pattern is rated as **High** in terms of **Overall agility** and **Ease of deployment** but **Low** in terms of **Performance**. This is because of the distributed nature of the pattern, even though you can most definitely create applications that are performant using this pattern.

It was easy to understand the concepts and follow the examples, and the analysis of the patterns was very useful in understanding the strengths and weaknesses of each pattern.

This analysis also guided me in understanding the trade-offs associated with choosing one pattern over another. It provided valuable insights into the kinds of questions that should be considered when making such a decision, whether individually or within a team context. Moreover, it encouraged a shift in my perspective from seeking the 'perfect' pattern to identifying the pattern that aligns best with the specific requirements of the system.

As a result, this change in mindset aided me in asking pertinent questions when designing a system, with a strong focus on the system's characteristics and the demands of the business, ultimately contributing to more informed and effective decision-making.

## Summary

This book was a very good read as it achieved what it set out to do.

I would recommend it as a very good starting point for anyone interested in learning more about different software architecture patterns. And you have the option of diving deeper into the patterns that are of interest to them in other books from this authos or others.

If you've read the book, let me know what you thought about it in the comments below. If you have any recommendations for other books on the topic of software architecture, I would love to hear them.

__Enjoy your coffee!__ ☕️
