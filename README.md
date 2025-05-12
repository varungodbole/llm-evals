# A strategy for meaningful LLM evaluations

**Authors:** Varun Godbole, Claude 3.7, ChatGPT o3.

**Substack:** www.varungodbole.com

## Who is this document for and what is its goal?
This document is for **engineers and product managers** that want to create **tangible value** from shipping **LLM-powered software**. It’s for readers who don’t have an ML background. But have the inclination to approach problems empirically and from first principles.

The word “evaluation” or “eval” is as broad as the word “testing”, which in traditional software development encompasses a wide array of testing patterns like unit testing, integration testing, load testing, etc. This can be overwhelming if one’s just getting started.

The **goal** of this document is to articulate ***why*** one should build evals, a definition of ***what*** they are, and an overall high-level process and philosophy of ***how*** they can be constructed. We leave the low-level implementation details of the ***how*** (e.g. specific libraries, tooling, etc) for the next document in the series. We’ve found that most engineers without ML experience often possess most of the requisite knowledge necessary to actually implement effective evals. However, they often lack an overall process of working through the why and what. We also offer a case study to make these ideas more concrete.

Eval construction is deeply contextual to the use-case, business constraints, etc. It’s impossible to provide The One True Way. Instead, this document presents *a specific lens* for eval construction borne from a decade of working in deep learning research, which we hope shall be helpfully provocative. In that vein, we also offer some anecdotes on eval construction.

We anticipate iterating on this document over time. It’s under Creative Commons on GitHub to encourage readers to fork/edit it to serve as onboarding material for their own organizations.
## Why, what and how \- the engineer's blind spot
Every problem is motivated by a *why*, *what* and *how*. Typically, the training and job role for many engineers (especially individual contributors) is to entirely focus on the *how*, rather than the *why* and *what*. The "failure mode" for many engineers in the face of ambiguity is to immediately start coding, at the risk of building the “wrong” thing.

Eval construction is not immune to this pattern. Constructing evals is hard and time consuming. A “bad” eval can profoundly misrepresent reality within one’s business, with all the downstream consequences that implies.

Therefore, one should always start eval construction with asking *why* they’re being created.
### Why create evals?
The typical answer is something like \- *“We want to understand whether the model actually works in production”*. Unfortunately, the model “working” in production isn’t necessarily indicative of business value. It’s an insufficient answer for *why* one creates evals.

A business instantiates a value chain of activities which transforms some raw resources into an output which produces values for its buyers. Businesses will often create various metrics/KPIs to track their ability to do this. The value of an AI deployment is proportionate to how aligned or integrated it is to this value chain. Moreover, the deployment should have a casual relationship to improve these overall metrics/KPIs.

But these metrics will often encompass more scope than what the AI deployment can tangibly influence. For example, the Monthly Active Users (MAUs) of an e-commerce website might have some casual relationship with the quality of an LLM-powered customer service chatbot. But it’s unlikely that changes in MAUs can be entirely explained by changes in the chatbot’s behavior. Therefore, it’s often necessary to create a measurement (or a series of them\!), which causally links an AI’s behavior to the top-level metrics.

That’s *why* a company should create evals for its products. It implies that what constitutes an “eval” can actually be fairly diverse. Perhaps more diverse than what many people typically imagine.
### What is an eval?
**An eval is a repeatable process that attempts to produce an answer to a question.**

As described above, the question should have significance to the business in terms of its top-level metrics or goals. There’s a lot of freedom in what these questions might look like, and the “right” answer is often deeply contextual. The answer could be binary (e.g. yes/no), categorical (e.g. good/bad/failure), quantitative (e.g. some percentage).

A good eval is one whose answer is *tangibly actionable* for the business. For example, it might inform whether a release candidate is fit for production. Or perhaps if the candidate is better than the baseline in production, one should prepare for more load from some increase in DAUs. 

The implementation of a good eval is deeply contextual, and there’s often substantial creativity involved. Notice that we haven’t made any assumptions about exactly how this repeatable process shall be executed. For example, whether it’s done by software or by a human. It could be as banal as a regex, or a PM that plays with their product every week, or paid raters that answer a set of rubrics. Nor have we made assumptions about how often it shall be executed. One might have fast and cheap evals akin to unit tests that are run everyday. Or one might have slower and expensive evals akin to integration tests that are run before every release.

