---
layout: post
title: Do not write that jailbreak paper
date: 2024-10-24 00:01:00
description: Jailbreaks are becoming a new ImageNet competition instead of helping us better understand LLM security. Some takes on how LLM jailbreak and security research should look like.
author: Javier Rando
---

Jailbreak papers keep landing on arXiv and conferences. Most of them look the same and jailbreaks have turned into a new sort of ImageNet competition. In this post, I write about the reasons that make me think most of these papers are no longer valuable to the community, and how we could maximize the impact of our work to improve our understanding of LLM vulnerabilities and defenses.

Let’s start with what *jailbreaks* are. LLMs are fine-tuned to [refuse harmful instructions](https://arxiv.org/abs/2204.05862). Ask ChatGPT to help you build a bomb, and it'll reply "I cannot help you with that". Think of this *refusal* as a *security feature* in LLMs. In a nutshell, jailbreaks *exploit* these safeguards to bypass refusal and *unlock* knowledge that developers meant to *be inaccessible*. Actually, the name comes from its similarities to [jailbreaking the OS in a iPhone](https://www.microsoft.com/en-us/microsoft-365-life-hacks/privacy-and-safety/what-is-jailbreaking-a-phone) to access additional features.

| What we have | What we want | What we do |
| :---- | :---- | :---- |
| Pre-trained LLMs that have, and can use, hazardous knowledge. | Safe models that do not cause harm or help users with harmful activities. | Deploy security features that often materialize as refusal for harmful requests. |

<br>

In security, it is important to [red-team](https://en.wikipedia.org/wiki/Red_team) protections to expose vulnerabilities and improve upon those. The first works on LLM red-teaming [(Perez et al., 2022](https://arxiv.org/abs/2202.03286); [Ganguli et al., 2022](https://arxiv.org/abs/2209.07858)) and jailbreaking ([Wei et al., 2023](https://arxiv.org/pdf/2307.02483)) exposed a *security vulnerability* in LLMs: refusal safeguards are not robust to input manipulations. For example, you could simply prompt a model to never refuse, and it would then answer any harmful request. We should think of jailbreaks as an <ins>evaluation tool</ins> for security features in LLMs. Also they help evaluate a broader control problem: **how good are we at creating LLMs that behave the way we want?**

Follow-up research found more ways to exploit LLMs and access hazardous knowledge. We saw methods like GCG, which optimizes text suffixes that surprisingly transfer across models. We also found ways to automate jailbreaks using other LLMs ([Shah et al., 2023](https://arxiv.org/pdf/2311.03348), [Chao et al., 2023](https://arxiv.org/abs/2310.08419)). These methods were important because they surfaced fundamentally new approaches to exploit LLMs at scale. 

However, the academic community has since turned jailbreaks into a new sort of ImageNet competition, focusing on achieving marginally higher attack success rates rather than improving our understanding of LLM vulnerabilities.  When you start a new work, ask yourself whether you are (1) going to find a new vulnerability that helps the community understand how LLM security fails, or if you are (2)  just looking for a better attack that exploits an existing vulnerability. (2) is not very interesting academically. In fact, coming back to my previous idea of understanding jailbreaks as evaluation tools, the field still uses the original GCG jailbreak to evaluate robustness, rather than its marginally improved successors.

We can learn valuable lessons from previous security research. The history of buffer overflow research is a good example: after the original "[Smashing The Stack for Fun and Profit](https://inst.eecs.berkeley.edu/~cs161/fa08/papers/stack_smashing.pdf)" paper, the field didn't write hundreds of academic papers on *yet-another-buffer-overflow-attack*. Instead, the impactful contributions came from fundamentally new ways of exploiting these vulnerabilities (like "return-into-libc" attacks) or from defending against them (stack canaries, ASLR, control-flow-integrity, etc.). We should be doing the same.

### What does meaningful jailbreak work look like?

A jailbreak paper I would like to see accepted in a main conference should:

* **Uncover a security vulnerability in a defense/model that is claimed to be robust**. New research should target systems that we know have been <ins>trained not to be jailbreakable</ins> and <ins>prompts that violate the policies used to determine what prompts should be refused</ins>. Otherwise, your findings are probably not transferable. For example, if someone finds an attack that can systematically bypass the [Circuit Breakers](https://arxiv.org/abs/2406.04313) defense, this would be a great contribution. Why? Because there is not any work that has systematically exploited this defense, and we will probably learn something interesting from such an exploit.  
    
* **Not iterate on existing vulnerabilities.** We know models can be jailbroken with role-playing, do not look for a new fictional scenario. We know models can be jailbroken with encodings, do not suggest a new encoding. Examples of novel vulnerabilities we have seen lately include latent-space interventions ([Arditi et al., 2024](https://arxiv.org/abs/2406.11717)), fine-tuning on unrelated data has unexpected effects in safeguards ([Qi et al., 2023](https://arxiv.org/abs/2310.03693)), or protections diluting on long contexts ([Anil et al., 2024](https://www-cdn.anthropic.com/af5633c94ed2beb282f6a53c595eb437e8e7b630/Many_Shot_Jailbreaking__2024_04_02_0936.pdf)). Think whether you can contribute a method that will become a new benchmark for robustness.

	  
	Another common problem is playing the wack-a-mole game with jailbreaks and patches. If a specific attack was patched, there is very little contribution in showing that a small change to the attack breaks the updated safeguards since we know that patches [do not fix the underlying vulnerabilities](https://arxiv.org/pdf/2403.05030). Do not get me wrong, it is cool to share with the community, but this is not a paper I would be excited to see accepted in a conference. 

* **Explore new threat models in new production models or modalities**. Models, their use cases, and their architectures keep changing. For example, we now have [fusion models](https://openai.com/index/hello-gpt-4o/) with multimodal inputs, and will soon have powerful [agents](https://arxiv.org/abs/2406.13352). The community should start thinking about new threat models and [safety cases](https://www.aisi.gov.uk/work/safety-cases-at-aisi). For instance, what vulnerabilities may arise from combining different modalities? Do existing safeguards transfer or do we need to come up with new methods? I have seen some nice attempts at this. [Schaeffer et al.](https://arxiv.org/abs/2407.15211) tried to find jailbreak images that transfer across models without success. A very nice follow-up project could look for images optimized on open-source models that transfer to production models. Also, there are new ways to [optimize attacks in novel multimodal fusion architectures](https://arxiv.org/abs/2410.03489) that will power the next-gen of models. Future work may think of more generalizable optimization objectives and interesting applications to e.g. speech.

However, the works I keep seeing over and over again look more like "we know models Y are/were vulnerable to method X, and we show that if you use X' you can obtain an increase of 5% on models Y". The most common example are improvements on role-play jailbreaks. People keep finding ways to turn harmful tasks into different fictional scenarios. This is not helping us uncover new security vulnerabilities\! Before starting a new project, try to <ins>think whether the outcome is going to help us uncover a previously unknown vulnerability</ins>.

### If you work on defenses, keep the bar high

Another common problem has to do with defenses. We all want to solve jailbreaks, but we need to maintain a high standard for defenses. This isn't new, by the way. I encourage you to read some of the [lessons learned](https://nicholas.carlini.com/writing/2020/are-adversarial-exampe-defenses-improving.html) from adversarial examples in the computer vision era.

If you work on defenses, I think you should take the following into account:

* **Reducing the attack success rate by 10% with simple methods is not valuable**. We already know that if we make the system more complex, it is going to be harder to attack. But we need to advance protections that target worst-case behavior\!  
* **Academics should be working on foundational defenses**. Industry is already taking care of scaffolding protections—filters here and there—to prevent misuse. I think academic work should take long-shot projects that try to understand the broader problem of robustly making models behave the way we want. I think [latent adversarial training](https://arxiv.org/abs/2403.05030) and [circuit breakers](https://arxiv.org/abs/2406.04313) are good examples of the work we should be aiming for.   
* **Please, be transparent and faithful in your evaluations.** I know claiming a perfect defense might make you a cool researcher for a while. But watch out, chances are [someone quickly breaks your defense](https://nicholas.carlini.com/writing/2020/are-adversarial-exampe-defenses-improving.html)\! Academia provides the perfect environment to take long-shots, fail, and collectively keep improving our methods to solve a very hard problem. Negative results can also be very valuable. You probably won't be able to solve this on your own\!  
* **Try your best to break your own defense.** You spent a lot of time building a defense and you really want to put it out there. You are probably missing the most important part of your work: doing an [adative evaluation](https://arxiv.org/pdf/1705.07263). Readers should know how your defense fails, and what they should work on next. I think you can write a great paper that says “we tried a new defense that looked great against existing attacks, but we found that method X can bypass it”. Again, this is not new and people have been asking for [proper adaptive evaluations](https://arxiv.org/pdf/1705.07263) for a long time.  
* **Release your models\!** A good defense should be tested by as many people as possible. Let the community red-team it.

### Should you work on that next jailbreak paper?

I do not know, you tell us. I would encourage all of us to think about the bigger problem we have at hand: **we do not know how to ensure that LLMs behave the way we want**. I think, by default, researchers should avoid working on new jailbreaks unless they have a very good reason to. I think answering these questions may help:  
* If my attack succeeds, are we going to learn something new about LLM security?  
* Am I going to release a new tool that can help future research better evaluate LLM security?  
* Is my attack an incremental improvement upon an existing vulnerability? Or in other words, does fixing an existing attack clearly fix my attack?

If you are interested in improving the security and safety of LLMs ([these two are very different](https://arxiv.org/abs/2405.19524)\!), jailbreaks have a small probability of taking you somewhere meaningful. I think it is time to move on and explore more challenging problems. I recently collaborated on an [agenda containing hundreds of specific challenges](https://arxiv.org/abs/2404.09932) the community thinks we should solve to ensure we can build AI systems that robustly behave the way we want.

#### Acknowledgements

I would like to thank Florian Tramèr, Edoardo Debenedetti, Daniel Paleka, Stephen Casper, and Nicholas Carlini for valuable discussions and feedback on drafts of this post.