---
title: "Kickstart text classification with GPT-3, part 1"
date: 2022-10-01
tags: ["ml", "nlp", "gpt3"]
selected: true
draft: false
---


## Motivation
In the life of any data scientist there comes a point when they need to train a baseline classification model for some task, but don't have labelled data to start. In this series of posts, I propose an approach to get labelled data for text classification using large pretrained neural networks, such as GPT-3. This approach will help you kickstart your classification problem. 

## TL;DR
Gives a brief introduction to GPT-3; discusses a way to think about text completion; explains what is zero-shot, few-show classification and fine-tuning; introduces the example dataset that will be used in this series of posts; gives several examples of a naive approach to completion.

## What is GPT-3?

GPT stands for “Generative Pre-trained Transformer”, and "3" is simply the version. There are [many](https://medium.com/sciforce/what-is-gpt-3-how-does-it-work-and-what-does-it-actually-do-9f721d69e5c1) [good](https://en.wikipedia.org/wiki/GPT-3) [resources](https://towardsdatascience.com/understanding-gpt-3-in-5-minutes-7fe35c3a1e52) describing what it is, and what it does. I won't go into these details here. <!--TODO Find and add links to resources--> 

In this series of posts, we can treat GPT-3 as a black box. There are a few important things to keep in mind:

1. GPT-3 does text completion. Whatever you give it, it will try to add text to it until it reaches either the token number limit you set or generates one of the stop-words/symbols you’ve specified (e.g. new line symbol).
2. GPT-3 was trained on a huge amount of text mostly scraped from the web[^1].
[^1]: See [Common Crawl](https://commoncrawl.org/)

### Accessing GPT-3
You can access GPT-3 on [OpenAI's website](https://beta.openai.com/). It has a nice interface and an API, both of which we will be using later.

GPT-3 is not free, but OpenAI gives you credits on sign-up to explore the potential applications, and it is enough for a few small-scale projects. In general, these kinds of engines will become cheaper as time goes on.

#### A note on different engines
There are [several versions of the GPT-3 models](https://beta.openai.com/docs/models/gpt-3) (sometimes referred to as "engines"). The differences are based on the size of the network and training data. Smaller networks work faster and consume fewer resources and are therefore cheaper. But the completions are dumber. We will be using the most capable engine Davinci because we need to squeeze all the smarts we can from the network.


## How I think about text completion

So what can we do with GPT-3? Well, we give GPT-3 some initial text, called ***prompt***, and GPT-3 completes it. What kinds of problems does it help us to solve?

Turns out, a lot of problems can be reframed as text completion problems. This is where creativity and imagination will be of use. OpenAI gives many [examples](https://beta.openai.com/examples), such as Keyword extraction, Text summarization, and more. The process of reframing the problem you want to solve into a text completion problem is called ***prompt engineering***[^pe].

[^pe]: Incidentally, prompt engineering is relevant not only for text completion such as GPT-3 but also for image generation networks, such as DALLE and Midjourney. It has gotten to the point and many people consider prompt engineering as its own way of programming, and predict that shortly we will have a whole profession of "prompt engineers".

But how to come up with good prompts? By now, prompt engineering has hundreds of posts written about it, and soon there will be dedicated books. However, as we just want to get started, I offer you a way to think about prompts that has helped me on many occasions. I call it **"it's gotta be somewhere on the internet"**. 

Whenever you create a prompt, think about whether the result (your prompt + completion) could be the content of a page somewhere on the internet. This idea comes from the fact that the training dataset for GPT-3 included a lot of web page content.

The Internet is a weird place, and there is life beyond the first couple of pages of Google Search results. The Internet is full of [old university pages](https://homepages.inf.ed.ac.uk/rbf/HIPR2/gsmooth.htm), [dumps of mailing lists](https://stat.ethz.ch/pipermail/r-sig-finance/2012q1/009645.html), [articles containing tables with data](https://ijpds.org/article/view/1680), and other interesting things. In the next post, I will show how I use table format to create an efficient ***zero-shot*** classifier. Wait, what's zero-shot?

## Zero-shot, few-shot and fine-tuning

There are a few terms you need to know if you're going to use GPT-3 for classification.

***Zero-shot*** simply means that you use GPT-3 for completion without giving specific examples of what you want to do. In essence, you rely on what the network has already learned during its training. An example of using zero-shot classification to infer the genre of a Steam game from description[^ds] ({{<gpt-comp>}}GPT completion is highlighted{{</gpt-comp>}}):

[^ds]: In this series of posts I will be using the dataset of [Steam games descriptions and tags](https://www.kaggle.com/datasets/trolukovich/steam-games-complete-dataset?resource=download). A subset of the most popular tags will be used as genres. Of course, the whole point of this approach is to label *unlabelled* data. But for demonstrating the approach it is useful to have labelled data.

{{<gpt>}}
<p>Game description:</p>

<p>Carefully guide your nation from the era of absolute monarchies in the early 19th century, through expansion and colonization, to finally become a truly great power by the dawn of the 20th century. Victoria II is a grand strategy game played during the colonial era of the 19th century, where the player takes control of a country, guiding...</p>

<p>Is this a strategy game? Yes or No: {{<gpt-comp>}}Yes{{</gpt-comp>}}</p>
{{</gpt>}}


***Few-shot*** is when *in the prompt* you provide one or more examples of what you want GPT-3 to do. This can be quite powerful, especially when the completion you want is non-trivial, and after a few tries, you see that GPT-3 doesn't make it correctly after a zero-shot try. This can also improve the stability of the response: by providing an example, you all but ensure that the rest of the response will be in the same format. But there are better ways of ensuring stability which I will discuss in future posts. Few-shot comes with additional challenges, such as the dependency of the responses on the example(s) you've provided. Example of 1-shot classification:

{{<gpt>}}
<p>Game description:

<p>Carefully guide your nation from the era of absolute monarchies in the early 19th century, through expansion and colonization, to finally become a truly great power by the dawn of the 20th century. Victoria II is a grand strategy game played during the colonial era of the 19th century, where the player takes control of a country, guiding...

<p>Is this a strategy game? Yes or No: Yes

<p>Game description:

<p>Tactical Squad-Based Combat comes to the Fallout® Universe! You are the wretched refuse. You may be born from dirt, but we will forge you into steel. You will learn to bend; if not you, will you break. In these dark times, the Brotherhood - your Brotherhood - is all that stands between the rekindled flame of civilization and the howling,...

<p>Is this a strategy game? Yes or No: {{<gpt-comp>}} Yes{{</gpt-comp>}}
{{</gpt>}}

***Fine-tuning*** refers to modifying the underlying network using a labelled dataset *before prompting for completion*. It allows turning a general-purpose network, such as GPT-3, into a very specific model. It requires having some labelled data upfront, and it is more expensive if you want to do it with GPT-3. You can read more about GPT-3 fine-tuning [here](https://beta.openai.com/docs/guides/fine-tuning).

<!-- ## Limitations of text completion

What I was showing before was end-of-text completion. This is when the completion comes after the prompt. However, it doesn't have to be like that. Theoretically, we could ask a language model like GPT-3 to fill a specific gap in the text. It seems that this is not so different from end-of-text completion.

However, it does add complexity and needs to be used with care. For example, should the filling depend on the text before it, or it can also look ahead? What if you have multiple gaps, do they influence each other? If so, what should be the order of the filling?

Anyway, for now, the "Insert mode" in GPT-3 is in Beta mode, and I will not be using it here. If you'd like to read more about it, let me know in the comments, I can write a post of that in this future. -->

## A naive approach to zero-shot classification

Consider the examples from the [Zero-shot section above](#zero-shot-few-shot-and-fine-tuning). We can see how GPT-3 can helps us do classification. However, it seems cumbersome to make requests for each of the samples, and each of the categories. Are there better ways?

### Multiple examples

We can pack multiple examples in the same request, like so:

{{<gpt>}}
<p># Game Descriptions

<p>1. Carefully guide your nation from the era of absolute monarchies in the early 19th century, through expansion and colonization, to finally become a truly great power by the dawn of the 20th century. Victoria II is a grand strategy game played during the colonial era of the 19th century, where the player takes control of a country, guiding…

<p>2. Build the adventure from Privet Drive to the Triwizard Tournament and experience the magic of the first four Harry Potter stories – LEGO style! Explore Hogwarts™ School of Witchcraft and Wizardry, learn spells, brew potions and relive the adventures like never before with tongue-in-cheek humor and creative customization that is unique to...

<p>3. Sudeki, a world torn apart into light, shadow and dark. Rent asunder by deceit and betrayal, the land cries out for a peace that only four united heroes can bring. Beyond the protective walls of Illumina Castle lie miles of pastoral, rolling countryside, though once a serene landscape, the Aklorian forces have transformed the countryside...

<p>4. Mini Ninjas is a game that combines furious action with stealth and exploration for an experience that appeals to a wide audience across age groups and preferences. It’s an action-adventure with a strong focus on allowing the player freedom to explore the world and has the depth to allow for very varied gameplay and approaches to getting...

<p>5. Life under the rule of the winged Skyborn race isn't so bad for Claret Spencer, the star mechanic of an independent repair shop. She can patch up just about anything...but when a certain cravat-wearing customer turns her life upside-down, she finds herself pulled into an epic, city-wide conflict that's going to take a lot more than elbow...
<p># Is this a strategy game? (Yes or No)
<p>1. {{<gpt-comp>}} Yes
<br>2. No
<br>3. No
<br>4. No
<br>5. No{{</gpt-comp>}}
{{</gpt>}}

As you can see, we can get predictions for multiple games in the same request.

However, what if we want to get predictions for multiple games and multiple labels (genres) at the same time? In the next post, I will show how to use the markdown tables structure to achieve this goal.
