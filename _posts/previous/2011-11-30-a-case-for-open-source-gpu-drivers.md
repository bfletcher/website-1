---
author: jesse.barker
date: 2011-11-30 19:39:50+00:00
layout: post
link: /blog/community-blog/a-case-for-open-source-gpu-drivers/
slug: a-case-for-open-source-gpu-drivers
title: A Case for Open-Source GPU Drivers
wordpress_id: 960
categories:
- blog
tags:
- Community
- Hardware
---

For some time after I signed on as the graphics working group tech lead at Linaro, upon meeting someone new, the first thing after "nice to meet you" was invariably, "When do we get free drivers?".  Ideologically, this seems like an automatic "great idea".  After all, Linux is all about free (definitely in terms of software, mostly in terms of beer :-), open-source software and the ability of the user to decide what (or what not) to run on his or her hardware.  Drivers, however, are a tricky subject.  As much as I would like to believe that software is software, my experience with a number of driver teams at various GPU and computer companies tells me that things are not always so simple.

What's the big deal?  There are a couple of things really.  First, GPUs tend to be really different beasts from other devices.  There are huge variations from generation to generation within a single vendor, and even more when considering the disparate architectures from different vendors.  They have all the complexity and more of CPUs, but with none of the convenient compatibilities.  If you are programming for an x86 CPU, you target the same set of instructions regardless of the supplying vendor or underlying microarchitecture.  With GPUs, it is precisely the microarchitecture that gets exposed.  In addition, as part of the system software rather than application software, GPU drivers - as well as those for other devices - are expected to be "trusted".  They should follow the principle of "first, do no harm".  Graphics is possibly the subsystem in which this principle has been violated the most.  With graphics, the principle is more one of "first, do not be slow"; the pixel might be "blue enough" if its value can be computed in three-quarters of the time of the correct value.  The issue is where the speed-ups come from.  One possibility is an algorithmic shortcut, but drivers also play fast and loose with buffer mappings, granting direct access to potentially protected device memory in order to ensure that the fastest path to pixels is taken.  There is potential here for real evil, particularly if that device memory is adjacent to, say, a piece of your file system.  The advent of hardware memory management units has alleviated much of this sort of concern, but it is certainly possible for a poorly written driver stack to trash a perfectly good system.

