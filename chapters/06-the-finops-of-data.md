Chapter 6

# The Bill That Doubled Overnight

Maya had been running the platform for two years when the CFO sent an email. It was short. It said the data infrastructure budget had been exceeded for the third month in a row. It said the platform was now the second largest line item in the technology budget, after salaries. It said the company needed to cut the cost by half.

Maya printed the email and pinned it to her wall. She did not panic. She had seen this coming. The platform had grown organically. Every team was adding new pipelines. Every pipeline was adding new clusters. Nobody was tracking the cost.

She pulled up the AWS billing dashboard. The bill was three thousand euros a month. That was not huge for a company of that size, but it was high for the workload the platform was running. Most of the cost was coming from a few sources.

The first source was EC2 instances. The Spark clusters were running twenty four hours a day. They were not being used at night. They were not being used on weekends. They were just sitting there, burning money.

The second source was S3 storage. The platform was storing every version of every table forever. It was storing logs and temporary files that nobody needed. It was storing data in the wrong storage class. Data that was accessed once a month was stored in the standard tier instead of the infrequent access tier.

The third source was data transfer. The Spark clusters were in one availability zone and the S3 buckets were in another. Every read from S3 crossed the boundary and incurred a cost. This was the silent killer. It was small per byte, but multiplied by petabytes of data, it added up.

Maya had three options. She could ask the teams to reduce their usage. That was not realistic. The teams were building products. They were not going to stop. She could renegotiate the AWS contract. That was above her pay grade. Or she could optimize the platform. That was her job.

She started with the clusters. She configured autoscaling so that the clusters scaled down when they were not being used. She set a minimum of zero instances and a maximum based on the workload. She moved the Spark jobs to spot instances where possible. Spot instances are cheaper than on demand instances, but they can be terminated by AWS at any time. She wrote retry logic to handle the interruptions.

The result was a thirty percent reduction in EC2 costs.

She moved on to storage. She configured lifecycle policies to move old data to infrequent access storage. She set retention policies to delete temporary files after seven days. She enabled versioning on critical tables, but she deleted old versions after ninety days.

The result was a twenty percent reduction in S3 costs.

She moved on to data transfer. She moved the Spark clusters to the same availability zone as the S3 buckets. She enabled S3 Transfer Acceleration for cross region reads. She configured the Spark jobs to read data in the same region where the clusters were running.

The result was a fifteen percent reduction in data transfer costs.

The total reduction was fifty eight percent. The bill went from three thousand euros to twelve hundred and seventy euros. The CFO sent a thank you email. The platform was no longer a problem.

But Maya knew that this was a temporary fix. The platform would grow again. New teams would onboard. New pipelines would be added. The costs would creep back up. She needed a system that would catch the problem before it became a crisis.

She built a cost monitoring dashboard. It tracked the cost of every resource. It broke down the cost by team and by pipeline. It sent alerts when the cost of a resource exceeded a threshold. It showed the trend over time.

She also wrote a set of cost guidelines. They were short. They said that every pipeline should have an owner. Every pipeline should have a budget. Every pipeline should be reviewed quarterly.

The platform was now a financial asset, not just a technical one. The platform engineer understands this. They know that the cloud is not free. They know that the bill is a first class engineering metric.

---

**The Lesson**

Cost is not a constraint that you impose on engineering. Cost is a design parameter. When you design a pipeline, you should think about how much it will cost to run. If you can achieve the same result with less money, you should do it. The platform engineer knows how to profile the cost, how to optimize the resources, and how to build the tooling that keeps the cost under control.