The process of creating a suite of evals has strong connections with the overall process of framing the product or feature itself. This often informs what the initial set of evals will actually be.
### How to implement an eval?
There’s two levels to this question:
1. What is the high-level process or philosophy for structuring the evals?  
2. What tools, libraries, etc. should one use to construct good evaluations?

As discussed above, this document entirely focuses on (1). We leave (2) for the next document in the series.
## An iterative process for creating evals
Our process can be broken down into specific “Generate”, “Edit” and “Reflection” sections. Typically, engaging in this full process yields more insight into the overall product goals and problem formulation itself, leading to successive iteration.

**Generate**
1. On the demand side, what are the overall “jobs to be done” or “use-cases” that we believe could be assisted using an AI? What does this user journey or flow look like?  
2. What broader business metrics/KPIs do we already possess, or need to create to capture this flow?  
3. For each of these flows, what are the **actionable questions** from the section above that we’re trying to answer about the model’s performance? 

**Edit**
1. What is the stack rank of these different flows, and how would one weigh them against each other? Is there one that seems especially relevant or critical to the business?  
   2. Much like alleviating technical debt, the act of creating evals is an investment that must be made strategically. One can’t boil the ocean, nor can they measure everything in the workings of their business.  
3. If we had to pick one specific **actionable question** which we’d like an eval to answer, which one would it be and why?

**Generate**
1. For this actionable, what are some canonical diverse example inputs that a user might feed into the system? At this stage, we’re unconcerned with the context window or raw feature inputs. Our focus is the user.   
   2. It can be very helpful to get creative and err on the side of diversity. Consider what are likely to be “typical” input patterns along with lots of edge cases. Imagine we’re writing tests for an API. These input examples collectively point towards the overall “contract” for our AI system.  
   3. It’s not uncommon at this stage to gain more insight into the scope of the product surface itself.  
4. For each of these example inputs, generate a lot of different assertions that would need to be true for the AI system to produce the “ideal” output. One can think of these assertions much in the same way that one might think about an assertion in a unit test. Specifically, an assertion here is a question one might ask of the output given some input that results in a yes/no/not applicable answer.  
   5. might even be helpful to map out a sketch of what the “ideal” output might look like.  
   6. It’s fine if the assertions involve constraints that weren’t specified in the inputs. This is potentially an implementation detail that can be resolved later. At this stage, our priority is to map out the input/output “contract” of the AI system.

**Edit**
1. In practice, there’s a combinatorial explosion of things that one could evaluate. It’s worth looking for patterns in the assertions generated in the previous stage, and asking these two broad questions:  
   2. Are there any assertions that seem independent of the input, and seem contingent on the overall use-case or deployment itself?  
   3. Are there any assertions that seem heavily dependent on the input? For example, assertions that are heavily dependent on context?  
4. Once all the assertions have been scanned for patterns, it’s time to figure out the scope of the eval. The section below has more details.  
5. Once an eval has been scoped, build it and ship it\!

**Reflections \- What to do after the eval is complete?**  
Hill-climbing the eval is a no-brainer once it’s ready. That is, to change the overall ML system to proactively improve the eval results. For LLMs APIs in particular, this could involve tuning the system prompt, better management of the context window (e.g. RAG), tuning the sampling parameters, etc.

Sometimes improving the eval results doesn’t improve the higher-order business metrics, and it can be due to the following reasons.
* The “job to be done” or use-case is framed incorrectly.  
* The relevant business metric is too insensitive or unresponsive to changes in the product’s behavior.  
* The causal relationship between the relevant business metric and the eval is either too unclear, time-delayed or otherwise noisy. This can also be because an eval was constructed for the “wrong” actionable question.  
* The implementation of the eval itself can be improved. For example, perhaps it’s too noisy or merely has a bug in it.

Once the issues above are fixed, hill-climbing can resume. When the eval saturates, it's time to create a new eval!
## Determining the scope of the eval

For the first eval, it can be impractical to attempt to implement all the assertions generated in the process above. The following dimensions of scope affect an eval’s complexity:
* Whether every eval example will be run against the same set of assertions.  
* How many assertions will be run on each eval example.  
* The type of mechanism used to implement the assertion.

Running the same set of assertions on every eval example is simpler to implement than maintaining different assertions for each example. Running a single assertion for each eval example is simpler than running multiple assertions per eval example.

