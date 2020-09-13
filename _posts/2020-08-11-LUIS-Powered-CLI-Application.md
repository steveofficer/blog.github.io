---
layout: post
tags: F# ML LUIS
---

LUIS is one of Microsoft's Cognitive Services and provides a way to perform text recognition and entity extraction on pieces of plain text.

One of the predefined models they offer illustrates how it can be used. This model allows you to enter a sentence such as "Book me a flight from Edinburgh to Las Vegas" and identifies the intent "Book a flight" and the entities "Source = Edinburgh" and "Destination = Edinburgh"

In order to become a bit more familiar with this service and learn a bit more about how it works I set out to create a console application that would allow developers to see all the pending pull requests and deploy a given pull request to a test environment.
With this in mind I created a model that would support the following intents:
 - List all Pull Requests
 - List all Test Environments
 - Deploy a Pull Request to a Test Environment


Use the authoring portal to create the application

Entities
![Entity Designer](/assets/images/posts/2020-08-11/luis-entities.jpg)

Intents and utterances
![Intent Designer](/assets/images/posts/2020-08-11/luis-intents.jpg)

![Intent Utterance Designer](/assets/images/posts/2020-08-11/luis-utterances.jpg)

Train

Test
![Test Tool](/assets/images/posts/2020-08-11/luis-test.jpg)

Publish

Calling the prediction API

Putting it all together