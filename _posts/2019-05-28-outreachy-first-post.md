---
layout: post
title: A beginner's journey to Outreachy at Openstack
date: 2019-05-28 13:36
categories: outreachy openstack
---

This is the first blog post that I have ever written in the history of the universe! \o/ (Much elation)

I had been procrastinating on starting a blog for quite some time now. Thanks to Outreachy, I got the final push to do it.

Before I begin, I would like to announce that I landed the Outreachy internship at Openstack a few weeks ago. I will be working with their Team Ironic this summer on the project 'Extend sushy to support RAID for Redfish'.

First, a bit background: 
>Outreachy provides paid internships to work in Free and Open Source Software (FOSS) and is open to applicants all around the world. Interns work remotely with experienced mentors from FOSS communities and the projects may include programming, user experience, documentation, illustration, graphical design or data science.

I came to know about Outreachy two years ago via twitter. I made my first contribution to open source because of Outreachy when I tried applying for the winter round in 2017 (had to back out due to some eligibility issues). Honestly, I got a bit dejected back then but now, when I look back, I can see how the sole experience of just applying and contributing transformed me. It was because I tried applying to Outreachy that I was introduced to this vast world of FOSS. I learnt so much about git, CI testing, coverage and communicating across timezones. Contributing to open source gave me the feeling that I am a part of something much bigger than my own self. And it got me so hooked that I could not stop after that and kept on exploring and contributing to the repositories of different organizations in bits and pieces. 

I again decided to apply for Outreachy this year for the summer round. When the applications opened in February, I browsed through the list of projects and shortlisted the ones which matched with the tech stack that I was acquanited/comfortable with and the level of skill that was required. I was more inclined to work on projects involving cloud computing and distributed systems since it was something that had been intriguing me for the past few months and hence, I was very eager to learn more about it. Going ahead, I read Openstack's project description and the details about the work to be done during the internship and honestly, I could not understand most of what was stated there. Because it sounded so challenging in my head, I knew that this project will offer me a lot to learn and so, I took the opportunity. I prepared myself for a lot of reading, watching videos and googling ahead.

### IRC
As mentioned in the project description, I looked up the IRC channel and started lurking on their #openstack-ironic channel for the days to come. For registering my nick to access the channel, I found the [link](https://freenode.net/kb/answer/registration) quite helpful. One can even read the past IRC logs of the channel [here](http://eavesdrop.openstack.org/irclogs/%23openstack-ironic/).

### Some research
Next up, I started with googling every word that I did not understand in the project description and reading all the related documentation about the project that I could manage to find. I even watched some videos/webcasts[^1] which were quite helpful. 

### Project setup and exploring bugs
Openstack uses gerrit for their code review processes. The docs[^2] for setting it up were quite straightforward and easy to follow. The bugs are recorded on the [storyboard](http://storyboard.openstack.org/). We were advised to look up for the outreachy tagged stories.

### First patch
After setting up the sushy code base on my machine, I started fiddling with the code and exploring it to get a bit overview of how things were happening. While going through the code, I came across a `todo` that sounded quite easy to me and so, I went ahead by submitting a story and a patch for it via git-review. I learnt a lot of things while making my first contribution in regards to the tox tests that need to be run locally before submitting the patch, the 80-character line limit, adding a release note[^3] and pointers regarding how to communicate effectively on the patch.
The Ironic community members were extremely helpful and patient with all the silly mistakes that I made in the beginning. Moreover, they give very detailed reviews on what needs to be changed and why, so you get more clarity with each patch that you submit for review. 

### Susequent patches
After submitting the first patch, the process of exploring additional bugs/stories and submitting patches became much much easier. The patches that I submitted and got merged can be found [here](https://review.opendev.org/#/q/owner:%22Varsha+Verma%22). Of course, you have to keep reading and exploring, but you kinda get a hang of the sequence of the steps that need to be undertaken with time. The greatest challenge, I believe, lies in the overcoming your initial apprehensions about the project while figuring out where to look for what. In case, the things seem a bit difficult/stuck, you should never shy away from asking questions on the IRC channel. The folks out there are really friendly and irrespective of your timezone, someone will, more or less, always be there to answer your queries and help you out.

### Final application and thereafter
For the final application on the Outreachy portal, you are required to answer a few questions about your past projects, explain why you took up that particular oragnization and give a detailed timeline of how you intend to work during the course of the internship period. 
I continued making contributions after submitting the final application, although, I did slow down a bit. 

My mentors [Dmitry Tantsur](https://twitter.com/creepy_owlet) and Ilya Etingof were extremely helpful during this period. They helped me fill up the timeline and also improve my final application for the project. Even while making the contributions, they were highly supportive and helped me make the most out of each patch that I tried to submit. I am also very thankful to the other Ironic community members [Riccardo Pittau](https://twitter.com/elfosardo), [Julia Kreger](https://twitter.com/ashinclouds) and the other folks at Ironic for all their support during the contribution process.

A big advice that I would like to give to all the beginners out there, with a thirst for knowledge and looking for an opportunity to quench their thirst, is that, "YES, YOU CAN!". Six months ago, I wasn't even clear on what the term 'Cloud Computing' actually meant, yet here we are. :)

All that is needed is a strong will to learn and an even stronger will to work hard and push yourself. All the factors other than that can very easily be taken care of, given how loving and warm the FOSS communities are to newcomers.

So, Good Luck and Happy Coding!

---

[^1]: Empowering ironic with Redfish support - [https://www.youtube.com/watch?v=KxRo5cRpj6k](https://www.youtube.com/watch?v=KxRo5cRpj6k)

[^2]: Openstack Docs: Developer's Guide - [https://docs.openstack.org/infra/manual/developers.html](https://docs.openstack.org/infra/manual/developers.html)

[^3]: Adding a release note to one's patch - [https://docs.openstack.org/ironic/latest/contributor/faq.html#create-a-new-release-note](https://docs.openstack.org/ironic/latest/contributor/faq.html#create-a-new-release-note)
   
