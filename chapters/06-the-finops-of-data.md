Chapter 6

The FinOps of Data

Most data engineers do not think about money. They think about latency, throughput, and reliability. They care about whether the pipeline finishes before the SLA. They care about whether the query returns the right result. They do not care about the bill.

This is a mistake. In a modern data platform, the bill is a first class engineering metric. It is just as important as latency. It is just as important as reliability. If you build a platform that is fast and reliable but costs three times as much as it should, you have built a liability.

Cloud providers make it easy to spend money. That is by design. They charge you for every byte you store, every byte you transfer, and every second of compute you use. They do not send you a warning when your costs spike. They just send you a bill at the end of the month. By the time you see the bill, it is too late.

The cost of a data platform comes from three places. The first is storage. This is usually the cheapest component. S3 costs pennies per gigabyte. But storage costs can explode if you are not careful. If you keep every version of every table forever, if you never expire old data, if you never clean up temporary files, your storage bill will grow linearly forever.

The second is compute. This is where the real money is spent. Spark clusters, EMR instances, Redshift warehouses. They are expensive. And they are easy to over provision. You spin up a large cluster to run a single job, you forget to shut it down, and you pay for idle capacity for a week.

The third is data transfer. This is the silent killer. Every time data moves between regions, between availability zones, or between storage and compute, you pay. If your Spark cluster is in one availability zone and your S3 bucket is in another, you are paying for every byte that crosses the boundary.

At BSH Hausgeräte, we had a platform that was reliable and functional but expensive. The monthly bill was three thousand euros. That is not huge for a company of that size, but it was high for the workload we were running. We were paying for idle compute, for inefficient queries, and for storage that we did not need.

We started by profiling the platform. We looked at every DAG and measured how long it took, how much compute it used, and what it actually produced. We found that most of the cost was coming from a small number of jobs. They were running on oversized clusters, they were shuffling more data than necessary, and they were writing files that were too small.

We made three changes. First, we reduced the cluster size and configured autoscaling so that the cluster scaled down when it was idle. Second, we rewrote the most expensive queries to reduce the amount of data they shuffled. Third, we ran OPTIMIZE on the tables to compact the small files.

The result was a fifty eight percent reduction in cost. The bill went from three thousand euros to twelve hundred and seventy euros per month. We did not sacrifice performance. We actually improved it, because the smaller clusters forced us to write more efficient queries.

This is the lesson of FinOps. Cost is not a constraint that you impose on engineering. Cost is a design parameter. When you design a pipeline, you should think about how much it will cost to run. If you can achieve the same result with less money, you should do it.

The engineer who understands FinOps is the one who can build a platform that the company can afford to run. The engineer who does not is the one who builds a platform that gets shut down because the bill is too high.

In the next chapter we will switch gears and look at the AI layer. We will look at how to deploy machine learning models in a way that is reliable, observable, and cost effective.
