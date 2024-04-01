---
layout: post
title: "Broken Windows Theory and Why Every SRE Should Learn From It"
date: 2024-03-27 10:00:00
categories: [article, theory]
tags: [article, theory]
---

## Introduction

The daily life of an SRE in a heterogeneous environment, with various architectures to be designed, maintained, and improved, in addition to occasional war rooms and hidden tasks, has its ups and downs and many surprises. After all, one certainty we have is that every system will fail eventually.

Without appearing stoic, but already being so, there are basically two dynamics at the table: events we can control and events we cannot control. In a technological environment, prevention and prediction can often control, in a way, the uncontrollable. Here, we can learn from the broken windows theory.

![broken window](https://rmnobarradev.blob.core.windows.net/rmnobarradev/broken_window.png)

## The Root of the Theory and Its Relevance

The Broken Windows Theory, proposed by James Q. Wilson and George L. Kelling in the 1980s, suggests that visible signs of disorder and neglect in an environment encourage disorderly and criminal behaviors, perpetuating a vicious cycle of urban decay. This theory was initially applied to the field of criminology, where the presence of minor crimes, like broken windows and dirty walls, in urban neighborhoods contributed to an increase in crime.

Transposing this theory to the SRE context, we can understand "broken windows" as unmanaged technical debt, inadequate coding practices, poor monitoring, and a general lack of adherence to best practices. Just like in the original theory, the presence of these "broken windows" in an IT environment signals an acceptance of chaos, discouraging the maintenance of quality standards and encouraging the accumulation of more problems.

## Technical Debt: The Primary "Broken Window"

At the heart of many chaotic IT environments lies technical debt – the decision to postpone ideal solutions in favor of quick and temporary fixes. The infamous "temporary permanent."

Although accumulating a certain level of technical debt might sometimes be a strategic necessity, failing to manage it properly can lead to an environment where the cost of adding new features or maintaining the system becomes prohibitive. I like to think of technical debt as a boomerang: the further you throw it, it may take time, but inexorably, it will come back to you.

In DevOps and SRE, proactive management of technical debt is not just a recommended practice; it's essential for the sustainability of the project and should always be considered during activity planning.

## Monitoring and Alerting: Vigilance Against Chaos

A robust and effective monitoring system is the sentinel that prevents small cracks from turning into catastrophic ruptures. Deficient monitoring systems are like neglected windows, through which minor issues can evolve into significant service disruptions. Worse, monitoring systems usually have 2 variants.

The one that monitors everything and monitors nothing: This refers to the "select all features" of the monitoring system, with thousands of entities, dashboards, charts, which in the end serves no purpose.

The always red: The monitoring system where red is the standard, where alerts are known by name and surname and usually celebrate their anniversary.

To remedy this, making use of effective monitoring and alerting practices not only detects problems in real-time but also provides insights for proactive failure prevention, helping to maintain the system's integrity and resilience.

## Best Practices: The Antidote to Chaos

Adopting best practices is crucial for maintaining order and efficiency.

This includes, but is not limited to, continuous integration (CI), continuous delivery (CD), infrastructure as code (IaC), test automation, and rigorous code reviews. These practices are not just preventive measures against the accumulation of technical debt; they also encourage a culture of excellence and accountability, where quality is prioritized.

Every technology professional should bear in mind that quality is non-negotiable.

## Culture and Communication: Pillars of Support

Finally, the strength of any strategy lies in the team's culture and the effectiveness of communication. A culture that values transparency, continuous learning, and continuous improvement is essential to prevent the emergence of "broken windows."

Failing and failing fast and learning from mistakes should also be intrinsic to a company with a mature culture, coupled with blameless behavior, where understanding what caused a particular failure and acting to prevent it from happening again is the rule, not the exception.

In conclusion, effective communication, both internally and externally, ensures that everyone is aligned with the goals and expectations, facilitating the quick identification and resolution of problems.

## Conclusion

Applying the Broken Windows Theory to the world of SRE is not just a useful metaphor but a critical reminder that excellence in IT operations is built on constant attention to detail and a refusal to accept complacency.

Preventing and caring for the small "disorders" in our day-to-day operations not only prevents the accumulation of larger problems but also cultivates an environment where innovation, stability, and efficiency can thrive. Again, no matter how strongly we throw the boomerang, sooner or later, it comes back.

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