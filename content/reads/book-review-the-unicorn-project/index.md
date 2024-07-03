---
title: "Book Review: The Unicorn Project"
date: 2024-07-01T22:17:25+02:00
draft: false
tags: ["coffee-reads","book-review"]
---
![The Unicorn Project](cover.png)

## The Unicorn Project

Since Gene Kim just came out with a new book (topic for another coffee read) I realized I never summarized his last one. So for this coffee read I chose to focus on *The Unicorn Project*.

The book came out in 2019 and it is a fictionalized story about a DevOps transformation taking place at the same time as *The Phoenix Project* with the following premise:

{{< lead >}}
*In The Unicorn Project, we follow Maxine, a senior lead developer and architect, as she is exiled to the Phoenix Project, to the horror of her friends and colleagues, as punishment for contributing to a payroll outage. She tries to survive in what feels like a heartless and uncaring bureaucracy and to work within a system where no one can get anything done without endless committees, paperwork, and approvals. One day, she is approached by a ragtag bunch of misfits who say they want to overthrow the existing order, to liberate developers, to bring joy back to technology work, and to enable the business to win in a time of digital disruption. To her surprise, she finds herself drawn ever further into this movement, eventually becoming one of the leaders of the Rebellion, which puts her in the crosshairs of some familiar and very dangerous enemies. -- Gene Kim*
{{< /lead >}}


## Thoughts on the book

I have to start by saying I loved *The Phoenix Project*. So an updated version matching the new reality in the industry was something I was really excited about to read. With that book I related a lot, nodding and laughing along the way, recognizing the similarities between the fictitious company with its struggles and my own working environment.

### Highlights

What resonated with me greatly from *The Unicorn Project* book were the five ideals:

- Locality and Simplicity
- Focus, Flow and Joy
- Improvement of Daily Work
- Psychological Safety
- Customer Focus

I found that they complemented really well the three ways from *The Phoenix Project* which focused on flow, feedback, and continuous learning. This connection created a cohesive and comprehensive approach to improving work processes and achieving organizational goals.

The five ideals where expressed quite well by Maxine, the protagonist, when faced with various challenges:

> Punishing failure and “shooting the messenger” only cause people to hide their mistakes, and eventually, all desire to innovate is completely extinguished.

with a touch of humor:

> You know you’re in real trouble when even the intern feels sorry for you, Maxine thinks. She leaves the building, carrying it requires an understanding of the business, incredible problem-solving skills, patience, and the political sophistication to work with sometimes Byzantine and occasionally incomprehensible processes that every large organization seems to have.

> Technical debt is what you feel the next time you want to make a change.

And I was excited to find a book that highlighted the importance of having a good CI/CD process in the company and an underlying well maintained infrastructure. This definitely, for me, mattered to be spelled out after having to fight with the mentality that infrastructure and CI/CD is less important than building new and cool features for the product in my role.

> Without constant feedback from a centralized build, integration, and test system, they really have no idea what will happen when all their work is merged with everyone else’s.

> Everyone around here thinks features are important, because they can see them in their app, on the web page, or in the API. But no one seems to realize how important the build process is. Developers cannot be productive without a great build, integration, and test process.

> Without continuous builds, we are like a car manufacturer without an assembly line, where anyone can do whatever they want, independent of the plant goals(...).

And *The Unicorn Project* even went a step further in asking the questions that I feel are sometimes overlooked or simply dismissed during planning or design sessions, but are nonetheless critical to building reliable software.

> How many transactions per second are we expecting for product displays and orders? And how many transactions per second are the current builds capable of handling right now? That will tell us how many servers we need for the horizontally scalable portions, as well as how far we’re off for the vertically scaled components, like the database.

So Maxine (_Gene Kim_), thank you from the bottom of my DevOps heart!

Maxine, and the book, did not shy away from highlighting not only the technical challenges of a disfunctional process, but also the cultural ones as well. This was something that sometimes I feel is not talked about enough in the industry.

> Although long hours are sometimes glorified in popular culture, Maxine views them as a symptom of something going very wrong.

Seriously, yes.

> Is being assigned to the Phoenix Project making her a bad person?

I mean we all know that *one* project.

> In his inexperience, he had merely closed her ticket by following the rules set for him. He was only trying to do his job the best he’d been shown how.

> The leader must constantly model and coach and positively reinforce these desired behaviors every day. Psychological safety slips away so easily, like when the leader micromanages, can’t say ‘I don’t know,’ or acts like a know-it-all, pompous jackass. And it’s not just leaders, it’s also how one’s peers behave.

Yes, yes and yes.

Another great point of the book was the importance of doing employee training close to the customers. By training employees in environments where they directly interact with customers, they gain a deeper understanding of customer needs and challenges. This proximity allows them to tailor their responses more effectively, leading to better customer service, increased satisfaction, and more relevant and impactful solutions driving the business forward. But whether you are working in a plant or on software, this mentality is definitely helpful since the last thing you want is to work on software that no one will use or that is hard to use.

The overall message of *The Unicorn Project* book is inspiring, particularly its emphasis on trusting developers and treating them as equals. This approach fosters a collaborative and respectful work environment where developers feel valued and empowered. In such a setup, their potential for creativity and innovation is unlocked, driving advancements for the business. The book highlights the importance of trust, respect, and empowerment in maximizing the contributions of every team member.

### Areas for improvements

All of this said, I did find Maxine to be perfect. And not in a good way. While in the *Phoenix Project* we had multiple protagonists that ended up in situations that challenged their skills whether technical or not, in *The Unicorn Project* we have mainly Maxine that knows how to do everything - every - single - thing - while having a family at home. She's already a very good developer and the book shows us this by her re-writing code in several instances reducing it by hundreds of lines with other developers watching in amazement. And if she's not doing that, she's ready with quoting principles and books to show her extensive knowledge. She also works long hours, meeting her team at the bar as well, but it seems there are no consequences for her family life, which realistically, will not be the case.

And then there is her assitant Kurt who already had a vision for Maxine and where they should be heading, putting his job on the line without any reservations. The external coach Eric does not guide the reader to a conclusion of the five ideals, but rather he just tells them while popping in and out of his bar. I think the book should've let the reader or the protagonist reach his own conclusions, rather then telling him what it is. 

So from this point of view the characters didn't grow in my view as they didn't have any real challenges. I think to be a great professional or person, really, you don't need to be perfect. You should just be open to failure and willingness to learn, openess to the new and to change. Adapt. Failure, in my opinion, is a greater teacher than instant success. It makes you motivated, resiliant and ever evolving. So no, Maxine did not need to be perfect to show how good she is.

Lastly, then there were some aspects of it that were glossed over. For example, creating autonomous teams is no easy feat, but somehow the characters just managed to 'do it'. There were no challenges there, everything just fit. And in my own experience this is not the case. Breaking down teams and making them efficient, while maintaining alignment with the business goals and a steady flow of seamless releases is always filled with hurdles and challenges, especially at the scale that Parts Unlimited, the fictious company, operates.

## Conclusion

Definitely if you read the book, that came out in 2019, now, in 2024, it will feel a bit outdated already and perhaps even in 2019 it was already a bit. Overall *The Unicorn Project*, in my opinion, did not match its predecessor, even though it had a lot of potential from the premise and the new realities in the industry. That said, the five principles are definitely still relevant and the book is very good at showcasing why these are needed so this still makes for a good read.

I have a lot of respect for the author, Gene Kim, and look forward to reading his next book!

*Hope you enjoyed this review and if you read the book let me know what your thoughts on it are!*

Enjoy your coffee! ☕️
