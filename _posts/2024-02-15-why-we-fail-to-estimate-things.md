---
layout: post
title: "Accidental & Essential Complexity, Hofstadter's and Parkinson's laws and why we fail to estimate things"
date: 2024-02-15 13:00:00
categories: [article]
tags: [article]
---

## Introduction

Working with a team, tribe, squad, or whatever your company calls it always involves delivering something: software, infrastructure, or an environment. To accomplish this, we often need to navigate through uncertainty and attempt to estimate delivery dates. However, it's not uncommon for us to fail completely and miserably at this task. This post aims to discuss the reasons behind such failures.

There are numerous factors that could contribute to our failures: ourselves, team leaders, the system, the government, or perhaps even the universe? The truth is, as human beings, we are inherently bad at estimating things accurately, particularly our own capabilities in creating, building, or delivering something.

Even with a multidisciplinary team composed of seniors, specialists, and guided by a Project Manager, we still tend to encounter failures. Once again, I believe this is an inherent human weakness.

And is totally ok.

Estimating the time required for software development is notoriously unreliable, and in a DevOps environment, it can be even more challenging. I like to ponder whether a paper written by Fred Brooks in 1986 might provide insights into this issue. The paper, titled "No Silver Bullet," delves into the concepts of accidental complexity and essential complexity.

## Accidental Complexity

Accidental Complexity refers to the challenges in software development that arise from the programming environment or the tools used. These are problems that could be resolved or eased with the evolution of technology, better practices, and more efficient tools.

## Essential Complexity

Essential Complexity relates to the inherent difficulties in the nature of the software itself. These challenges revolve around managing complexity, handling changes, ensuring compliance, and dealing with the inherent invisibility of the software, irrespective of the tools or technologies employed.

## Accidental and Essential Complexity in a DevOps Context

## Accidental Complexity in Devops Context

DevOps and related tools are relatively new.

New tools, technologies, and frameworks emerge daily, while established technologies continually evolve. This constant flux inherently adds complexity and cognitive load, resulting in inaccurate estimation plans.

Hidden tasks often lurk within seemingly simple tasks. Understanding what needs to be done, planning task execution, and uncovering implicit sub-tasks can be akin to feeding or wetting a gremlin after midnight.

## Essential Complexity in Devops Context

Essential Complexity is Inherent DevOps Complexity itself

Before we proceed, let's briefly enumerate the myriad elements surrounding a simple web server hosted on an AWS EC2 instance:

- Electrical connections
- Physical hardware
- Physical networking
- Logical networking
- Storage
- Operating system
- Web server software
- Static content
- Internet connection

While it may be unnecessary for an ordinary user to possess in-depth knowledge of all these points, someone within a cloud provider must be well-versed in all of those disciplines, or even more so, whether for troubleshooting, improvement purposes, new deployments, and so on.

Managing a simple or complex ecosystem typically entails dealing with numerous other complex components.

As a complement to the concepts of accidental and essential complexity, it's beneficial to consider Hofstadter's and Parkinson's Laws.

## Hofstadter’s Law

Hofstadter's Law asserts that a project always takes longer than expected, even when accounting for the law itself. In essence, time estimates for project completion consistently underestimate the actual time required.

Hofstadter's Law highlights a planning fallacy, wherein we tend to underestimate the time needed for any given task. This tendency leads to frequent failures in project time estimates, even with well-managed projects. Actual durations almost invariably surpass even the most pessimistic estimates, including those adjusted to accommodate Hofstadter's Law.

## Parkinson’s Law

Parkinson’s Law posits that "work expands to fill the time available for its completion." If a task must be completed within a year, it will take a year; if within six months, then six months it shall be.

Parkinson’s Law serves as both a caution against procrastination and a reminder to better estimate tasks. Meanwhile, Hofstadter’s Law acts as a gravitational force, inexorably pushing us toward failure—an intriguing balance between these two phenomena.

