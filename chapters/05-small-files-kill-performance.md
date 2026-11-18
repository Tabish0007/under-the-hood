Chapter 5

Small Files Kill Performance

If you ask a data engineer why their query is slow, they will usually blame the compute. They will say the cluster is too small, or the Spark configuration is wrong, or the query is not optimized. Those can be valid reasons. But in a lakehouse architecture, the most common culprit is the physical layout of the data on disk.

A lakehouse is built on object storage. Object storage is excellent for storing massive amounts of data cheaply. It is terrible for reading small amounts of data fast. Every time an engine reads a file from S3, it has to make a network request. That request has latency. It does not matter if the file is one megabyte or one gigabyte. The overhead is the same.

If your table is made up of ten thousand files that are each one megabyte, a query has to make ten thousand network requests to read the data. That is slow. If your table is made up of ten files that are each one gigabyte, the query only makes ten requests. That is fast.

This is the small file problem. It is the most common cause of poor read performance in a data lake.

Small files are created by streaming jobs, frequent batch jobs, or aggressive partitioning. If you partition a table by hour, you will create a new file every hour. If you partition by minute, you will create a new file every minute. After a year, you will have hundreds of thousands of files. The table will become unusable.

The solution is to compact the files. In Delta Lake, the command is called OPTIMIZE. It reads the small files and rewrites them into larger files. This is a background maintenance task. It does not change the logical content of the table. It just changes the physical layout.

But OPTIMIZE is not free. It costs compute. And if you run it too often, you are wasting money. If you run it too rarely, your queries get slow. You need to know how often to run it and how to measure the impact.

This is where observability matters. You need to know the average file size of your tables. You need to know how many small files exist. You need to know whether the file count is growing over time.

This is exactly why I built the delta-doctor CLI. It scans a table, counts the files, computes the average size, and flags any files that are under the recommended threshold. If you run it against a table that has thousands of tiny files, it will tell you to run OPTIMIZE.

The lesson here is that the physical layout of the data is just as important as the logical layout. You can write perfect Spark jobs and perfect SQL queries, but if the underlying files are fragmented, the platform will be slow. The engineer who understands the physical layout is the one who can fix it.

In the next chapter we will look at the other side of the cost equation. We will look at how to reduce your cloud bill by understanding how storage and compute interact.
