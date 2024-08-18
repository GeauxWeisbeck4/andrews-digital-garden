---
id: 01J5K0BB7YFNQY7DGJSEKD4CSB
title: Introduction to RAG
modified: 2024-08-18T11:10:55-04:00
description: Intro and overview of retrieval-augmented generation in LLMs
tags:
  - algorithms
  - artificial-intelligence
  - rag
  - vectors
  - machine-learning
  - llms
---
# Introduction to RAG
- High Level overview:
![[Pasted image 20240818110627.png]]

- Retrieval-augmented Generation (RAG) balances information retrieval, which finds existing information, with generation, which creates new content. When generation occurs without context, it can generate "hallucinations" - inaccurate / incorrect results. In customer support and content creation generally, hallucinations can have disastrous consequences. RAG prevents hallucinations by retrieving relevant context. It combines Large Language Models (LLMs) with external data sources and information retrieval algorithms. This makes RAG an extremely valuable tool in applied settings across many industries, including legal, education, and finance.
- ## Hallucinations
	- A machine hallucination is a false piece of information created by a generative model. Though some argue that this anthropomorphism is [more harmful than helpful](https://betterprogramming.pub/large-language-models-dont-hallucinate-b9bdfa202edf), saying that a machine "sees something that isn't there" is a helpful analogy for illustrating why machine hallucinations are bad for business. They are often named as one of the biggest blockers and concerns for industry adoption of [Generative AI and LLMs](https://fortune.com/2023/04/17/google-ceo-sundar-pichai-artificial-intelligence-bard-hallucinations-unsolved/). Google's inclusion of made-up information in their Chatbot new achievements presentation (February 2023) was [followed by a 7% fall in Alphabet's stock price](https://www.cnbc.com/2023/02/08/alphabet-shares-slip-following-googles-ai-event-.html). While the stock has since recovered and hit new historic highs, this hallucinatory incident demonstrated how sensitive the public, including investors, can be to the generation of incorrect information by AI. In short, if LLMs are put into production for use cases where there's more at stake than looking up restaurants on the internet, hallucinations can be disastrous.
- ## Retrieval and Generation
	- Let's take a quick look at our two components separately and what they are doing on a basic level, and then examine how they operate in their most common use cases.
		1. **Retrieval**: A retrieval model, usually called a "retriever," searches for information in a document or a collection of documents. You can think of retrieval as a search problem. Traditionally, retrieval has been performed using rather simple techniques like term frequency-inverse document frequency (TF-IDF), which basically quantifies how relevant a piece of text is for each document in the context of other documents. Does this word occur often in document A but not in document B and C? If so, it's probably important. The retrieved documents are then passed to the context of our generative model.
		2. **Generation**: Generative models, or generators, _can_ generate content without external context. But because context helps prevent hallucinations, retrievers are used to add information to generative models that they would otherwise lack. The most popular generative text models right now are without a doubt the LLMs of OpenAI, followed by Google's and Anthropic's. While these generators are already powerful out-of-the-box, RAG helps close the many knowledge gaps they suffer from. The retrieved context is simply added to the instruction of the LLM, and, thanks to a phenomenon called in-context learning, the model can incorporate the external knowledge without any updates to its weights.