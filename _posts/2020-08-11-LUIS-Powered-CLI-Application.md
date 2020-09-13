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

## Entities
The first thing to do is define the entities that form part of the model. In my use case I only have 1 entity, namely the Pull Request.
![Entity Designer](/assets/images/posts/2020-08-11/luis-entities.jpg)

## Intents and utterances
The next thing to do is to define what intents the bot can perform. If entities are the nouns of the model, then intents are the verbs.
For each intent we have to provide examples of what a user might say, this is then used as the training set to teach the underlying Machine Learning algorithm how to
classify a user's input so that our client application can carry out the correct action.
![Intent Designer](/assets/images/posts/2020-08-11/luis-intents.jpg)

![Intent Utterance Designer](/assets/images/posts/2020-08-11/luis-utterances.jpg)

## Train
We then have to train the model, this is the point where the service looks through each intent and utterance and creates the model that "understands" a user's input.

## Test
There is a handy test utility in the portal that allows us to enter some text and see what the model predicts and what level of confidence the model has. Given the prediction we can then refine the model further by adding new utterances if necessary.
Remember, that if any new utterances are added then the model must be retrained to pick the changes up.
![Test Tool](/assets/images/posts/2020-08-11/luis-test.jpg)

## Publish

## Calling the prediction API
Calling the API is really simple, all the information is passed in as a URL with the response being sent back as a JSON object.

## Putting it all together