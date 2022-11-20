---
title: "Kickstart text classification with GPT-3, part 3"
date: 2022-11-20
tags: ["ml", "nlp", "gpt3"]
selected: true
draft: false
---

# Into
In the [previous part]({{< ref "/post/gpt3_2" >}}) we created a table-based prompt and got the completion in the form of TRUE / FALSE values for each of our samples and labels. In this post, we will automate this process.

# What's an API?
API (Application Programming Interface) is a developer-friendly way to communicate programmatically with a particular service / application. APIs are usually used to automate actions and integrate services. API is the opposite of a visual GUI (Guided User Interface), which is what we normally use. In the previous posts, we've used GUI of GPT-3, but it also has an API which we will use now.

# Setting up GPT-3 API
For Python, there is a dedicated package `openai` which simplifies API calls, but there is also a way to [make requests with a URL](https://beta.openai.com/docs/api-reference/making-requests), which will work in any language capable of making HTTP requests.

First, install `openai` Python package:

```bash
pip install openai
```

Then, obtain your API key. Go to https://beta.openai.com/account/api-keys and click "+ Create new secret key". Copy the value which appeared on the screen (e.g. in my case it starts with `sk-neOy3i0r...`) and make sure to never expose it on the Internet. For example, do not commit scripts or notebooks containing this key, as anyone who has it will be able to use funds on your account.

In your Python script or notebook, import `openai` and specify the `api_key`:

```python
# import openapi SDK
import openai

# paste the API key here
openai.api_key = "sk-neOy3i0r..."
```

# Making a test request
Set the parameters for a test request. We limit `max_tokens` to 10, and set `temperature` to 0, which is better for classification tasks. If you want to know more about the parameters, take a look at the [API documentation](https://beta.openai.com/docs/api-reference/completions/create).

```python
# prepare dictionary with the parameters
params = {
    "prompt": "test",
    "engine": "text-davinci-002",
    "max_tokens": 10,
    "temperature": 0,
}
```

Finally, request GPT-3 completion:

```python
# get completion response
response = openai.Completion.create(**params)
response = response.to_dict_recursive()
```

If everything worked fine, the `response` should be something like[^err] this:

```python
{'id': 'cmpl-6BhAmdV1eevida1MWsIklSTB6kven',
 'object': 'text_completion',
 'created': 1668245584,
 'model': 'text-davinci-002',
 'choices': [{'text': '_split(X, y, test_size',
   'index': 0,
   'logprobs': None,
   'finish_reason': 'length'}],
 'usage': {'prompt_tokens': 1, 'completion_tokens': 10, 'total_tokens': 11}}
```

[^err]: If it didn't work, let me know in the comments, and I can help you troubleshoot the issue.

Let's see what we've got here. The important bit is `response["choices"][0]["text"]`, which in my case equals to `'_split(X, y, test_size'`[^py]. This is our completion. It is what we saw highlighted green in the GPT-3 GUI.

[^py]: Which is ironically also seems to be a bit of Python code, completely by accident. Yours can be different, because no one guarantees that GPT-3 is exactly the same at the time you are using it.

# Getting the completion for a real prompt
Now we can obtain the completion for our real task:

```python
params = {
    "prompt": """# List of games

name: DOOM
description: Now includes all three premium DLC packs (Unto the Evil, Hell Followed, and Bloodfall), maps, modes, and weapons, as well as all feature updates including Arcade Mode, Photo Mode, and the latest Update 6.66, which brings further multiplayer improvements as well as revamps multiplayer progression.

name: This War of Mine
description: In This War Of Mine you do not play as an elite soldier, rather a group of civilians trying to survive in a besieged city; struggling with lack of food, medicine and constant danger from snipers and hostile scavengers. The game provides an experience of war seen from an entirely new angle.

name: Divinity: Original Sin 2 - Definitive Edition
description: The eagerly anticipated sequel to the award-winning RPG. Gather your party. Master deep, tactical combat. Join up to 3 other players - but know that only one of you will have the chance to become a God.

name: Portal
description: Portal™ is a new single player game from Valve. Set in the mysterious Aperture Science Laboratories, Portal has been called one of the most innovative new games on the horizon and will offer gamers hours of unique gameplay.

name: AdVenture Capitalist
description: Welcome, eager young investor, to AdVenture Capitalist! Arguably the world's greatest Capitalism simulator!

# Table of games and their genres (TRUE or FALSE)

| name | genre_action | genre_rpg | genre_strategy | genre_simulation | genre_puzzle | genre_casual | genre_adventure | genre_platformer |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |""",
    "engine": "text-davinci-002",
    "max_tokens": 200,
    "temperature": 0,
}


response = openai.Completion.create(**params)
response = response.to_dict_recursive()
```

And I get this as a response:

```python
{'id': 'cmpl-6Bh7ouHD97KKO8eoauancd8eVCpcW',
 'object': 'text_completion',
 'created': 1668245400,
 'model': 'text-davinci-002',
 'choices': [{'text': '\n| DOOM | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE |\n| This War of Mine | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE |\n| Divinity: Original Sin 2 - Definitive Edition | FALSE | TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE |\n| Portal | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE |\n| AdVenture Capitalist | FALSE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE |',
   'index': 0,
   'logprobs': None,
   'finish_reason': 'stop'}],
 'usage': {'prompt_tokens': 369,
  'completion_tokens': 115,
  'total_tokens': 484}}
```

# Parsing the completion
Now I need to parse the result to obtain labels for each of the games and genres. Instead of writing some kind of parsing loop, I will use a little trick and pretend that our table is a CSV-like object and use `pandas.read_csv()`:

```python
from io import StringIO
import pandas as pd

# create a string buffer, which allow to pretend that a string comes from a file
s = StringIO("| name | genre_action | genre_rpg | genre_strategy | genre_simulation | genre_puzzle | genre_casual | genre_adventure | genre_platformer |" + response["choices"][0]["text"])

# parse the string as a CSV with separator = "|"
df = pd.read_csv(s, sep="|")
# discard first and last columns, as they are empty
df = df.iloc[:, 1:-1]
# remove spaces around column names
df.columns = [c.strip() for c in df.columns]
# remove spaces around values in the table
df = df.applymap(lambda x: x.strip())
# map values TRUE and FALSE from string to bool
mapper = lambda x: {"TRUE": True, "FALSE": False}.get(x, x)
df = df.applymap(mapper)
```

If everything worked well, we get a nice table like this:

![Example table](/true_false_table_example.png)

Now you can write automation code to request GPT-3 completion in a loop for many games, and concatenate the resulting tables to obtain a labelled dataset.

# What's next

In the next post, we will go 1 step deeper and obtain not only labels but also the probability of each label. This will give us a better idea of how confident GPT-3 is in its predictions and can be used to filter out low-confidence predictions.