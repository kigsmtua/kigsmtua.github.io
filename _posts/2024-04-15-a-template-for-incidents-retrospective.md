---
layout: post
title: Handling production incidents for software teams a process
date: 2023-10-31 10:00:00 +0300
description: Streamlining the Process of Handling Production Incidents for Software Teams
img: software.jpg
tags: [on-call, incidents, processes]
---
<!-- The content of this blog  -->

Incidents are part of running software in production. It's how we handle them, learn from them and recover from them and prevent them from happening again that we will try and address in this article. I'll try and use my experience to define a process that looks sensible, a template for retrospective and how to run the retrospective meetings so as to obtain as much value as possible from the incident.

This article only covers an internal process, external communication to stakeholders outside the company will be tackled by another article.

I  will try and look at two parts in this article, mostly focussing on the retrospective aspect of the incident.

1.  **During the incident**
2.  **A retrospective process**


**During the incident**

During an incident, two key aspects are immediately essential: communication and resolution. We appoint two individuals:

* Communication Lead:
This individual is responsible for updating stakeholders, including internal teams and external parties if necessary, on the incident's progress. Clear and timely communication helps manage expectations and reduces confusion

* Resolution Lead:
This role involves identifying the root cause of the incident, coordinating technical efforts to resolve it, and ensuring that all necessary actions are taken promptly. The resolution lead may also work closely with other teams such as development, operations, and security to address the issue comprehensively.


Our communication strategy would be as below. We want to have two separate communication places.
1. A dedicated incident channel where we can have our warroom and technical discussions
2. An public channel where we can share periodic updates on the incident. For this communication channel we try and keep it about only updates, no technical jargon or discussion should happen here. Regular updates ideally every 20-30 minutes should be done here.

A timeline for this would look like this

* 12:40: Issue identified; team working on resolution.
* 13:10: Still investigating the problem; no ETA yet.
* 13:40: Root cause identified; resolution plan in progress.
* 14:20: Ongoing resolution efforts; ETA for resolution provided.

We try to keep this as technical free as possible so all parties can follow as some parties are just interested in resolution

**Retrospective**

After we have everything up and running its time to recollect and take stock of what happened and this is the retrospective the most important part of any incident I would argue. To make this a bit easier below is a template on what we should do to make sure we collect enough information.

All people involved in the incident, should participate in filling out the template below  so we can capture as much information as possible and have different perespectives onto what happened and how we can prevent it from happening again.

Our templates has a couple of parts:

1. Introduction

We describe the incident and what happened, what we found to be the issue and how we went ahead to the resolution

2. Timeline

This will detail all individual timings as to when each action was taken, what action was taken basically what each and every minute of the incident was spent. In this maybe we will uncover some deficiencies in how we handled the incident as we do an analysis later on

3. The five whys

The 5 whys is a  good technique for identifying the root cause of an incident, as we ask questions that lead us in the right path until we come up with a solution. A detailed explanation of the 5 whys can be found here https://www.mindtools.com/a3mi00v/5-whys

Start with the immediate cause (e.g., database instance failure) and ask "why" multiple times to uncover underlying issues. This method helps move beyond surface-level explanations and identify systemic issues that need addressing.


4. Actions

Now that we know what the root cause of our problem is we can define Actions that need to be taken. We have immediate actions, which should be taken and tackled immediately long term actions which we should definetely plan for and see how they will be done.


Now that we have a template, we should aim to fill this out alteast a couple of hours after the incident has occured. All people actively involved in the incident should contribute to filling out our template so as we can have a better and cleaner picture of what happened

Having all incidents in a similar template, means we can have a repository and I would be able to go back and look at what was done for another incident for example, identify trends and we can be able to look after our production systems better. Remember the goal is learning and being able to prevent this issues from happening again.


After our template is ready, the next step is to share it with the rest of the engineering teams. I would suggest a meeting where we can get a couple of relevant stakeholders and go through our Retrospective.This allows for the rest of the team to question, critique and even offer suggestions on things we may have missed during our tackling of the incident and important step in ensuring everybody is involved in the reliablity of the systems we build.

congratulations, you have defined a process for dealing with production incidents.

<b>P.S</b> try out my tool [link](https://buggie.io/ "Buggie").  an tool for managing on-call, handovers and incidents on slack, we will be adding this template and process (customizable ofcourse) in the next release of the app. Hope you enjoy reading it.
