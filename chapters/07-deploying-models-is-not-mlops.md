Chapter 7

# The Model That Stopped Working

Sam had been at the company for three years when the data science team shipped their first model. It was a recommendation engine. It suggested products to users based on their browsing history. The team had been working on it for six months. They had trained it on two years of data. They had evaluated it against a holdout set. They had deployed it to Kubernetes.

The deployment was clean. It had nine FastAPI services. It had a load balancer. It had a p95 latency of one hundred and twenty milliseconds. The data science team was proud of it. They threw a party.

Three months later, the model stopped working. The recommendations were bad. Users were complaining. The data science team could not figure out why. The model was the same. The code was the same. The infrastructure was the same.

Sam was called in to help. They started by looking at the model's predictions. They compared them to the predictions from the day it was deployed. They were different. The model was predicting different things than it used to.

They looked at the input data. It was different too. The distribution of user behavior had changed. Users were browsing different products. They were clicking on different categories. They were spending more time on mobile than on desktop. The model had been trained on desktop behavior. It was now serving mobile users.

This is called data drift. The model is not broken. The world has changed. The model is still doing what it was trained to do, but the training data no longer represents the real world.

The data science team did not know this was happening. They had no monitoring. They had no alerting. They had no way to detect that the model was drifting. They found out when users complained.

Sam knew that this was a common problem. It happens to every machine learning team eventually. The solution is not to retrain the model every day. The solution is to build a monitoring system that detects drift and alerts the team when it happens.

They started by tracking the input data. Every prediction request was logged. The logs contained the features that were sent to the model. They computed the statistical distribution of those features over time. They compared the current distribution to the distribution from the training data. If the difference exceeded a threshold, they alerted the team.

They also tracked the output data. Every prediction was logged. They computed the distribution of the predictions. If the model started predicting different things than it did during training, that was a signal that something had changed.

They also tracked performance metrics. The team had a labeled holdout set. They ran the model against that set every day. They computed accuracy, precision, and recall. If any of those metrics degraded, they alerted the team.

The monitoring system caught three degradations in the first six months. In each case, the model was still returning predictions, but the predictions were getting worse. Without monitoring, the team would not have known until a user complained.

But monitoring is only half the problem. The other half is retraining. Once you know the model is drifting, you need to retrain it on fresh data. This is where data engineering and machine learning intersect. The model is only as good as the data it was trained on. If the data pipeline is broken, the model will be trained on garbage.

Sam looked at the training pipeline. It was a mess. It ran on a single GCP instance. It took five days to train the model. By the time the model was trained, the data was already stale.

They rewrote the pipeline. They parallelized the training across multiple instances. They used a smaller subset of data for the initial evaluation. They reduced the training time from five days to fourteen hours. That meant the team could retrain weekly.

They also added a model registry. Every training run was logged. Every model was versioned. Every deployment was tied to a specific version. If a model degraded, the team could roll back to the previous version in minutes.

The platform stabilized. The model was not perfect, but it was alive. It was monitored. It was retrained. It was versioned.

The platform engineer understands this. They know that deploying a model is not MLOps. MLOps is the entire lifecycle. It is the monitoring, the retraining, the versioning, the rollback. It is the system that keeps the model alive.

---

**The Lesson**

Deploying a model is the easy part. Keeping it alive is the hard part. A model is not a static artifact. It is a living system that degrades over time. The platform engineer builds the monitoring, the retraining pipeline, and the registry that keeps the model healthy. The data scientist who only knows how to train a model is helpless when it drifts.
