---
title: You want your SMEs to read this
date: 2024-10-21
authors:
  - lpetralli
comments: true
---

# A Truth Your Domain Experts Can’t Ignore

Working with generative AI isn't not just about building cool agents that can cover multiple use cases — it's also about making sure those agent deliver the right answers, **every time**. 

One of the key ingredients that gets overlooked is the *Ground Truth Dataset*: the set of human inputs used as reference against which we evaluate a language model to ensure that it works as expected. While there are reference-free metrics in LLM evaluation that do not require a Ground Truth, **a good *evaluation suite* should include reference-based metrics** to ensure comprehensive assessment.

<!-- more -->

## Ground Truth is not optional

Let's talk about something that’s often treated as optional, when it really shouldn’t be: *having a solid Ground Truth is as necessary as defining a project roadmap*. 

You wouldn’t kick off a project without knowing what success looks like, so *why would you launch an LLM-based project without knowing what correct answers look like?* 

The Ground Truth is what defines how you want your agent to respond and the specific standards you are aiming for. Without it, how can you measure your progress or even know if your solution is hitting the mark?

#### No vibe check allowed

We've seen many projects where people skip this step because it’s challenging or costly—requiring involvement from subject matter experts (SMEs) and the broader team. But trust me, **building a well-defined Ground Truth dataset is a must from the start**. Yes, it’s resource-intensive, but it’s also foundational. 

Without it, you’re flying blind, relying only on a *vibe check* or intuition about whether the LLM is doing well. And that’s just not enough...

## Ground Truth must be true

So, what does a good Ground Truth look like? We don't always know the perfect answer, but we must know what information is needed to generate the correct answer. 

Also, *format* and *style* should be considered when defining the Ground Truth. 

Let's look at the examples below:

Input prompt: *What are the main features of the premium membership plan?*

- **Poor Ground Truth**: “The premium plan offers lots of great features and benefits that make it a good choice for many users.”
- **Good Ground Truth**: “The premium membership plan provides exclusive benefits, including unlimited access to resources, personalized support, priority booking, and additional discounts on services — designed to enhance your experience at every step!. [Source: Membership Policy Document, Section 3.1]”

## Ground Truth is a joint effort

When embarking on an LLM-based project, you have to ask yourself two main questions: What do we want the system to do? And what should the system never do? Right from this initial discussion, we need to think about the Ground Truth — how will we get it, who will create it, and who will validate it. 

**Here's the deal: Your SMEs don't need a PhD in AI** — but they do need to grasp what makes Ground Truth tick. Think of it like teaching someone to drive: they don't need to know how the engine works, but they need to understand the rules of the road. Let's show them exactly how their expertise fits into the bigger picture. 

You should aim to have something like this:

![SMEs Scale](../../assets/SMEs-scale.png)

#### It is not a solo SME job

Ground Truth creation is a collaborative process that should involve both SMEs and AI engineers, working together in a well-oiled loop. The SME's role is to provide all the necessary information for a good response, while it's the responsibility of the science team to achieve this in practice. Any updates to the documentation or changes to Ground Truth should be easy to implement across the system. 


## Using Synthetic Data

A tip I’d like to share: if you don’t yet have validated Ground Truth, definitely consider starting with synthetic data. 

Generate questions from your existing documentation and have an LLM provide theoretical answers—these serve as initial synthetic Ground Truths, and is better than having nothing as a reference. 

#### Just a starting point

This approach can be useful for setting up a baseline evaluation suite without relying entirely on SMEs from day one. But remember, **synthetic data is a starting point, not an end goal**. It’s tempting to stick with it, but you need to move on to real Ground Truths validated by SMEs. 

#### Not a long-term solution

In the long run, incorporating production traces into your dataset is also a must. 

*Synthetic datasets are a great way to get off the ground, but they’re not where you should stop.*

## Avoid These Common Pitfalls

- *Overburdening SMEs*: Don't expect SMEs to become LLM experts. Instead, create structured processes where they focus solely on domain expertise while AI engineers handle technical aspects.

- *Dependence on Synthetic Data*: While synthetic data helps get started, it can introduce biases and make transitioning to human validation difficult. Use it strategically and plan for a gradual shift to human-validated data.

- *Stuck in the Iteration Loop?*: Don't let ground truth updates become a bottleneck! Keep your momentum by using version control, breaking changes into bite-sized pieces, focusing on high-impact updates first.




