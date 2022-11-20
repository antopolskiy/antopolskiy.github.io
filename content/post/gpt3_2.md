---
title: "Kickstart text classification with GPT-3, part 2"
date: 2022-10-22
tags: ["ml", "nlp", "gpt3"]
selected: true
draft: false
---

## Where we stopped

In the [previous part]({{< ref "/post/gpt3_1" >}}) we finished with the desire to get predictions for multiple samples and multiple categories. We will do it here using a Markdown table format.

## What is Markdown?

> Markdown is a lightweight markup language that you can use to add formatting elements to plaintext text documents. Created by [John Gruber](https://daringfireball.net/projects/markdown/) in 2004, Markdown is now one of the world’s most popular markup languages.

Basically, Markdown is a way to specify formatting in a plain text document, using special symbols. E.g. to make bold text, you'd write two asterisks before and after, like so: `**bold**`. The platforms which support markdown will automatically render this as **bold** text.

## How Markdown tables are written in plain text

There is a way to create tables in Markdown, which, in the most basic version, looks like this:
```markdown
| column 1 | column 2 | column 3 |
| --- | --- | --- |
| 1 | 2 | 3 |
| 4 | 2 | 3 |
```

And it will be rendered like so:

{{<table "table table-striped table-bordered">}}
| column 1 | column 2 | column 3 |
| --- | --- | --- |
| 1 | 2 | 3 |
| 4 | 2 | 3 |
{{</table>}}

Why is this useful for us? Because GPT-3 has seen a lot of Markdown. And once we hint at the fact that what it is completing is a Markdown document, it will stick to this format pretty well. This will give us a stable and structured output, which we can easily parse.

So the main idea of the approach is that we will prompt GPT-3 in such a way that it will create a table of samples and labels. How exactly?

## Organizing a response as a Markdown table

Here is the basic breakdown: 

1. Provide the text to be classified as a list of samples. 
2. Each sample needs to have some kind of identifier, which will be included in the table (in our example of Steam games classification, it can be the name of the game). 
3. Create the first two rows of Markdown table: column titles and the separator. 
4. Let GPT-3 do the rest of the magic. 

Here are a couple of examples of prompts with GPT-3 completion:

{{<gpt>}}
# List of games
<br>
<br>Name: PlanetSide 2
<br>Description: PlanetSide 2 is an all-out planetary war, where thousands of players battle as one across enormous continents. Utilize infantry, ground and air vehicles, and teamwork to destroy your enemies in this revolutionary first-person shooter on a massive scale.
<br>
<br>Name: SMITE®
<br>Description: Join over 30 million SMITE players! Wield Thor’s hammer, transform your foes to stone as Medusa, or choose from the other 100+ mythological icons. You’ll always have more to discover with free regular updates. Enter the Battleground of the Gods!
<br>
<br>Name: Grand Theft Auto IV
<br>Description: What does the American dream mean today? For Niko Bellic fresh off the boat from Europe, it is the hope he can escape from his past. For his cousin, Roman, it is the vision that together they can find fortune in Liberty City, gateway to the land of opportunity.
<br>
<br>Name: Chivalry: Medieval Warfare
<br>Description: Besiege castles and raid villages in Chivalry: Medieval Warfare, a fast-paced medieval first person slasher with a focus on multiplayer battles
<br>
<br>Name: Darkest Dungeon®
<br>Description: Darkest Dungeon is a challenging gothic roguelike turn-based RPG about the psychological stresses of adventuring. Recruit, train, and lead a team of flawed heroes against unimaginable horrors, stress, famine, disease, and the ever-encroaching dark. Can you keep your heroes together when all hope is lost?
<br>
<br># Table of games and their genres
<br>
<br>| Name | Action | RPG | Strategy | Simulation | Puzzle | Casual | Adventure | Platformer |
<br>| --- | --- | --- | --- | --- | --- | --- | --- | --- |{{<gpt-comp>}}&nbsp;
<br>| PlanetSide 2 | Yes | No | Yes | No | No | No | No | No |
<br>| SMITE® | Yes | No | Yes | No | No | No | No | No |
<br>| Grand Theft Auto IV | Yes | Yes | Yes | Yes | No | No | Yes | No |
<br>| Chivalry: Medieval Warfare | Yes | No | Yes | No | No | No | No | No |
<br>| Darkest Dungeon® | Yes | Yes | Yes | No | No | No | Yes | No |{{</gpt-comp>}}
{{</gpt>}}

{{<gpt>}}
# List of games
<br>
<br>Name: DOOM
<br>Description: Now includes all three premium DLC packs (Unto the Evil, Hell Followed, and Bloodfall), maps, modes, and weapons, as well as all feature updates including Arcade Mode, Photo Mode, and the latest Update 6.66, which brings further multiplayer improvements as well as revamps multiplayer progression.
<br>
<br>Name: This War of Mine
<br>Description: In This War Of Mine you do not play as an elite soldier, rather a group of civilians trying to survive in a besieged city; struggling with lack of food, medicine and constant danger from snipers and hostile scavengers. The game provides an experience of war seen from an entirely new angle.
<br>
<br>Name: Saints Row IV
<br>Description: The US President must save the Earth from alien overlord Zinyak using an arsenal of superpowers and strange weapons in the wildest open world game ever.
<br>
<br>Name: Divinity: Original Sin 2 - Definitive Edition
<br>Description: The eagerly anticipated sequel to the award-winning RPG. Gather your party. Master deep, tactical combat. Join up to 3 other players - but know that only one of you will have the chance to become a God.
<br>
<br>Name: Portal
<br>Description: Portal™ is a new single player game from Valve. Set in the mysterious Aperture Science Laboratories, Portal has been called one of the most innovative new games on the horizon and will offer gamers hours of unique gameplay.
<br>
<br># Table of games and their genres
<br>
<br>| Name | Action | RPG | Strategy | Simulation | Puzzle | Casual | Adventure | Platformer |
<br>| --- | --- | --- | --- | --- | --- | --- | --- | --- |{{<gpt-comp>}}&nbsp;
<br>| DOOM | X |  |  |  |  |  |  |  |
<br>| This War of Mine |  | X |  |  |  |  |  |  |
<br>| Saints Row IV | X |  |  |  |  |  |  |  |
<br>| Divinity: Original Sin 2 - Definitive Edition |  | X | X |  |  |  |  |  |
<br>| Portal |  |  |  | X | X |  |  |  |{{</gpt-comp>}}
{{</gpt>}}


# Limiting values in the table

As you can notice above, the tables in the two examples are not exactly in the same format. In my experiments, I have seen `Yes` / `No`, `T` / `F`, `true` / `false`, `+` / `-`, `TRUE` / `FALSE`, `X` / `O`, `X` / nothing. Why is this important for us?

It is time to think about what we are going to do with the responses. Let's remind ourselves that our goal is to kickstart classification. That is, we want to label some new data so that later we can train our own supervised classified. Let's start with the intention to label 200 samples. 

But with a single request to GPT-3 we can only label 5-10 samples (depending on the size of the prompt, we will discuss this below in [Number of samples section]({{<ref "#number-of-samples">}}) section). Hence we need to automate the process of making requests and parsing the responses. And the first step is to make sure that the responses are uniform: we want the same type of values in the table, instead of many different ones.

I've found that the simplest way to do that is to include the values in the prompt ({{<red>}}note the change{{</red>}}):

{{<gpt>}}
# List of games
<br>
<br>Name: DOOM
<br>Description: Now includes all three premium DLC packs (Unto the Evil, Hell Followed, and Bloodfall), maps, modes, and weapons, as well as all feature updates including Arcade Mode, Photo Mode, and the latest Update 6.66, which brings further multiplayer improvements as well as revamps multiplayer progression.
<br>
<br>Name: This War of Mine
<br>Description: In This War Of Mine you do not play as an elite soldier, rather a group of civilians trying to survive in a besieged city; struggling with lack of food, medicine and constant danger from snipers and hostile scavengers. The game provides an experience of war seen from an entirely new angle.
<br>
<br>Name: Saints Row IV
<br>Description: The US President must save the Earth from alien overlord Zinyak using an arsenal of superpowers and strange weapons in the wildest open world game ever.
<br>
<br>Name: Divinity: Original Sin 2 - Definitive Edition
<br>Description: The eagerly anticipated sequel to the award-winning RPG. Gather your party. Master deep, tactical combat. Join up to 3 other players - but know that only one of you will have the chance to become a God.
<br>
<br>Name: Portal
<br>Description: Portal™ is a new single player game from Valve. Set in the mysterious Aperture Science Laboratories, Portal has been called one of the most innovative new games on the horizon and will offer gamers hours of unique gameplay.
<br>
<br># Table of games and their genres {{<red>}}(TRUE or FALSE){{</red>}}
<br>
<br>| Name | Action | RPG | Strategy | Simulation | Puzzle | Casual | Adventure | Platformer |
<br>| --- | --- | --- | --- | --- | --- | --- | --- | --- |{{<gpt-comp>}}&nbsp;
<br>| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE |
<br>| Saints Row IV | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |{{</gpt-comp>}}
{{</gpt>}}

In my experiments, this all but ensures that the values in the table will be TRUE or FALSE[^tf]. We will also add some checks in our automation code later to make sure that GPT-3 doesn't generate other values[^ofb]. 

[^tf]: I use TRUE and FALSE because it is usually how boolean types are written in SQL databases. It is as if I try to convince GPT-3 that the table is a dump of a SQL table.
[^ofb]: These types of errors are usually called "out-of-bound response". This is an umbrella term, which means that the classifier generated something unforeseen as a label.

I will hint that there is a better way to ensure the stability of responses, using `logit_bias`. I may address this at the end of the series, but it is not essential -- it is usually enough to modify the prompt as shown above.

# Experimenting with the prompt

At this point, we're close to fixing our prompt. Of course, the samples will change on every request, but the surrounding will stay the same. This is a good spot to experiment with the prompt a little more. 

In particular, I've found that thinking about the labels (names of the columns in the table) is very useful. Since we've not provided any description of the labels, GPT-3 can only go with the names of the labels to infer their meaning. For example, we may have the label "Action" -- but the word itself has multiple meanings. GPT-3 will consider the context (our prompt mentions games and genres), but we may want to try making it easier for GPT-3 to understand what we want. For example, instead of "Action", we can make the label "Genre Action". Or even make it more technical, as if it is a dump of the database, e.g. "genre_action".

{{< callout emoji="⚠️" 
 text="Before you start experimenting, check the settings of the completion on the right pane in the GPT-3 Playground UI. In particular, set `Temperature` to 0, `Maximum length` to something around 200, `Top P` to 1, `Frequency Penalty` to 0, `Presence Penalty` to 0, `Best of` to 1. I will discuss the settings in the next post.">}}


So what I usually do is pick some samples which I think GPT-3 should be able to classify and some samples which seem difficult for me and try to change the prompt and look at the responses for correctness. In parallel, I am trying to gauge the stability of the responses: how much do they change when I make modifications to the prompt?

I usually do 3-7 experiments like these. There is space for automation here, but I find this part fun and creative, and eventually not taking too much time.

Let's try this in practice. First, I change the games a little, picking them such that they span different genres, and try our initial prompt:


{{<gpt>}}
# List of games
<br>
<br>Name: DOOM
<br>Description: Now includes all three premium DLC packs (Unto the Evil, Hell Followed, and Bloodfall), maps, modes, and weapons, as well as all feature updates including Arcade Mode, Photo Mode, and the latest Update 6.66, which brings further multiplayer improvements as well as revamps multiplayer progression.
<br>
<br>Name: This War of Mine
<br>Description: In This War Of Mine you do not play as an elite soldier, rather a group of civilians trying to survive in a besieged city; struggling with lack of food, medicine and constant danger from snipers and hostile scavengers. The game provides an experience of war seen from an entirely new angle.
<br>
<br>Name: Divinity: Original Sin 2 - Definitive Edition
<br>Description: The eagerly anticipated sequel to the award-winning RPG. Gather your party. Master deep, tactical combat. Join up to 3 other players - but know that only one of you will have the chance to become a God.
<br>
<br>Name: Portal
<br>Description: Portal™ is a new single player game from Valve. Set in the mysterious Aperture Science Laboratories, Portal has been called one of the most innovative new games on the horizon and will offer gamers hours of unique gameplay.
<br>
<br>Name: AdVenture Capitalist
<br>Description: Welcome, eager young investor, to AdVenture Capitalist! Arguably the world's greatest Capitalism simulator!
<br>
<br># Table of games and their genres (TRUE or FALSE)
<br>
<br>| Name | Action | RPG | Strategy | Simulation | Puzzle | Casual | Adventure | Platformer |
<br>| --- | --- | --- | --- | --- | --- | --- | --- | --- |{{<gpt-comp>}}&nbsp;
<br>| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE |
<br>| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |
<br>| AdVenture Capitalist | FALSE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE |{{</gpt-comp>}}
{{</gpt>}}

The result is the following (rendering is for ease of reading):

{{<table "table table-striped table-bordered">}}
| Name | Action | RPG | Strategy | Simulation | Puzzle | Casual | Adventure | Platformer |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE |
| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |
| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |
| AdVenture Capitalist | FALSE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE |
{{</table>}}

At a first glance, this is not bad. I'd like it to classify Portal as Action and Platformer, but well, Puzzle is already pretty good. Still, I try to change the prompt a little. Remember that there are two main goals here: try to improve the responses and gauge the stability. I change the prompt so that it seems more like a database dump (lowercase labels with `snake_case`):


{{<gpt>}}
# List of games
<br>
<br>name: DOOM
<br>description: Now includes all three premium DLC packs (Unto the Evil, Hell Followed, and Bloodfall), maps, modes, and weapons, as well as all feature updates including Arcade Mode, Photo Mode, and the latest Update 6.66, which brings further multiplayer improvements as well as revamps multiplayer progression.
<br>
<br>name: This War of Mine
<br>description: In This War Of Mine you do not play as an elite soldier, rather a group of civilians trying to survive in a besieged city; struggling with lack of food, medicine and constant danger from snipers and hostile scavengers. The game provides an experience of war seen from an entirely new angle.
<br>
<br>name: Divinity: Original Sin 2 - Definitive Edition
<br>description: The eagerly anticipated sequel to the award-winning RPG. Gather your party. Master deep, tactical combat. Join up to 3 other players - but know that only one of you will have the chance to become a God.
<br>
<br>name: Portal
<br>description: Portal™ is a new single player game from Valve. Set in the mysterious Aperture Science Laboratories, Portal has been called one of the most innovative new games on the horizon and will offer gamers hours of unique gameplay.
<br>
<br>name: AdVenture Capitalist
<br>description: Welcome, eager young investor, to AdVenture Capitalist! Arguably the world's greatest Capitalism simulator!
<br>
<br># Table of games and their genres (TRUE or FALSE)
<br>
<br>| name | genre_action | genre_rpg | genre_strategy | genre_simulation | genre_puzzle | genre_casual | genre_adventure | genre_platformer |
<br>| --- | --- | --- | --- | --- | --- | --- | --- | --- |{{<gpt-comp>}}&nbsp;
<br>| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE |
<br>| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |
<br>| AdVenture Capitalist | FALSE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE |{{</gpt-comp>}}

{{</gpt>}}
{{<table "table table-striped table-bordered">}}
| name | genre_action | genre_rpg | genre_strategy | genre_simulation | genre_puzzle | genre_casual | genre_adventure | genre_platformer |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE |
| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |
| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |
| AdVenture Capitalist | FALSE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE |
{{</table>}}

Aaaaand nothing has changed. This is good news for stability, and bad for improving our labels. Here I get another idea: what if I add price to the input data? Since platformers are usually cheaper than AAA games, maybe this would help.


{{<gpt>}}
# List of steam games
<br>
<br>name: DOOM
<br>description: Now includes all three premium DLC packs (Unto the Evil, Hell Followed, and Bloodfall), maps, modes, and weapons, as well as all feature updates including Arcade Mode, Photo Mode, and the latest Update 6.66, which brings further multiplayer improvements as well as revamps multiplayer progression.
<br>price: $19.99
<br>
<br>name: This War of Mine
<br>description: In This War Of Mine you do not play as an elite soldier, rather a group of civilians trying to survive in a besieged city; struggling with lack of food, medicine and constant danger from snipers and hostile scavengers. The game provides an experience of war seen from an entirely new angle.
<br>price: $19.99
<br>
<br>name: Divinity: Original Sin 2 - Definitive Edition
<br>description: The eagerly anticipated sequel to the award-winning RPG. Gather your party. Master deep, tactical combat. Join up to 3 other players - but know that only one of you will have the chance to become a God.
<br>price: $44.99
<br>
<br>name: Portal
<br>description: Portal™ is a new single player game from Valve. Set in the mysterious Aperture Science Laboratories, Portal has been called one of the most innovative new games on the horizon and will offer gamers hours of unique gameplay.
<br>price: $9.99
<br>
<br>name: AdVenture Capitalist
<br>description: Welcome, eager young investor, to AdVenture Capitalist! Arguably the world's greatest Capitalism simulator!
<br>price: Free to Play
<br>
<br># Table of games and their genres (TRUE or FALSE)
<br>
<br>| name | action | rpg | strategy | simulation | puzzle | casual | adventure | platformer |
<br>| --- | --- | --- | --- | --- | --- | --- | --- | --- |{{<gpt-comp>}}&nbsp;
<br>| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE |
<br>| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |
<br>| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |
<br>| AdVenture Capitalist | FALSE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE |{{</gpt-comp>}}
{{</gpt>}}

{{<table "table table-striped table-bordered">}}
| name | action | rpg | strategy | simulation | puzzle | casual | adventure | platformer |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |
| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE |
| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |
| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |
| AdVenture Capitalist | FALSE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE |
{{</table>}}

Still, there are no changes. At this point, I am inclined to stop. The responses are rather stable, and the labels, while not perfect, are pretty good.

# Limitations

Let's talk a little about the limitations of this approach before we go further.

## Stability of responses {#stability}

We have to come to terms with the fact that everything influences GPT-3 response to a certain extent. This is inherent to the nature of the approach. Anything can change the probabilities of the responses: the wording of the prompt, a change in capitalization, a change in the order of the labels and samples[^teststab]. We're hoping that the responses are stable enough that we don't have to think too much about it.

[^teststab]: It would be an interesting project to test the stability of the responses for some of these factors properly. 

That said, in manual labelling, there is the same issue. Different people will label samples differently, people are also influenced by instructions and wording, by cognitive biases, such as the [anchoring effect](https://en.wikipedia.org/wiki/Anchoring_(cognitive_bias)). So in reality it is not so different.

## Truthfulness and typos

How correct are the responses of the model? There is a whole area of research dedicated to Large Language Model truthfulness (see [this](https://www.alignmentforum.org/posts/PF58wEdztZFX2dSue/how-truthful-is-gpt-3-a-benchmark-for-language-models) for example). Since the training set contains a lot of information from the web, nothing guarantees that it will be correct. In some ways, we are relying on the fact that the training set contains much more correct information than false.

However, there is another fun point, relating to the stability of responses. Depending on the problem, while GPT-3 may be able to solve it in principle, it may "think" that our prompt implies some degree of incorrectness. How's that?

Think about an example: we try to generate responses to a math test. If our prompt implies that this is the control version of the test, it will tend to be correct. But if the prompt implies that this is a response of a student to the test, it may be slightly incorrect.

How does it relate to our problem? Well, if you make a typo somewhere in your prompt, while GPT-3 will for sure understand what you meant, it might "think" that the text it is generating is not, let's say, "written by a professional". This could lead to the response also being somewhat mistaken. So check your prompt several times for typos and wording.

## Number of samples {#number-of-samples}

How many samples can we insert in one request? GPT-3 has a hard limit on the total number of tokens (prompt + generations) -- **2048** tokens[^ntok]. Each token is 4 characters on average, so we can say that in total we can use around 8400 characters.

[^ntok]: Actually, the newest models are supposed to support 4096 tokens, according to the [documentation](https://beta.openai.com/docs/api-reference/completions/create#completions/create-max_tokens)

Depending on the size of each sample, we can determine the maximum number of samples to be put in on request. However, since GPT-3 charges per token, we're not saving a lot by cramming as many samples as possible in each request. I prefer to stay way below the limit, usually at 5-7 samples per request.

# What's next

In this post, we've formatted the responses as Markdown tables, limited responses to TRUE / FALSE to improve stability, experimented with prompts and discussed some limitations and considerations of this approach. 

In the [next post]({{< ref "/post/gpt3_3" >}}), we will write Python code to automate requests and parse the responses to get a set of labels for each sample.