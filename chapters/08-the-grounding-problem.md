Chapter 8

# The Answer That Was Wrong

Sam had been at the company for four years when the leadership team asked for a chatbot. It was supposed to answer questions about the company's financial metrics. An executive could ask how many active users did we have last month and the chatbot would return the number. It was a simple use case. The data was in the warehouse. The numbers were accurate. What could go wrong.

The team built it in two weeks. They used a large language model. They gave it access to the database. They wrote a prompt that told the model to query the database and return the answer. They tested it on a few questions. It worked.

Then they deployed it to production. The executives loved it. They used it every day. They asked it about revenue, about churn, about customer acquisition cost. The chatbot answered every question in seconds.

Two weeks later, Sam got an email from the CFO. The email was short. It said the chatbot had reported the wrong revenue number in a board meeting. The number was off by twelve percent. The CFO was not happy.

Sam pulled up the logs. They found the conversation. The CFO had asked a simple question. What was the revenue last month. The chatbot had responded with a number. The number was wrong.

Sam looked at the execution trace. The chatbot had queried the database. It had retrieved the correct number. But then it had generated a response that contained a different number. The model had hallucinated the number. It had taken the correct number from the database and then changed it in the response.

This is the hallucination problem. The model is not lying. It is not trying to deceive. It is doing what it was trained to do. It is generating text that looks plausible. The problem is that it does not have a mechanism for verifying that the text it generates is actually correct.

Sam knew that this was a fundamental limitation of language models. You cannot solve it by fine tuning. You cannot solve it by writing a better prompt. You have to solve it by building a system around the model that verifies the output.

They built a grounding gate. The gate sits between the model and the user. It intercepts the model's response. It parses the response. It extracts every numerical claim. It checks whether each claim is supported by the execution trace. If a claim is not supported, the gate rejects the response. It tells the model to try again, or it tells the user that the system cannot answer the question with confidence.

The gate is not a language model. It is a deterministic system. It compares numbers. It checks references. It does not rely on probability. It relies on logic.

Sam wrote the gate in Python. It took a few days. The logic was simple. If the response contains a number, check whether that number appears in the execution trace. If it does not, reject the response.

The gate caught the CFO's question. The model had generated a number that was not in the execution trace. The gate rejected the response and told the model to try again. The second time, the model generated the correct number.

The chatbot was deployed with the gate. The executives used it for another six months. There were no more complaints. The numbers were accurate.

But Sam knew that this was only the first step. Grounding is necessary, but it is not sufficient. The model can still hallucinate in other ways. It can make claims that are not numerical. It can reference documents that do not exist. It can cite sources that are fabricated.

The solution is to extend the gate to cover more types of claims. Every claim should be traceable to a source. Every source should be verifiable. If a claim cannot be traced, it should be rejected.

This is the future of AI engineering. It is not about building bigger models. It is about building systems that constrain the model. The model is the engine. The gate is the steering wheel. Without the gate, the engine will drive you off a cliff.

---

**The Lesson**

Language models are probabilistic. They generate text that looks plausible. They do not know what is true. The platform engineer builds the grounding gate that verifies the output. They do not trust the model. They trust the system that surrounds the model.
