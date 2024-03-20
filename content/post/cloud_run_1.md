---
title: "Developing your own Firebase Functions on Cloud Run -- any interest?"
date: 2024-03-20
tags: ["cloud-run", "gcp", "cloud-functions", "firebase"]
selected: true
draft: false
---

# Context

At my company, [Mosaiq Labs](https://mosaiqlabs.com), we're using Firebase for our SaaS product, and in particular we are using Firestore as our main DB and Firebase Functions as a backend. We've had a lot of success with this setup, but recently ran into some limitations.

In particular, as we discovered the hard way, Firebase Functions for Python can sometimes get stuck on access to database (in particular, using `firestore` client from `firebase_admin` library). This happens especially often when multiple invokations are generated quickly on cold starts (as few as 5).

It took us a while to figure this out with Google Cloud support, but it turns out that the Python runtime for Firebase Functions will not support this well, until they release async runtime for Python (which may not happen any time soon).

So, we decided to move some of our functions to Cloud Run. However, getting this to play well with Firebase is not straightforward. Especially tricky is local development, as you can no longer rely fully on Firebase Emulator.

Over the last 3 weeks, I've been working on this, and I've finally got it in a place where I am somewhat happy with it. I wanted to share my learnings here, as I couldn't find a good guide on this topic.

If there is any interest for this, please leave a comment. Writing this up will take some time, and I'd like to know if it is something people are interested in.

If that is a very niche topic, I'll just move on to something else.

If you are also stuck over this and need immediate help, feel free to reach out to me directly at [my email](mailto:s.antopolsky@gmail.com) or on [LinkedIn](https://www.linkedin.com/in/antop/). I'd be happy to help.