So, given a community with the right sort of experience and low-level knowledge, it is certainly conceivable that you could end up with a serviceable driver stack.  For one "family" of one GPU (GPUs tend to go through iterative spins of the same basic architecture between radical revamps of the whole thing).  What about all the other ones?  A clever community might even design a driver stack architecture that allows them to change only certain pieces to support either a new underlying GPU or a different overlying API.  In fact, as we know, this is precisely what has happened (i.e., [Gallium 3D](http://www.freedesktop.org/wiki/Software/gallium)).  It seems like we're back to the automatic "great idea", right?  Maybe.  The problem is that there are two sides of this story and the two sides don't typically see eye-to-eye.  The GPU vendors have experience and low-level knowledge but are reluctant just to let their cleverness go off into the universe to live a life of its own.  The community has the best intentions and want to grant access to all of this great stuff, but they don't always have the right knowledge and experience (of course, this situation improves progressively over time).  Not long ago, I was asked about this situation by a senior technical manager at a GPU vendor.  When I thought about the questions posed, I realized that what is happening on both sides is part misunderstanding and part disparity of priorities.  I'll include here the questions and my original answers to them to illustrate:

1. Why does the Linux community want an open-source GPU driver?

There are really two pragmatic answers to this question: 1) security 2) debugging.  Given that a driver stack is made up of a kernel component, which, technically must be licensed under the GPLv2 on Linux, and a user-space component (e.g., the EGL and OpenGL ES libraries that an application links with), these two issues have their weakest point at this boundary if the user-space portion is closed.  It is possible, though, in the modern era, getting somewhat less likely all the time, for a GPU driver to play games with memory mapping that would allow the user-space library to overwrite, say, a filesystem, based upon a bad memory access (e.g., some off-by-one error causing a write past the end of a buffer boundary and hitting another device's memory).  There are other scenarios around this, but this is the gist.  Then there is the issue of what to do when a bug is encountered.  If the same closed-source library does something bad that crashes the system, or simply puts the wrong results on the screen, the maintainer and other developers of an open-source component like DRM (the kernel subsystem that is responsible for GPU resource management) have no recourse to figure out what went wrong, let alone address the bug.  Now they must rely on the mercy of the closed-source developers to address the problem.  Of course, there's really also a third non-technical reason: the basic ideology of free open-source software itself.

2. What exactly does that [producing an open-source driver] involve?

Again, it's a two-fold answer and it's relatively simple in concept, though a lot of work in practice, especially if you've developed your drivers in a proprietary environment.  One writes (or ports) a driver stack according to the architecture and coding standards of the relevant components.  In the case of the graphics stack, these are: DRM for the kernel bits, X.org for the X11 driver, and Mesa - preferably the Gallium3D model - for the EGL and OpenGL ES libraries.  Then you submit these pieces as patchsets to the relevant developer lists (each project has one), wait for the developers on the lists to pelt you with virtual rocks and garbage, revise your code based upon the comments, then resubmit as necessary until the maintainers will accept your patchsets.  Once that happens, it is part of the open-source "upstream" and anyone can work on it.  The other way is to release the hardware specifications and foster a community project to write drivers based upon the documentation.  This is actually the ideal as it empowers the community to code the drivers based upon the same documentation that vendor engineers would use; it has the added advantage of being cheaper and less work for vendor engineering teams.  Of course, from the vendor perspective, there are a lot of stakeholders to convince (including the IP lawyers) that doing so will not bring harm to the company (e.g., law suits by patent trolls).

3. Why doesn't graphics engineering want to do it?

Porting a proprietary driver costs a lot of time and resource, and it ties the vendor engineering team to a development effort over which they potentially have no control.  As I said, once your driver stack is open-sourced and part of an upstream repository, anyone can work on it, and the driver may be taken in a direction the vendor team does not like; worse yet from their perspective, their own patches may be refused.  There is a potential "gotcha" here as well.  For a product driven entity (the hardware vendor), the sense of priority for open-source driver development may not seem a great fit.  Of course, if someone from the vendor is positioned as the driver maintainer (Intel seems to benefit from this approach), there is a better chance of the stack going in a desired direction.  The reality is also that graphics IP vendors are concerned with supporting operating environments apart from Linux.  So, if they can't use the open-source driver to support, say, Windows, it looks a lot like double the effort to support multiple platforms.  This, as well as the original engineering investment in the driver stack, is largely why NVIDIA, AMD, and IMG, for example, still do proprietary drivers.

4. Who else in GPU land (e.g., IMG, NVIDIA, etc.) does this?

These examples don't manage their driver stacks this way, but others do to varying degrees.  AMD release specs and foster the open-source driver efforts.  Of course, they also release a proprietary driver at the same time, which is their "official" support.  Intel is really the king of this approach, as they release specifications and actually use the open-source drivers as the officially supported Linux software for their hardware.  Many maintainers of graphics-related open-source components are Intel employees, and they are _very_ active in the community.  Their model would be good if you choose to go that route as they have an Intel driver maintainer that feeds into each of the core upstream components (this helps them govern the direction that their driver development takes - see my comments above under question 3).  Of course, the glaring exception here is that neither of them do this for their latest generation.  Also of interest is the Nouveau project which reverse engineers NVIDIA GPUs with absolutely no public contribution from them (just an abundance of available hardware).  This is something you could expect to see when enough low-cost developer boards with a given GPU become available.

Since this e-mail exchange, I've had a little additional time to ponder some of these questions a little further.  Much of what I wrote was considered from the perspective of the common Linux graphics stack; i.e., the answers are likely to be somewhat different for Android.  For question 1, there is another answer, or, rather, there is a wider answer than "debugability".  In particular, I think the overarching theme is maintainability.  This covers not just the two parts of my original answer, but also the ability of a community to fix, improve, and extend the life of device support well beyond the cycle intended by the vendor.  I also realize that the issue of patents and their impact on the open driver situation, not to mention open software in general, has had little more than a glossing over here.  I wish I could do the topic justice.  Some of my venerable colleagues have posted excellent positions on the topic.  In particular, Kiko's [LCA2011 presentation](https://wiki-archive.linaro.org/ChristianReis), and Dave Airlie's well written [blog post](http://airlied.livejournal.com/73337.html) take valiant stabs at addressing this.  My personal feeling is that in the discrete PC space, the IP vendors live in a state of "mutually assured distruction" that keeps them from poking too deeply at GPU patent litigation.  The embedded/mobile space has not yet reached this stasis, and so the fear of litigation outweighs the potential benefits of opening things up as has been done for discrete GPUs.

So what is the lesson here?  On the community side, we may have the greater challenge.  We can argue until we're blue in the face about all of the great benefits of community development, but we can't indemnify graphics IP vendors against lawsuits when the details of their designs are revealed in open-source drivers.  This obviously isn't guaranteed to happen, but it only has to happen once to foul up the works.  On the vendor side, with a little help, trust and understanding you get a Linux driver stack, lots and lots of testing on a variety of platforms and an active community supporting your platforms.  This isn't a free lunch, though, and it is important to seed the development community with engineering resources; this improves the community as well as the software it produces.