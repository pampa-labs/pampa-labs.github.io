---
title: You want your SMEs to read this
date: 2024-10-21
authors:
  - lpetralli
comments: true
---

# You want your SMEs to read this

Working with generative AI isn't not just about building cool agents that can cover multiple use cases — it's also about making sure those agent deliver the right answers, **every time**. 

One of the key ingredients that gets overlooked is the *Ground Truth Dataset*: the set of human inputs against which we evaluate a language model to ensure that it works as expected. While there are reference-free metrics in LLM evaluation that do not require a Ground Truth, a good *evaluation suite* should include reference-based metrics to ensure comprehensive assessment.

<!-- more -->

## Ground Truth is not optional

I want to talk about something that’s often treated as optional, when it really shouldn’t be. Having a solid Ground Truth is as necessary as defining a project roadmap. You wouldn’t kick off a project without knowing what success looks like, so *why would you launch an LLM-based project without knowing what correct answers are?* The Ground Truth is what defines how you want your agent to respond and the specific standards you are aiming for. Without it, how can you measure your progress or even know if your solution is hitting the mark?

I’ve seen many projects where people skip this step because it’s challenging or costly—requiring involvement from subject matter experts (SMEs) and the broader team. But trust me, **buildinga well-defined Ground Truth dataset is a must from the start**. Yes, it’s resource-intensive, but it’s also foundational. Without it, you’re flying blind, relying only on a *vibe check* or intuition about whether the LLM is doing well. And that’s just not enough...

So, what does a good Ground Truth look like? We don't always know the perfect answer, but we must know what information is needed to generate the correct answer. Also, format and style should be considered when defining the Ground Truth. Let's look at the examples below:

Input prompt: 

- **Poor Ground Truth**: 
- **Good Ground Truth**: 

## Collaboration is the Key to Ground Truth Success

When embarking on an LLM-based project, you have to ask yourself two main questions: What do we want the system to do? And what should the system never do? Right from this initial discussion, we need to think about the Ground Truth — how will we get it, who will create it, and who will validate it. **SMEs don’t need to be experts in LLMs**, but they do need to understand the basics of what makes good Ground Truth. This means we have to educate them — they need to know how their knowledge will be used for evaluation.

![SMEs Scale](../../assets/SMEs-scale.png)

**Ground Truth creation must not be a solo SME job**. It's a collaborative process that should involve both SMEs and AI engineers, working together in a well-oiled loop. The SME's role is to provide all the necessary information for a good response, while it's the responsibility of the science team to achieve this in practice. Any updates to the documentation or changes to Ground Truth should be easy to implement across the system. Creating a streamlined pipeline here makes it easier to keep everything accurate and current, allowing the SMEs to focus on providing expert knowledge while the AI team focuses on implementation and optimization.

## Using Synthetic Data as a Starting Point

A tip I’d like to share: if you don’t yet have validated Ground Truth, definitely consider starting with synthetic data. Generate questions from your existing documentation and have an LLM provide theoretical answers—these serve as initial synthetic Ground Truths, and is better than having nothing as a reference. This approach can be useful for setting up a baseline evaluation suite without relying entirely on SMEs from day one. But remember, **synthetic data is a starting point, not an end goal**. It’s tempting to stick with it, but you need to move on to real Ground Truths validated by SMEs. In the long run, incorporating production traces into your dataset is also a must. Synthetic datasets are a great way to get off the ground, but they’re not where you should stop.

## Avoid These Common Pitfalls

- *Overburdening SMEs*: Is it realistic to expect SMEs to become experts in LLMs and the creation of golden datasets? This can be seen as an additional burden that distracts experts from their primary responsibilities.

- *Dependence on Synthetic Data*: Starting with synthetic data can create a shaky foundation, potentially perpetuating biases or errors in the model if not handled carefully. The transition to human-validated data might be more challenging than anticipated.

- *Difficulty in Iteration*: Constant iteration of the golden dataset can be unrealistic when trying to maintain an agile pipeline. Changes in documentation or expectations could require deeper and slower revisions than expected.

- *Excessive Emphasis on Human Validation*: Relying heavily on human validation can be costly and slow, especially at scale. 


