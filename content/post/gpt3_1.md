---
title: "Kickstart text classification with GPT-3 zero-shot learning, part 1"
date: 2022-08-27
tags: ["ml", "gpt3", "nlp"]
draft: false
---

## What is GPT-3?

GPT stands for “Generative Pre-trained Transformer”, and "3" is simply the version. There are many resources describing what it is, what it does, etc. <!--TODO List resources--> 

However, we can treat GPT-3 as a black box. There are a few important things to keep in mind:

1. GPT-3 is a text completion engine. Whatever you give it, it will try to add text to it until it reaches either the token number limit you set or one of the stop-words you’ve specified.
2. GPT-3 was trained on a huge amount of text, mostly scraped from the web[^1].
[^1]: See [Common Crawl](https://commoncrawl.org/)

### Accessing GPT-3
You can access GPT-3 on [OpenAI's website](https://beta.openai.com/). It has a nice interface and an API, both of which we will be using later.

GPT-3 is not free, but OpenAI gives you enough credit on sign-up to explore the potential applications, and it is enough for a few small-scale projects. In general, these kinds of engines will become cheaper as time goes on.

#### A small note on different engines

We will be using Davinci engine.

## How I think about text completion

So what can we do with GPT-3? Well, we can complete text. But kinds of problems does it help us to solve?

Turns out, a lot of problems can be reframed into text completion. This is where creativity and imagination will be of use. OpenAI gives many [examples](https://beta.openai.com/examples), such as Keyword extraction, Text summarization, and many more.

## Fine-tuning, few-shot and zero-shot classification

## Limitations of text completion

## Naive approach