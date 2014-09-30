---
layout: post
title: "Why Doesn't Linux Allow Sleeping in External Interrupt Context"
modified:
categories:
comments: true
excerpt: Thoughts on Linux threads implementation
tags:
  - Operating System
  - Linux
image:
  feature:
date: 2014-09-24T13:45:34-07:00
---

An apparent answer for why Linux designers disallow external interrupt handler to sleep is that, of course, these handlers are meant to run and terminate in a really short time period. But there's more than one reason.

When you google this question, most webpages you see will say that it is because there is NO process context. What does it mean?

Let's take a look at what happens when an external interrupt happens. As you may remember, usually in Linux, every thread has two stacks, a user stack and a kernel stack. When an external interrupt pops up during a thread's execution, kernel puts it on the interrupted thread's **kernel stack**.

This is important. The interrupt is indeed living on a thread's stack. However, it isn't associated with whatever the thread does at all. Sure, we can have an OS which allows external interrupt handler to go to sleep. But why does it take away the innocent thread? In the end, it makes no sense to sleep/wake a thread only to satisfy the needs of a handler that only happens to live on the thread's stack.

On the other hand, internal interrupts, such as exceptions, could be associated with the thread. For instance, an user accidentally performs a zero division! The alarm is triggered and internal interrupt gets pushed onto the kernel thread. Now is this interrupt closely related to the thread? Surely it is. With this subtle difference between internal and external interrupt, Linux, in my opinion, makes the right choice to prevent sleeping in external interrupt context.