---
layout: post
title: The Worst (But Only) Claude 3 Tokenizer
date: 2024-03-12 12:01:00
description: We reverse-engineer the Claude 3 tokenizer. Just ask Claude to repeat a string and inspect the network traffic.
author: Javier Rando
---

_Not an official implementation_. By _Javier Rando and Florian Tramèr._

Check our [code](https://github.com/javirandor/anthropic-tokenizer) and [Twitter thread](https://twitter.com/javirandor/status/1767602845111492685).

---

Anthropic recently released [Claude 3](https://www.anthropic.com/news/claude-3-family), a new family of large language models. However, they have not publicly released their tokenizer (yet?). But no worries! You can reverse-engineer the tokenizer by analyzing the generation streaming. Let me walk you through the reverse-engineering process.

> 💡 The idea is simple. Ask Claude to repeat some text and observe how the generation is streamed through the network. It turns out that Anthropic serves one token at a time!

## The reverse-engineering process

#### 1. Naive idea: inspect the network traffic

Can we figure out how specific strings are tokenized? As a first step, we inspected the network traffic after making a request to Claude through the Anthropic Workbench. The network traffic contains the detailed generation stream and some promising **text deltas**. Nevertheless, these deltas could simply be words (space separated), that not necessarily match the underlying tokenization.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0"></div>
    <div class="col-sm-6 mt-3 mt-md-0">
       {% include figure.html path="assets/img/blog-tokenizer-traffic.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0"></div>
</div>
<div class="caption">
    Network traffic inspection after a request through the Claude workbench.
</div>

#### 2. Verify the stream are not simply words

We want to inspect the traffic when Claude generates a string that we are somehow confident has more than one token. How can we do this? We simply ask Claude to repeat a string. We chose "asdfasdfasdf" and checked that OpenAI tokenizer returned >1 tokens. 

We ask Claude to repeat this string and inspect the network traffic as before. We find that the text deltas are small pieces of the string, so these are not space-separated words. But, are they tokens?

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0"></div>
    <div class="col-sm-6 mt-3 mt-md-0">
       {% include figure.html path="assets/img/blog-tokenizer-asdf.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0"></div>
</div>
<div class="caption">
    Network traffic inspection after asking Claude to repeat the string `"asdfasdfasdf"`.
</div>

#### 3. Check if the stream chunks are actual tokens

Claude Workbench allows you to limit the number of maximum tokens to generate. We repeat the previous experiment and limit the number of output characters to 1 and 2, respectively.Teh generations are "as" (1 token) and "asdf" (2 tokens). This is a strong indication that the text deltas are actual tokens. This worked for several experiments.

We also verify that the number of tokens obtained with our method matches those reported in the API usage. We find that the number of tokens reported by the API is always `number of text tokens + 3`. This may include start and end of sentence tokens.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
    {% include figure.html path="assets/img/blog-tokenizer-1tok.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
    <div class="col-sm mt-3 mt-md-0">
       {% include figure.html path="assets/img/blog-tokenizer-2tok.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Generations when limiting the number of tokens to 1 and 2.
</div>

#### 4. Write a nicer prompt and implement with Python

We look for a generic prompt that successfully instructs the model to copy **only** the text to be tokenized. We also implement our tokenization using the streaming functionalities in the official Anthropic Python library. You can find the resulting implementation [here](https://github.com/javirandor/anthropic-tokenizer). We consider this "tokenizer" does not expose any propietary information. If your usage of this tool falls within Anthropic's [Responsible Disclosure Policy](https://www.anthropic.com/responsible-disclosure-policy), we encourage you to follow their protocols.

**Store and share your reverse-engineered vocabulary**: The tokenizer will save locally all the tokens you extract from your text. You can share these with us, and we will maintain a joint vocabulary by merging everyone's knowledge.

## The result: the worst (but only) Claude 3 tokenizer

This is probably the **least efficient implementation of a tokenizer** (but it is also the only publicly available one that we know of!). This may be useful for experiments where tokenization plays an important role and spending some tokens is not a problem. It is unclear how faithful this tokenization will be, but our experiments suggest this is very likely a close approximation.

Hope you find these experiments insightful!