## Avoidance Strategies

Sadly, there's no Silver Bullet, as mentioned earlier. However, a combination of personal time management techniques and a simple enterprise management framework might aid in better estimation. 

Some strategies include:

Enterprise Perspective: Shape Up

"Shape Up" is a guide to Basecamp's product development methodology, divided into three main sections: Shaping, Betting, and Building. Shaping focuses on pre-project work, defining solutions and risks, while Betting deals with project selection decisions within six-week cycles. Building covers project team practices and progress tracking.

Personal Perspective: Managing Daily Tasks

Regardless of whether a task is planned, unplanned, or personal, a task is a task. Keeping things simple is always a better starting point. For this reason, a list can be a powerful tool.

At the start of each day, it could be a good approach to divide the work day into time slots and list everything that needs to be done, including daily tasks, meetings, or even bill payments. These time slots can be used initially to prioritize activities and for executing tasks, attending meetings, studying, handling unplanned events, or even taking a break.

As a result, even if something goes down in a production environment, the daily tasks have a slot for unplanned things, providing a little bit of order in the chaos.

## Conclusion

As new systems, technologies, and tools emerge, complexity increases exponentially, whether it's accidental or essential. Understanding this complexity is crucial, and devising effective mechanisms to manage it is imperative. Only through this understanding and proactive approach can we hope to improve—or at least mitigate our shortcomings—in estimating tasks and projects accurately.

Furthermore, it's very important to recognize the inevitability of complexity and uncertainty in software development and correlated components. Rather than fearing them, we should see them as opportunities for growth and innovation. By understanding and accepting the dynamic and unpredictable nature of the work we do, we can become more agile and resilient in the face of challenges.

Ultimately, improving our estimation skills is not just about techniques and tools, but also about a fundamental shift in our mindset and approach to work. By embracing complexity and adopting a posture of adaptation and continuous learning, we'll be in a better position to handle whatever challenges come our way and succeed in what we're trying to do, in a professional or personal perspective.

## References

[No silver bullet](https://worrydream.com/refs/Brooks_1986_-_No_Silver_Bullet.pdf)

[Hofstadter's Law](https://www.techtarget.com/whatis/definition/Hofstadters-law)

[Parkinson's Law](https://personalmba.com/parkinsons-law/)

[Shape Up](https://basecamp.com/shapeup)

## Any sugests or doubt? 

Feel free to reach out to me on social media: [twitter](https://twitter.com/rmnobarra)
,[linkedin](https://www.linkedin.com/in/rmnobarra/) and [github](https://github.com/rmnobarra).

You can also email me directly at rmnobarra@gmail.com. 

## Support

Did you really enjoy my content? Consider buying me a coffee through my Bitcoins wallets: 

![Donate with Bitcoin](https://img.shields.io/badge/Donate%20with-Bitcoin-orange)

Bitcoin Wallet:

`bc1quuv5hml9fjkf7azgwkt4xp867pzdwzyga33qmj`

![Bitcoin wallet QRCODE](https://rmnobarradev.blob.core.windows.net/rmnobarradev/bItcoin-address.png)

![Donate through Lightning Network](https://img.shields.io/badge/Donate%20with-Lighting-blue)

Lighting Address: 

`lnbc1pjue6mkpp5yj737e7fm6efhlj6sns42a875pmkencqmvdshf4ghlnntaet5llsdqqcqzzsxqrrsssp5d9hxl686w839qkwmkm0m30cf5gp4gnzxh68kss393xqzlsg0wr3q9qyyssqp3933zc3fg46nk3vafh63r3lqd0jn2p04w5xrz77h33rrr3xm7ckegg6s2ss64g4rf4jg87zdjzkl5tup7umqpghy2qnk65tqzsnplcpwv6z4c`

![Lighting wallet QRCODE](https://rmnobarradev.blob.core.windows.net/rmnobarradev/lighting-address.png)

Bye!