Chapter 5

# The Query That Took Forever

Maya had been at the company for two years when the platform started to slow down. It was subtle at first. Queries that used to take ten seconds took fifteen. Dashboards that used to refresh in a minute took ninety seconds. Nothing was broken, exactly. It was just slower.

Then the complaints started. The analytics team said the morning reports were late. The data science team said the training pipeline was taking twice as long as it used to. The on-call engineer said the cluster was constantly at capacity.

Maya pulled up the monitoring dashboard. The cluster was at ninety percent utilization. The jobs were running. The queries were finishing. But everything was taking longer.

She picked one of the slow queries and looked at the query plan. It was a simple aggregation. It was reading a table with a filter on a date column. The filter should have eliminated most of the data. But the query was reading the entire table.

She checked the table. It was partitioned by date. The partition pruning should have worked. The query should have read only the files in the relevant partition. But it was reading every file in every partition.

She opened the _delta_log directory. She looked at the file count. The table had one hundred and twenty thousand files. That was the problem.

The ingestion job was writing a new file every minute. Each file was small. The table had been running for a year. That was five hundred and twenty five thousand minutes. But the table was partitioned by day, so each partition had one thousand four hundred and forty files. That was still too many.

The engine had to open every file in the partition to read the data. Each file was a separate network request to S3. Each request had latency. The query was spending most of its time waiting for the network, not processing data.

This is the small file problem. It is the most common cause of poor read performance in a data lake. The solution is to compact the files. In Delta Lake, the command is called OPTIMIZE.

Maya ran OPTIMIZE on the table. It read all the small files and rewrote them into larger files. The table went from one hundred and twenty thousand files to one thousand two hundred files. The query time dropped from ninety seconds to four seconds.

But Maya knew that this was a temporary fix. The ingestion job would keep writing small files. In a week, the table would be back to one hundred and twenty thousand files. She needed a better solution.

She had two options. She could change the ingestion job to write larger files. That would require buffering the data in memory before writing. It was possible, but it would add complexity to the job. The second option was to run OPTIMIZE on a schedule. That would keep the file count under control without changing the ingestion job.

She chose the second option. She added a new DAG to Airflow. It ran every night at midnight. It called OPTIMIZE on every table in the lakehouse. She added a monitoring step that checked the file count after the optimize. If the count was above a threshold, it would alert the team.

The platform stabilized. The queries were fast again. The clusters were underutilized. The complaints stopped.

But Maya was not satisfied. She wanted to know how often she needed to run OPTIMIZE. If she ran it too often, she was wasting compute. If she ran it too rarely, the tables would slow down again. She needed data.

She built a small tool. It was a Python script that connected to the Delta table, listed the files, and computed the average file size. It also counted the number of files under a threshold. The threshold was thirty two megabytes. Files under that size were considered small.

She ran the tool against every table in the lakehouse. She found that some tables were fine. They had large files and low file counts. Other tables were in bad shape. They had thousands of small files.

She added the tool to the monitoring dashboard. It ran every morning and produced a report. The report showed the file count and the average file size for every table. If a table was in bad shape, it was flagged.

This is the platform engineer mindset. You do not just fix the problem. You build the tool that detects the problem before it happens again. You instrument the system so that you can see what is going on.

The data engineer who does not understand the small file problem will keep running queries that take forever. They will keep adding workers to the cluster. They will never fix the root cause. The platform engineer builds the tool that makes the problem visible.

---

**The Lesson**

The physical layout of the data is just as important as the logical layout. You can write perfect SQL and perfect Spark jobs, but if the underlying files are fragmented, the platform will be slow. The platform engineer knows how to measure the file count, how to compact the files, and how to build the tooling that keeps the problem from coming back.
