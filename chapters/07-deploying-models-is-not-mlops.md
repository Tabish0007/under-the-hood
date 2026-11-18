Chapter 7

Deploying Models is Not MLOps

There is a misconception in the industry that MLOps means putting a model behind a FastAPI endpoint. You train a model, you serialize it, you wrap it in a REST API, and you deploy it to Kubernetes. Congratulations. You have deployed a model. But you have not done MLOps.

Deploying a model is the easy part. The hard part is keeping it alive. A model is not a static artifact. It is a living system that degrades over time. The data it was trained on changes. The distribution of the input data shifts. The users start asking questions that the model was not trained to answer. If you do not monitor the model, it will silently fail.

This is what happened at Ignitus. We had nine FastAPI model services running on Kubernetes. They were fast and reliable. They had a p95 latency of one hundred and twenty milliseconds. On paper, they were perfect.

But latency is not the only metric that matters. A model can have perfect latency and be completely wrong. We needed to know whether the model was producing correct predictions. We needed to know whether the input data was drifting. We needed to know whether the model was degrading.

We built a monitoring system that tracked three things. The first was input drift. We compared the statistical distribution of incoming data to the distribution of the training data. If the drift exceeded a threshold, we alerted the team. The second was output drift. We tracked the distribution of predictions over time. If the model started predicting a different distribution than it did during training, that was a signal that something had changed. The third was performance metrics. We tracked accuracy, precision, and recall against a labeled holdout set. We ran this evaluation daily.

This monitoring system caught three model degradations before they impacted users. In each case, the model was still returning predictions, but the predictions were getting worse. Without monitoring, we would not have known until a user complained.

The lesson here is that MLOps is not about deployment. It is about lifecycle management. It is about knowing when a model is healthy and when it is not. It is about having the processes and the tooling to retrain a model when it degrades and to roll back to a previous version if the new one is worse.

This requires a model registry. A model registry is a catalog of every model that has been trained, every version of every model, and every artifact associated with it. It stores the model weights, the training metrics, the evaluation results, and the lineage of the data that was used to train it. Without a registry, you have no way to reproduce a model. You have no way to roll back to a previous version. You have no way to audit what happened.

We used MLflow as our model registry. Every training run was logged. Every model was versioned. Every deployment was tied to a specific version. If a model degraded, we could roll back to the previous version in minutes.

But a registry is not enough. You also need a way to retrain the model. This is where data engineering and machine learning intersect. The model is only as good as the data it was trained on. If the data pipeline is broken, the model will be trained on garbage. If the data is stale, the model will be stale.

At Ignitus, we built a training pipeline that ran on GCP. It pulled the latest data from the data lake, trained the model, evaluated it against a holdout set, and pushed it to the registry. The entire process took five days. That was too slow. By the time the model was trained, the data was already stale. We rewrote the pipeline to parallelize the training and to use a smaller subset of data for the initial evaluation. The retraining time went from five days to fourteen hours. That meant we could retrain weekly. That meant the model was always close to the current distribution of data.

The lesson here is that MLOps is a pipeline, not a deployment. It is the entire process from data to model to production to monitoring to retraining. If any part of that pipeline is broken, the whole thing fails.

The engineer who understands MLOps is the one who can build that pipeline. The engineer who does not is the one who deploys a model and then wonders why it stopped working six months later.

In the next chapter we will look at the frontier of AI engineering. We will look at how to evaluate large language models and agentic workflows, and why grounding is the most important problem in the field right now.
