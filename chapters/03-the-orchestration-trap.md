Chapter 3

# The Job That Took Twenty Four Hours

Maya was in a meeting when her phone buzzed. It was a Slack message from the on-call engineer. The nightly ETL job had not finished. It had started at midnight. It was now nine in the morning. The SLA was six hours. They were three hours late.

She excused herself from the meeting and opened her laptop. She pulled up the Airflow UI. The DAG was called nightly_ingest. It had one task. The task was called run_spark_job.

She looked at the task. It had been running for nine hours. It had not failed. It had not timed out. It was just stuck. The Spark job was running on a cluster that she had provisioned the week before. The cluster had ten workers. Each worker had eight cores and thirty two gigabytes of memory.

She SSH'd into the cluster. She ran top. The workers were idle. The CPU usage was near zero. The memory usage was near zero. The Spark job was not doing any work. It was waiting for something.

She checked the Spark UI. The job had thirty thousand tasks. Each task was processing one megabyte of data. That was the problem. The data was partitioned into tiny files. Each file was being processed as a separate task. The overhead of scheduling thirty thousand tasks was larger than the time it took to process the data.

The root cause was upstream. The ingestion job was writing files every minute. Each file was small. After a year, the table had five hundred thousand files. Spark could not handle that.

She had three options. She could increase the cluster size. That would add more workers, but the overhead would still be there. She could rewrite the ingestion job to batch the writes. That would take a week. Or she could compact the files using OPTIMIZE. That would take an hour.

She chose OPTIMIZE. She ran it on the table. It read the five hundred thousand small files and rewrote them into fifty large files. The job took an hour. When it was done, she reran the ETL job. It finished in two hours.

This is the small file problem. It is the most common cause of poor performance in a data lake. The solution is not to throw more compute at it. The solution is to fix the physical layout of the data.

But that was not the real lesson of the day. The real lesson was about the orchestrator.

Maya had built the DAG with a single task. That task ran a Python script. The script called Spark submit. The script also did some pre processing, validation, and cleanup. All of that logic was running on the Airflow worker.

Airflow is a scheduler. It is not a compute engine. It is designed to coordinate tasks, not to execute them. When you put heavy logic inside an Airflow task, you are competing with the scheduler for resources. You are also making the DAG harder to test, harder to debug, and harder to reuse.

Maya rewrote the DAG. She split it into three tasks. The first task triggered a Spark job on EMR. The second task ran a dbt model. The third task ran a validation script. Each task was a thin wrapper. The logic was moved to separate repositories. The DAG was now declarative. It described the dependencies, not the implementation.

The new DAG finished in forty five minutes. The old DAG had taken twenty four hours.

The lesson here is about the separation of concerns. The orchestrator is the control plane. It decides what to run and when. The compute engine is the data plane. It actually runs the work. If you mix the two, you get the worst of both worlds. The control plane becomes slow and fragile. The data plane becomes underutilized.

The platform engineer understands this separation. They know that Airflow is not a data processor. They know that Spark is not a scheduler. They know how to use each tool for what it is designed to do.

The data engineer who does not understand this will keep building DAGs with heavy Python logic. They will keep wondering why the scheduler is slow. They will keep adding more workers to the cluster. And they will never fix the root cause.

---

**The Lesson**

The orchestrator is not the platform. It is one component. If you treat it as a compute engine, it will become the bottleneck. The platform engineer separates the control plane from the data plane. They let Airflow schedule and Spark compute. That separation is what allows the platform to scale.
