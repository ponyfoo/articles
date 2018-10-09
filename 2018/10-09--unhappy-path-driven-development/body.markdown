[This story][story] *--- about all the things that could, and do, go wrong in software design and development ---* is an incredibly important cautionary tale about the work we do in our field, and our role in it.

Read it thoroughly and always from the perspective of how, given the domain, the product failed its us. Customizability breeds problems because it's the opposite of enforcing constraints, which are fundamental to robust system design. Without constraints, it becomes difficult to scope the impact of changes, to understand how different consumers use our products, and thus, to build.

Repeating information under the format, over and over, dilutes its meaning. Imagine if instead, the application displayed an image of the medicine and maybe even a cute visualization of the pills the hospital would be administering. It's the same repetition, but easier to digest.

The article talks about the perils of mode switching (think the Caps Lock key and password inputs, or in their case: dosage units). When you're in a mode, the application should make it obvious that that's your life now. Otherwise, chaos ensues. *(oh hi vim!)*

A perhaps even more crucial-to-get-right mode is failure. The product indeed had warnings about the case where an overdose is ordered. Along with just about everything else. In a month, systems for a hospital like the one in the article dump over 381,560 such alerts on the staff.

It's easy to blame the user. They ignored the warnings, repeatedly. Or the induction process where staff learns early on to ignore every warning the system lobs their way. And for good reason: they wouldn't be able to do their work otherwise. The user is hardly ever to blame.

The problem here, as is often the case in it industry, is that we want so hard to believe the "happy path" is what's most important. After all, it's what we sell. But no. The many, extremely sad garden paths of despair is what we should be must focused on. So what does that mean?

A holistic approach to failure modes. Users trust the system, and in the case of a hospital, they trust process above all. When every single action results in a warning, then warnings mean absolutely nothing. We know that perfectly well, so why do we keep doing this?

Then you get exactly what happened here: users ignore virtually-always meaningless warnings but trust the process and the inputs made by staff before them.

So how do we fix this? Better input modes, visual confirmation of the correctness of those inputs, judicious use of warnings with different severity levels, and for the love of the Matrix, fewer customizability options to prevent an untestable cambrian explosion of use cases.

[story]: https://medium.com/backchannel/how-technology-led-a-hospital-to-give-a-patient-38-times-his-dosage-ded7b3688558