It’s better to err on the side of simplicity for the first eval. Often, one can collapse a number of “specific” assertions into a more “general” but fuzzier assertion. Over time, these can gradually be “unfolded” back into more specific assertions as needs for signal-to-noise evolve.

At a high-level, there’s a few different mechanisms for implementing each assertion:

1. **Deterministic checking** \- If the output is structured in some way (e.g. JSON dictionary or discrete classification task), you can simply deterministically check the actual value against some expected value.  
2. **Fuzzy checking** \- Unlike deterministic checking, it will have false positives and false negatives and checks the response heuristically. There's a vast array of strategies for implementing something like this. They range from using bags of regular expressions to using embeddings with cosine similarity against some golden response.  
3. **Human raters** \- A human examines each response against some assertion, and decides if the assertion is true or not.   
4. **LLM raters** \- An LLM is prompted to simulate a human examining each response against some assertion, to decide if the assertion is true or not.

In practice, deterministic checking is far simpler than the other options discussed above. LLM raters are probably the most “dangerous”, since they’re trivial to wire up but extremely difficult to get right. As described in [this prompt tuning playbook](https://github.com/varungodbole/prompt-tuning-playbook), LLMs are really great role-players that are trained to role-play human raters paid to role-play as AI assistants. Human data collection is extremely difficult to implement in practice, and using LLMs merely imports a lot of that complexity along with some hidden and unintentional biases that LLMs may possess.

Prioritizing the scope of an eval is extremely contextual, and it’s extremely difficult to prescribe an algorithm for this. But we’d recommend practitioners ask themselves the following questions:

1. What’s the simplest, dumbest eval in terms of the dimensions of scope described above that will provide the most actionable signal?  
2. Has at least one engineer on the team manually evaluated a handful of model outputs to convince themselves that they actually experientially understand the eval task?

## Case study \- chatbot for an internet service provider

We’ll attempt to concretize the overall process described in this document by considering an LLM-powered customer service chatbot called ChattyMcChatFace for a large Internet Service Provider (ISP) named FakeISP. The next document in the series shall actually implement the eval using [promptfoo](https://www.promptfoo.dev/).

LLMs have the capacity to act as a cognitive prosthetic for the mind. They can be extremely effective at augmenting our creativity, especially for the “generate” stages of the process described above.
### Prompt Template {#prompt-template}
Let’s consider the following prompt template.

```
## Project Context  
I work for a large internet service provider, and my goal is to design a customer service chatbot. This chatbot will be available to users on my company's website. Our hope is that deploying this chatbot will improve the overall user experience since the bot will be able to be infinitely patient, kind and professional. Of course, the user will also have some way to escalate to a human representative if necessary. However, we hope that over time, we'll be able to translate an increasing amount of workload off the humans to this bot.

## Chatbot Scope

We've identified a few use-cases for the initial deployment of this chatbot.

- It should provide general technical support. That is, when users experience difficulty accessing their internet, navigating the website or paying their bills.  
- It should be able to answer questions around the ISP's current offerings. For example, perhaps the user might have questions about whether their current plan is the best for them.  
- If a user decides to churn on their internet connection, it should offer them promotions and otherwise persuade them to not churn.  
- It should not do anything outside of the overall scope above. For example, it should not attempt to sell users fruits and vegetables or step outside the bounds of propriety and professionalism.

## Chatbot Limitations

- It shouldn't have access to the internet.  
- We'll likely improve the overall grounding of the chatbot using retrieval-augmented completions.

## Business Metrics

There are a number of business metrics we're trying to improve with this chatbot:  
- Reduce OpEx - we'll ultimately track this by looking at how often the user asks to escalate to a human, and the overall volume sent to humans.  
- User happiness - we periodically run surveys where users can self-report their overall happiness.  
- Customer retention - we currently track what percentage of our customers have remained with us over quarters and years.

## Your Task

<%= it.taskDescription %>
```

`<%= it.taskDescription %>` can be replaced with a task description for each generate task.
### Mapping out the key use-cases or "jobs to be done"
We’d expect these to be supplied by the product owner, and for them to be aligned with the company’s overall strategy. We’ve arbitrarily come up with some use-cases below for pedagogical purposes:
* It should provide general technical support. That is, when users experience difficulty accessing their internet, navigating the website or paying their bills.  
* It should be able to answer questions around the ISP's current offerings. For example, perhaps the user might have questions about whether their current plan is the best for them.  
* If a user decides to terminate their internet connection, it should offer them promotions or otherwise persuade them against churning.  
* It should constrain itself to the scope described above. For example, it should refrain from selling users products not sold by the ISP (e.g. fresh produce), and should remain within the bounds of propriety.

In the interests of expediency, we’ll focus on the first use-case for the rest of this case study.
### Generating actionable questions
**Use case:** *Provide general technical support. That is, when users experience difficulty accessing their internet, navigating the website or paying their bills.*

**Metric:** Customer retention

Given this use-case and metric, some actionable questions immediately come to mind:
* How often are the chatbot’s responses useful?  
* Users would be self-selecting into this use-case when they're likely experiencing genuine difficulty with our products. How often are the chatbot's responses simultaneously empathetic and professional?  
* How often does the chatbot confabulate its proposed solutions? Does it fail gracefully when it doesn't know something?  
* How often does the chatbot engage in unsanctioned promotional behavior on behalf of the company?  
* How often does the chatbot "break out" of its persona as ChattyMcChatFace? For example, generating Python code if prompted or doing anything else outside the bounds of its persona?

We can use the prompt template above along with the following task description to generate more actionable questions.

```
In particular, we're really focused on the use case of our chatbot providing general technical support. That is, when users experience difficulty accessing their internet, navigating the website or paying their bills. Our broader thesis is that users that experience a high quality of technical support are more likely to be retained. Retention is the overall top-level business metric that we care about for this project.

We'd like to brainstorm the right sets of evaluations for this use-case. Our preferred lens for thinking about evals is that they're a repeatable process that produces an answer to a specific question. And a good eval is one that produces an answer to a question that's actionable in terms of a decision we'd make within the business. For example, a broad actionable question for this use-case could be `How often are the chatbot's responses helpful?`. More targeted questions could be:
- Users would be self-selecting into this use-case when they're likely experiencing genuine difficulty with our products. How often are the chatbot's responses simultaneously empathetic and professional?  
- How often does the chatbot confabulate its proposed solutions? Does it fail gracefully when it doesn't know something?  
- How often does the chatbot engage in unsanctioned promotional behavior on behalf of the company?  
- How often does the chatbot "break out" of its persona as `ChattyMcChatFace`? For example, generating Python code if prompted or doing anything else outside the bounds of its persona?

Your job is to help me brainstorm a lot more of these sorts of questions. Specifically, please reply with 20 other questions for this use-case that we might want to routinely generate answers for via some repeatable process. Please ensure that each of those questions is maximally diverse from the other. Let's get generative and cover a very wide base here.
```

Notice that the task description was conditioned with the actionable questions we manually generated above. This can be a double-edged sword. On the one hand, it’ll make it easier for the model to generate questions of the same sort that we’ve already generated. On the other hand, it’ll make it harder for us to pierce through the bubble of our own projections. It can be helpful to play with both possibilities in a multi-turn conversation. It might also be helpful to copy/paste this document into the context window so the model has a better idea of what we’re asking it to do.

Here's an [example](https://chatgpt.com/share/681b5145-a398-8006-8b03-62ab93e28663) of such a conversation generated with ChatGPT o3.
### Generating example inputs and assertions
Since this is our first eval, we’ll attempt to choose the most pressing actionable question. In this context it would be \- *How often is the chatbot’s response useful?*

We’ll now generate a lot of different example inputs that a user may have. Users of an ISP may encounter a large diversity of situations styming their internet access. 

This might be a useful opportunity to further specify what is or isn’t in scope for “technical support”, if we haven’t thought about it already. For example, suppose a user has issues paying their bill on the ISP’s website. Would that be considered in scope for the “technical difficulties” that the chatbot is trying to tackle?

Eventually we might generate a long list of such example prompts:
* All the lights on my modem are on, but I can’t access the internet from my phone.  
* Why is it taking so long to visit [youtube.com](http://youtube.com)?  
* How can I measure the speed of my internet connection?

For each prompt, we’ll then brainstorm assertions that we’d expect to be true.

Here’s a few examples:
- All the lights on my modem are on, but I can’t access the internet from my phone.
	- Does the response ask the user to take a photo of the modem?  
	- Does the response clarify whether the user is connected to the modem’s WiFi account?
	- Is the response terse, yet polite?  
* Why is it taking so long to visit [youtube.com](http://youtube.com)?  
	* Does the response ask the user to check if they experience high latency for any other websites?
	* Does the response only provide a single intervention for the user to perform?  
* How can I measure the speed of my internet connection?
	* Does the response tell the user to visit [www.speedtest.net](http://www.speedtest.net)?
	* Does the response tell the user to search for “speed test” in Google?

Once lots of example inputs and their corresponding assertions have been generated, it can be helpful to use an LLM to expand the envelope of possibility. We can use the same prompt template from above, with appropriately adjusted task descriptions to generate prompts and their corresponding assertions. It’s helpful to engage in multi-turn conversations to identify systematic patterns we’ve failed to consider.
## When the rubber hits the road - some helpful anecdotes
As the process above demonstrates, the creation of evals can be extremely contextual and strategic. It’s impossible to prescribe a one-size-fits-all. In the face of such complexity, we hope the anecdotes in this section will impart a specific *perspective* with which we view the realm of evals.
### Deterministic versus stochastic software
LLMs represent a different *kind* of software. They invite different cultural processes for developing and delivering this *kind* of software.

Engineers receive training to create deterministic software that fulfills specific business logic contracts. This paradigm treats all non-determinism as *errors* requiring elimination or careful management. Cultural processes and engineering interactions have evolved specifically to deliver this type of software. Test-driven development emerged logically from organizations needing hermetically testable, deterministic software.

Working within a stochastic paradigm changes everything.

ML models (specifically LLMs), are inherently stochastic in both their training and inference, which necessarily injects complexity of their use. This increased complexity makes them tools of last resort when all other more deterministic approaches won’t work. However, there are many valuable problems that possess underlying structure which we either don’t understand sufficiently or can’t articulate well enough for traditional deterministic approaches. When we integrate LLMs into a product surface, we necessarily trade off the *formal certainty* of deterministic software for various notions of *statistical confidence* available in stochastic software.

To put it another way \- senior engineers can evaluate PRDs specifying standard CRUD applications to determine the feasibility of implementation with some *certainty*. However, each new ML problem represents its own research challenge. Teams must construct evals and run experiments. Past experience might suggest a ballpark level of performance on some new task, but there’s no *certainty* here. The best we can hope for is *confidence* that the model’s performance might improve based on what tricks have already been applied.

This shift necessarily shifts how teams incorporating ML must operate. Technical leads need to embrace the inherent nebulosity and technical risk in every ML project which wouldn’t have been there when delivering deterministic software.

Organizations that foster cultures which value the process of accelerating the experimental flywheel rather than one specific experimental outcome seem to perform the best. In the same vein, starting small and jointly expanding the scope of both the product and evals seems to work best.
### “Real world” evals always feel a bit shit
Evals can only be implemented to the extent that the implementor already understands the problem. Therefore, a product’s evals are a *lagging indicator* of the team’s understanding of the problem itself. In fact, the process of creating evals can often provide clarity on the overall product’s framing. On the other hand, evals can be very costly to make. There’s often a combinatorial explosion of things that *could* be evaluated in tension with what the team can *afford* to evaluate. Moreover, investments here are often in tension with other engineering work.

So a sense of overwhelm at the start is quite normal. Especially because the first eval might not capture the full complexity of the problem. Not even close. Furthermore, they might shine a light on various ambiguities and unresolved questions in the product’s framing itself. Therefore, one’s initial evals often feel kind of “shit” because of the gaping hole of content they don’t initially capture.

Eval construction can also be a Sisyphean task. Once an eval is constructed, it’s rational for the team to hill-climb and saturate it. At which point, a new eval is immediately required.

>*A work of art is never finished, only abandoned.*
>	- A translation of a longer quote by Paul Valery
>	- *https://www.goodreads.com/author/quotes/141425.Paul\_Val\_ry?page=5*

An eval is never finished, only abandoned. The natural conclusion is that the eval always feels kind of shit. In fact, assuming the team is conscientious, it couldn’t be any other way. Therefore, it’s worth growing a culture that values the process of spinning the flywheel itself, rather than any specific destination along the path.
### Confusion is bad \- as above, so below; as below, so above
There’s a circular and feedback relationship between a product and its evals. A confused product engenders confused evals, engendering an even more confused product, and so on.

In many ways, a product’s eval specification can be even more consequential than its Product Requirements Doc, since it describes what the model should actually *do* in production.

Key Eng/PM/UX leads should carefully pore over a healthy sample of the product’s eval examples at least once. Perhaps even create some examples themselves to better understand the product. Leads abdicating this responsibility correlates shockingly well with the overall product going off the rails. Especially if the organization is large enough to staff a dedicated “modeling team” to relentlessly climb all their product evals.

In fact, healthy teams often have escalation pathways for IC engineers to clarify “issues” or “critical ambiguities” that they might notice within the evals. When effective, these questions might trigger hard questions around the product’s overall scope or normativity in specific important edge-cases. When leads abdicate the responsibility of making time for such escalations, engineering teams often make unilateral decisions that may not properly reflect the overall values of the product leading to suboptimal market outcomes.
### If the vibes are bad but the evals are good, the evals are probably bad

> *People are not rational animals, they are rationalizing animals.*  
   -Robert Heinlein

Most people are uncomfortable sticking their necks out in the presence of ambiguity. They crave certainty and often seek to delegate the determination of value to some “blessed” and “objective” source. Therefore, the quantitative nature of most evals often tempts humans to bestow upon them more authority than they rightly deserve. They forget that evals are an attempt to codify intangible contextual judgement into a series of numbers. That is, evals are “vibes” frozen in amber. But the real world is dynamic and messy. No two moments will have exactly the same context, and therefore the same problem formulation. Therefore, the quantitative judgements of an eval suite should be balanced with the qualitative intuitions of the development team.

It’s a good idea to subject each release candidate with various sorts of qualitative analysis before deployment. This could be as simple as someone with substantial domain expertise (e.g. product manager) playing with the system for half an hour to check its “vibes”. If such a person reports that the vibes are bad, but the evals claim the system is good, the system is likely bad somehow. 

### Evaluations datasets should be treated as "code"
Most tech companies often make their codebases broadly accessible within the company, and engender a strong culture of code reviews. They’ll make it possible for an engineer from disparate parts of the company to easily fix incremental bugs within the codebase. That is, the codebase is a shared asset with a shared culture which seeks to collectively gradually improve it.

It’s worth thinking about evals in a similar way. It’s worth checking in evals into version control so that it’s possible for engineers to:
* See clear provenance for each eval example.  
* Incrementally improve one or many eval examples via pull requests.  
* Instrument continuous integration for the eval examples to facilitate rapid releases.  
* Instrument linters that run over each eval example.  
* Create clear “owners” for different evals to ensure their continued maintenance.

In particular, this substantially lowers the friction for engineers to propose new eval examples as they dogfood their own products.

### Quality isn't a single number, it's a dashboard
As discussed above, a good eval provides actionable information to further a company’s goals. But real life is often very complex, multidimensional and rife with trade-offs. There’s often multiple actionable questions whose answers might live in tension.

There’s a very seductive pull to attempt to reduce the entire business into a single number that will always capture everything. There might well be such a number that is most important within a specific season of time. But “quality” for a real and complex product is often encapsulated in a dashboard rather than a single number. The best release decisions are often those that skillfully integrate a number of relevant quantitative signals with qualitative intuition about the product itself.

This integration is deeply normative and can engender substantial strategic clarity within a product. In fact, this process is connected with a company’s ability to craft a differentiated value chain as it seeks competitive advantage.
## Concluding thoughts and future work
The process of crafting LLM evals is extremely rich, and this document is our first foray into this space. 

Future work will explore the following ideas:
* Replace the "fake" case study with a more realistic case study. This should explore not just the high-level wrestling of what the eval should measure, but also an actual implementation of an eval using an OSS tool like [promptfoo](https://www.promptfoo.dev/).  
* Tooling to better automate the process of creating evals. I suspect that it should be possible to create an LLM-driven "eval generation wizard" to make a lot of the steps in this playbook a lot easier.  
* A bunch of instructional videos on using LLMs to iteratively explore and brainstorm ideas. For example, there's ways you can do this both by iterating on the prompt itself as well as via iterative multi-turn conversations. There's also a number of tricks you can use to condition the model to make it more "creative".

If there’s anything in particular that you’d like to see more resources for, please open a ticket in the Issue Tracker. Please also subscribe to my substack at [www.varungodbole.com](http://www.varungodbole.com), where I often write about LLMs.

