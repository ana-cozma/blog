---
title: "Navigating the Trenches: A Guide to Effortlessly Handling Customer Support Tickets"
date: 2023-01-17T09:16:45+01:00
draft: true
tags: ["coffee-reads", "sre", "on-call"]
---

If you work with real data and customers you will have to deal with customer support tickets. This is a very important part of your business and you should take it seriously. In this article, I will give my lessons learned and some tips on how to handle customer support tickets.

### What is a customer support ticket?
A customer support ticket is a question, a bug report, a feature request or a complaint coming from a customer of your product. It is a communication channel between you and your customer. These complaints are logged in a ticketing tracking system such as JIRA. Generally, the expectation is to handle these tickets in a timely manner and to make sure that your customer is happy.

More often than not, they are given priority and you have to handle them in order of priority. You might even have SLAs tied to them and that means based on the priority you have to handle them within a certain time frame.

### Why is it important?

This is from the customer's point of view, but customer support tickets can also be a distraction to your software development team, adding cognitive load to your team and slowing down the development process. If not handled correctly, they can also be a source of frustration for your team. And even lead to burnout.

This can be avoided by having a good process in place and by having a good understanding of the customer's needs.

The following are some of the Do's and Don'ts that I have learned over the years while navigating the trenches of customer support tickets and having experienced a lot of pain myself.

## Do's

### 1. Do have a correct prioritization of tickets

The first thing you need to do is to have a correct prioritization of tickets. This is very important because it will help you to focus on the most important tickets and to avoid distractions. When the workload is high, asking a team to focus on the lowest-priority tickets is a recipe for disaster.

But understanding what makes a ticket a P1 (Priority 1 - should be taken immediately) or a P4 (Can wait 2-3 days) needs to be understood by all teams working with the ticketing system and the ticket itself. This means different things for the stakeholders involved in the ticket:
- **customer support** : they are assessing the impact of the issue on the customer, they need to understand clearly how the customer is impacted and what is the urgency of the ticket
- **software development** : they need to understand the impact of the issue on the product and what time frame they have to fix it
- **product management** : they need to make sure the SLAs are met and that the customer is happy

**Tips** on how you can achieve this:
- Have a clear internal definition of what a P1, P2, P3, P4 ticket is
- Have the definitions clearly defined in a document and make sure that everyone is aware of it and is easily accessible (make sure you are not relying on tribal knowledge).
- Provide training often to make sure that everyone is on the same page

### 2. Do have a clear escalation process in place

This is two-fold.
From the _organization's point of view_ have a clear process in place on how to escalate tickets, who is responsible and who is the next person in contact. Make sure these roles are clearly defined and visible to everyone in the company.

If customer support people are tagging teams in the ticket, make sure that the team's competency is clear and the boundaries are set and all parties responsible are aware of it. If unsure, don't hesitate to ask the team directly. Be careful: you are asking about the team's domain, not help on the issue itself!

Otherwise, you will end up with a lot of back-and-forth between teams, losing time that could've been put into investigating the issue and fixing it. Moreover, you might end up wasting unnecessary time with the wrong team causing frustration and eventually a lack of trust and cooperation between the teams. Everyone has tasks to do and they need to be able to focus on them. So being mindful of their time is basic respect.

From the _software team's point of view_ create an escalation process in the team as well. 
**Tips** on how to achieve this:
- Divide the tickets among the team
- Create a rotation and make sure that everyone is aware of it
- Make sure there is an escalation process inside the team as well: juniors take easy tickets, quick wins, etc. and seniors take the more complex ones, when in doubt they can ask for help from the seniors. Same for the seniors, ask for help from the lead or the point of contact if you are unsure of how to proceed, if there is a delay in fixing the issue, if there needs to be communication with the customer, etc.

### 3. Do create an internal knowledge base

I cannot stress how important this is. You need to have a place where you can store all the knowledge you have gathered over the years. This is a very important source of information and it will help you to avoid repeating the same mistakes over and over again. It will also help you to avoid having to rely on tribal knowledge.

**Tip** It doesn't matter where you have it: Confluence pages, a Google Doc, a wiki, a GitHub repo, a Notion page, workbooks etc. The important thing is that you have it _written down_, it's _up-to-date_ and you are using it.

This will also help with the onboarding process. You can slowly introduce new members of the team or junior members of the team to the knowledge base and they will be able to learn from it and handle low-level tickets on their own. This also means you are freeing up your more senior members of the team to focus on more complex issues. Win-win!

