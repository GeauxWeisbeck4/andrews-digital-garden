---
id: 01J5K2XJ7DYEH9AT0126N3VTWE
title: Mental Health Chat Bot Apps
description: A look at how to develop mental health chat bot apps
modified: 2024-08-18T11:59:53-04:00
tags:
  - mental-health
  - artificial-intelligence
  - machine-learning
  - rag
  - vectors
  - db
  - tidb
  - hackathon
---
# Mental Health Chat Bot App
- Overview of pipeline architecture
![[Pasted image 20240818115136.png]]

- **Speech Recognition**
Depending on how the chatbot is set up, we use off-the-shelf algorithms that transcribe user input into natural text. For text-based chatbots, this step would be excluded. 
- **Natural Language Understanding**
This step aims to process the user input using natural language processing. In a nutshell, the system tries to grasp the user’s intent and emotional state (sentiment analysis). The software also tries to extract various entities (parameters / attributes) relating to the user’s input. 
**![entities intent](https://topflightapps.com/wp-content/uploads/2022/08/Entities-Intent.png)
- **Dialog and Task Manager**
This module tries to control the flow of dialogue. This is based on the different pieces of information that are stored throughout the conversation. In other words, the dialogue manager develops a strategy (or rules) to effectively navigate the dialogue based on user input information and the overall context.
- **Natural Language Generation**
The dialog manager decides how to respond to the user, and the natural language generation module creates a response in a human-friendly format. Responses can be predefined or have a more free form. 
- **Text-to-Speech Synthesis**
As an optional step, the speech synthesis module converts the text back to speech so the user can hear it.
## **Latest NLP Developments Concerning Chatbots**
State-of-the-art NLP algorithms typically use large neural networks (so-called deep learning) to learn the nuances of language and to perform complex tasks with minimal intervention (i.e., natural language understanding and natural language generation). 
Here are the main models used in such linguistical neural networks:
### **Generative model**
Allow you to train a model to generate (output) results that have similar characteristics to what has been trained on the dataset.
![generative model](https://topflightapps.com/wp-content/uploads/2022/08/generative-model.png)Part of this process involves language modeling. Essentially, we are trying to  answer, “How can we best predict the next sequence of words (outputs) given an initial sequence of words?”
**Example**: If you train the model with sufficient “therapy” data, the implication is that the model will act in ways that are statistically consistent with what a “therapist” would say

### **Pretrained Model**
You train the model on a much more general data corpus and then apply a smaller annotated data set so that the model can be fine-tuned based on the specifics of your problem.
**![pretrained-model](https://topflightapps.com/wp-content/uploads/2022/08/pretrained-model.png)Example**: Large NLP models are generally trained on massive amounts of data (corpus) found on the internet. The model can learn to respond more appropriately if we have a more dedicated dataset for it to focus on (i.e., in addition to the large general knowledge base from which the model is pretrained).
### **Transformer Model**
Neural network model that tries to take into account how the words are used in context (instead of just one word at a time). This model can track the relationship between words in a sentence and decide what parts of the word sequence are worth attention.
## **What Tools Can We Use to Build a Mental Chatbot?**

What tools would be most appropriate in building a mental health chatbot? It depends on a number of factors, including but not limited to:

- Whether the conversation is goal-oriented, or whether we want it to be more open-ended?
- If the conversation is goal-oriented, do we already have the conversation flow mapped out?
- How much real-life conversational data do we have on hand?
- What general features are we looking for? Do these features come right out-of-the-box, or do they have to be custom-made, specific for the application?
- What platforms do we want to integrate the chatbot with (e.g., mobile application, web application, messenger, etc.)?

Some chatbot-building systems, for example, DialogFlow, come with certain integrations out of the box.

- Do we need such features as speech-to-text (STT) or text-to-speech (TTS)?
- Does the chatbot need to support multiple languages?
- Does the chatbot need to make backend API calls to use collected or external data?
- What type of analytics do we plan to collect?
### **Goal-oriented therapeutic bot building**

Consider using a goal-oriented chatbot if you want to guide your patient through a fixed set of questions (or procedures) during a counseling session. You should also consider this approach if you have very limited conversational data or need explicit guardrailing to frame how the conversation would flow.

#### **DialogFlow**

DialogFlow is one of the most popular dialog-building cloud platforms by Google. We can use [DialogFlow’s web user interface](https://dialogflow.cloud.google.com/) to start with development right away (min initial coding or installation required). Then, for more advanced scenarios, they provide the DialogFlow API to build advanced conversational agents.

#### **DialogFlow ES**

In general terms, we map out how a potential conversation can unfold. DialogFlow then uses NLP to understand the user’s intent (and collect entities) and, based on that, makes appropriate responses.

![dialog flow ES](https://topflightapps.com/wp-content/uploads/2022/08/Screenshot-2022-08-31-at-3.15.31-PM-1024x498.png)The NLU gets better as we collect more conversation data. The conversation flow is maintained based on how DialogFlow explicitly keeps track of context. The ES version is rule-based and is meant to cover more straightforward applications. 

_Also Read: [How to Develop a Natural Language Processing App](https://topflightapps.com/ideas/how-to-build-a-natural-language-processing-application/)_

Replies are generally static (i.e., based on predefined options). However, responses can be made more dynamic by calling the platform’s backend APIs with webhook requests. Here’s how DialogFlow works in the back end:

_![dialog flow](https://topflightapps.com/wp-content/uploads/2022/08/dialog-flow-1024x637.jpeg)_

 _(image credit: Lee Boonstra, from “The Definitive Guide to Conversational AI with Dialogflow and Google Cloud”)_

_Read more on_ [_conversational AI in healthcare_](https://topflightapps.com/ideas/conversational-ai-in-healthcare/)_._

I’d like to emphasize that the platform has the most necessary features out of the box, but the trade-off is that DialogFlow is a closed system and requires cloud communications. In other words, we can’t operate it using on-premise servers.

Some of the highlight features that come off-the-shelf include:

- support for speech-to-text and text-to-speech
- options to review and validate your chatbot solution
- integrations across multiple platforms
- built-in analytics

#### **DialogFlow CX**

Whereas DialogFlow ES should be reserved for simple chatbots (i.e., if you are a startup or small business), we recommend using DialogFlow CX for more complex chatbots or when developing an enterprise application. The core principles behind ES still apply to CX.

Here’s how DialogFlow CX helps you build a more advanced mental health chatbot:

- supports large and complex flows (hundreds of intents)
- multiple conversation branches
- repeatable dialogue, understanding the intent and context of long utterances
- working with teams collaborating on large implementations

**![dialog flow CX](https://topflightapps.com/wp-content/uploads/2022/08/dialog-flow-CX.jpeg)**

#### **Rasa**

In general, Rasa would be used in scenarios where customization is a priority. The trade-off behind this customizability is that more tech knowledge (e.g., Python) is required to use these components. In addition, it requires more upfront setup (including installation of multiple components) compared to using DialogFlow (minimal or no upfront setup).

You should also keep in mind that Rasa has no or minimal user interface components that come out of the box.

Rasa has two main components: Rasa NLU and Rasa Core

#### **Rasa NLU**

Rasa NLU is fundamentally similar to how DialogFlow ES works. The platform can match user intents and collect entities / parameters during conversations. The natural language understanding gets better with more conversation data being gathered and analyzed.

The NLU components are designed with customization in line when building the pipeline: we can swap out components at different stages depending on the use-case or application needs:

**![NLU](https://topflightapps.com/wp-content/uploads/2022/08/nlu.jpeg)**

#### **Rasa Core**

Rasa Core has a more sophisticated dialog management system compared to DialogFlow. The dialogue in Rasa is more explicitly driven based on stories, which organizes the order of what intent, action, and parameters are collected during model development. It is also possible to weave multiple stories together using custom logic.

![rasa stories](https://topflightapps.com/wp-content/uploads/2022/08/Rasa-Stories.png)

Since Rasa is a code-first framework, there is a greater degree of flexibility on how you can design conversations compared to DialogFlow. Generally, responses are static (predefined options) but can be made more dynamic using API calls. 

Here are some additional features you can expect from Rasa:

- more flexibility in importing training data across multiple formats
- open-source libraries
- hostable on your own servers
- Rasa Enterprise includes analytics

Also, note these features that the Rasa platform lacks:

- no hosting provided
- no out-of-box integrations
- no speech-to-text or text-to-speech (optionally, third-party libraries)
### **Open-ended therapeutic bot building**

We recommend creating an open-ended chatbot if you want conversations to be more in the form of chit-chatting (i.e., an AI therapist providing open-ended, empathetic conversations for those going through a rough time).

It’s strongly recommended that you have some conversational data to work with to fine-tune the model and minimize the risk that a transformer model (more on that below) will output inappropriate responses.

Fortunately, the GPT-3 and BlenderBot platforms work decently out of the box for creating open-ended dialog agents, but you get the most out of either model with fine-tuning. That’s necessary to minimize the risk of inappropriate responses.

#### **GPT-Based Models**

GPT-3 uses a transformer model that is trained on large text corpora across the internet. In terms of training data, GPT-3 models are trained on a variety of text such as Wikipedia, corpora of books, and various web pages (for example, high-quality ones like WebText2 and petabytes of data such as Common Crawl).

GPT-3 is considered a game-changer in the field of NLP AI in that it can be used for a large number of NLP tasks:

- text classification
- text summarization
- text generation, etc. instead of just one specific task

Fundamentally, OpenAI’s GPT-3 changes how we think about interacting with AI models. The OpenAI API has drastically simplified interaction with AI models by eliminating the need for numerous complicated programming languages and frameworks.

The quality of the completion you receive is directly related to the training prompt you provide. The way you arrange and structure your words plays a significant role in the output. Therefore, understanding how to design an effective prompt is crucial to unlocking GPT-3’s full potential.

![gpt 3](https://topflightapps.com/wp-content/uploads/2022/08/gpt-3.webp)

GPT-3 shines best when you:

- Supply it with the appropriate prompt
- Fine-tune the model based on the specifics of the problem at hand

_Related: [Medical Chatbots: Use Cases in the Healthcare Industry,](https://topflightapps.com/ideas/chatbots-in-healthcare/)_ _[How To Build Your Own ChatGPT Chatbot,](https://topflightapps.com/ideas/how-to-build-chatgpt-like-chatbot/)  [How is ChatGPT Used in Healthcare](https://topflightapps.com/ideas/chatgpt-in-healthcare/)_

Also, please note some of the drawbacks when considering GPT-3 for making a mental health chatbot:

- AI bias

Since GPT-3 is trained on large amounts of data, there’s a risk of it using toxic language / saying wrong things (i.e., that are reflective of the dataset being used for training). For example, here’s a [story](https://www.artificialintelligence-news.com/2020/10/28/medical-chatbot-openai-gpt3-patient-kill-themselves/) where a GPT-3 based bot agreed that a fake patient should kill themselves. Fortunately, this issue can be addressed through careful fine-tuning (i.e., retraining the model on focused datasets so that it can emulate proper conversation behavior).

- OpenAI has a tendency to restrict API access for higher-risk applications. You’ll need permission to use their GPT-3 technology in a live app.

Consider online platforms as GPT alternatives, such as [Forefront](https://www.forefront.ai/), that allow you to leverage their computational resources with ease / minimal restrictions if the above poses an issue.

![forefront statement](https://topflightapps.com/wp-content/uploads/2022/08/ForeFrontStatement.png)You can also consider other alternatives, such as [GPT-2](https://huggingface.co/docs/transformers/model_doc/gpt2) (i.e., previous iteration) or possibly [DialoGPT](https://huggingface.co/microsoft/DialoGPT-medium) for offline model training and use fine-tuning if needed. Unfortunately, the trade-off is that the above models have fewer parameters. And so, the depth and quality of responses may be less impressive compared to GPT-3.

Here’s a [prototype](https://topflightapps.com/mobile-app-prototyping-services/) of therapeutic chatbot we put together at Topflight, using a GPT-3 model:

# Resources
- [How to Build an AI-Powered Mental Health Companion: A Step-by-Step Tutorial](https://www.linkedin.com/pulse/how-build-ai-powered-mental-health-companion-tutorial-dhaval-bhatt-cxdrc/)
- [Creating a mental health resource chatbot with a deep neural network. | by Alisha Arora | Medium](https://alishaarora56.medium.com/creating-a-mental-health-resource-chatbot-with-a-deep-neural-network-89b806dc87d0)
- [Andela | Build an AI mental health chatbot](https://www.andela.com/blog-posts/ai-health-innovation-building-a-mental-health-chatbot-using-fastapi-langchain-and-openai-in-python)
- 