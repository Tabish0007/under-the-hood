Chapter 8

The Grounding Problem

Large language models are incredible at generating text. They can write code, summarize documents, and answer questions. But they have a fundamental flaw. They do not know what is true. They only know what is probable.

If you ask a language model a factual question, it will generate an answer that sounds correct. It will use the right vocabulary and the right structure. But the number it gives you might be completely wrong. It might have been invented on the spot.

This is the hallucination problem. And it is the single biggest obstacle to deploying language models in production.

The solution is grounding. Grounding means forcing the model to base its answer on a verifiable source of truth. If the model is answering a question about sales data, it should not generate a number from memory. It should query the database, retrieve the actual number, and then use that number to construct the answer.

This sounds simple. It is not. The challenge is that language models are probabilistic. They generate the next token based on a probability distribution. They do not have a built in mechanism for verifying that the token they just generated is correct. They do not have a built in mechanism for checking that the number they just cited exists in the database.

To solve this, you have to build a system around the model. You have to give the model tools. You have to give it a way to execute code, query a database, or call an API. And then you have to verify the output.

At a high level, the architecture looks like this. The user asks a question. The model generates a plan. The plan is a sequence of steps. Each step is an action. The action is executed by a separate system. The result of the action is fed back to the model. The model uses the result to generate the final answer.

This is called an agent. An agent is a language model that can take actions. It can use tools. It can observe the results of those tools and adjust its plan accordingly.

But an agent is not automatically grounded. The model can still hallucinate the output of a tool. It can still generate a number that the tool did not return. This is why you need a verification layer. You need to check that every claim the model makes is supported by the output of a tool.

This is the deterministic grounding gate. The gate sits between the model and the user. It intercepts the model's response. It parses the response. It extracts every numerical claim. It checks whether each claim is supported by the execution trace. If a claim is not supported, the gate rejects the response. It tells the model to try again, or it tells the user that the system cannot answer the question with confidence.

I built this system for a project called Agentic Data Analyst. The model was allowed to write Python code to query CSV datasets. It was allowed to execute that code in a sandbox. It was allowed to return the result. But before the result was shown to the user, the grounding gate checked that the numbers in the response matched the numbers that were actually produced by the Python execution. If they did not match, the response was rejected.

This is not just a safety feature. It is a trust feature. It is what allows you to deploy a language model in a context where correctness matters. If you are building a chatbot for a restaurant, you can tolerate some hallucination. If you are building a system that reports financial metrics to an executive, you cannot. The grounding gate is the difference between a toy and a product.

But grounding is only half the problem. The other half is evaluation. How do you know that your agent is actually working. How do you know that it is not overfitting to one benchmark. How do you know that it will generalize to the questions that your users will actually ask.

This is where cross benchmark evaluation comes in. You cannot evaluate an agent on a single dataset. You have to evaluate it on multiple datasets and then measure how well it transfers. If an agent is trained on dataset A and tested on dataset A, it will do well. If it is trained on dataset A and tested on dataset B, it will do worse. This is overfitting. It is the same problem that plagues traditional machine learning, but it is harder to detect in agents because the model is doing more complex reasoning.

I built a framework called Evolve Generalize to measure this. It compares the agent's performance on the target task to its performance on a baseline. It measures the upper bound of what the agent could achieve if it were perfectly grounded. And it measures the gap between the actual performance and the upper bound. That gap is the overfitting.

The lesson here is that LLMs are not magic. They are probabilistic systems that need to be constrained and verified. The engineer who understands this is the one who can build production AI systems. The engineer who does not is the one who ships a demo that fails in the real world.

In the final chapter we will step back and look at the big picture. We will look at what it means to be a platform engineer and how to build a career on top of these technologies.