**Tip** In order to achieve this, you need _team buy-in and commitment to growth_. As a team lead or manager, you need to make sure that everyone is aware of the importance of this and that they are willing to contribute to it for the betterment of the team. You need to make sure that everyone is aware of the knowledge base and that they are using it. You need to make sure that everyone is aware of the importance of updating it.

If there is one thing you do, do this with your software development team. It will save you a lot of time and frustration.

### 4. Do check for patterns in the tickets

Have you seen an increase in the number of tickets related to a specific feature, browser, OS, location, etc? If the answer is yes, then you need to evaluate and investigate why this is happening. It could be a bug, a regression, a change in the product, a change in the customer's environment, a change in the customer's behavior, etc.

It can even be an issue that has not manifested itself as a failure yet, but there is a potential for it. As an SRE, you need to be aware of this and be proactive in preventing it. This is a good opportunity to improve the product and to make it more resilient.

It can also mean the users are changing the way they use the product: either they find the functionality hard to use, don't understand it or they are using it in a way that was not intended. This is a good opportunity to improve the product and to make it more user-friendly.

**Tip** Review them as a team and make a resolution: dismiss it as a one-off, investigate it, fix it, improve the product, etc. And set ownership: there is no point in giving a user behavior change pattern to an engineer, let the product owner or the product manager investigate it. Keep it divided and efficient. You get my point.

## Don'ts

### 1. Don't rely on tribal knowledge

Tribal knowledge is a very dangerous and fragile thing. It is also a very toxic thing because well check the next point on this one. It is a very bad thing. How can I reiterate this enough?

You will lose so much time trying to figure out who worked on a similar issue, what was the cause, and how did they fix it. And you will never learn from it because once the person who worked on it leaves, you will lose all that knowledge. So instead of learning from it, you will have to fix the same issue over and over again. That's just running in circles.

### 2. Don't have a single point of failure/ single person responsible for handling tickets

If you use tribal knowledge, you will end up with a single point of failure.

We LOVE heroes. But unless it's Scarlet Witch, Black Widow or Ironman (yes, I know), heroes have no place in a team in my opinion.

What a "hero" will do is fix issues without disclosing the root cause or sharing the knowledge with the rest of the team. This is a very bad practice. What if the person leaves the company? What if the person is on vacation? What if the person is sick? What if the person is overloaded with tickets and cannot handle them all? What if the person is not available for some reason?

Assuming the person is not a 'hero' and is just a hard worker, it will also lead to a lot of frustration and eventually burnout for the person that will be contacted all the time. We cannot expect people to give 100% when we flood them with tickets and we don't give them the tools to succeed.

**Tip** When we see an engineer brilliant at debugging complex systems, the tendency is to give them more tickets to handle because they can 'fix it fast'. Don't. That person is a _precious resource that should be given only the most complex tickets_. Leave the groundwork to the rest of the team. This will also help with the onboarding process and will help the team to grow.

### 3. Don't let customer support people assume the issue in the ticket is the root cause

Alrighty! Customer support people are not technical people. This is not a bad thing, their biggest asset is communicating with the clients. They are the ones that are in direct contact with the clients and they are the ones that know the clients the best. They are the ones that can provide the most valuable information about the issue.

The tempting thing for them to do is to not describe the issue in the ticket, what the user is seeing, but, based on similarity to other tickets fixed, assume it has the same root cause and write that one down as the issue the user is facing.

This can actually cause more confusion for the engineering team because you are sending them on a different path to the issue. 

Keep it simple: describe the issue with as many details as you can, add steps to reproduce, add screenshots, add logs, and add anything that can help the engineering team to understand the issue and to fix it. And stop, don't assume anything. Trust the engineering team to figure it out.

**Tip:** A good practice to prevent this is to have a template for the tickets. This will help the customer support people provide all the necessary information and it will help the engineering team to understand the issue.

### 4. Don't ignore patterns in the customer support tickets

They will come and haunt you. You will have to deal with them eventually.

It can come back as anything from a system failure to losing business on the platform because the users find it too hard to use. It can also come back as a loss of reputation because the users are not happy with the product.

## Conclusion

Have simple, clear processes and procedures in place: organization level and team level.
Share the responsibilities and the ownership of the tickets.
Have a knowledge base.
Check for patterns in the tickets.
Train your teams - often until they are comfortable.

_Hope this helps! Let me know what you think in the comments below._
