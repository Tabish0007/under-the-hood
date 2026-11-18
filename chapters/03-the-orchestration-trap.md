Chapter 3

The Orchestration Trap

If the data lake is the storage layer and Delta Lake is the transactional layer, the orchestrator is the conductor. It decides when a pipeline runs, what order the tasks execute in, and what happens when something fails. For most companies, that conductor is Apache Airflow. But Airflow is not a data processor. It is a scheduler. And the moment you forget that distinction, you walk straight into the orchestration trap.

The trap is seductive because it is so easy to build. You open your editor, import the DAG class, and write a Python function. You put your transformation logic inside that function. You call it a task. It works in development. It works in staging. So you ship it to production. Six months later, you have a DAG with forty tasks, each one doing heavy lifting in Python. The scheduler is struggling to keep up with the metadata overhead. The workers are constantly out of memory. And you cannot figure out why the whole thing is so slow.

The answer is simple: you are running your compute on the orchestrator. You are competing for CPU and memory with the scheduler that is supposed to be managing the pipeline. You have turned a control plane into a data plane.

The right way to use Airflow is to treat it as a dispatcher. A task should be a thin wrapper that calls out to something else. It should trigger a Spark job on EMR. It should run a dbt command against your warehouse. It should execute a container that does the heavy lifting. The DAG file should be small and declarative. It should describe the dependencies, not the implementation. The logic belongs in a separate repository, tested and versioned independently.

But even if you use Airflow correctly, you still have a problem. You have hundreds of DAGs running every day. Each one has dependencies, retries, and SLAs. When a pipeline fails at 3 AM, how do you know which DAG caused it. When a task takes twice as long as usual, how do you know before the SLA is missed. You cannot answer those questions by looking at the Airflow UI. The Airflow UI shows you the state of a single task at a single point in time. It does not show you the health of your entire data platform.

This is where observability comes in. You need to build a system that monitors the orchestrator itself. It should track the success rate of every DAG. It should detect anomalies in run duration. It should alert you when a critical pipeline is stuck or when a data quality check fails.

At BSH Hausgeräte, we built a Data Quality Notification System to solve this problem. It monitored over forty Airflow DAGs and thirty SAP Datasphere task chains. It kept a historical record of every run, every failure, and every retry. It computed anomaly detection scores using historical run times to flag when a pipeline was behaving out of character. The system reduced pipeline retry rates by sixty six percent and caught issues before they impacted downstream consumers.

Building that system taught me something fundamental. The orchestrator is not the platform. The orchestrator is just one component. The platform is the combination of the orchestrator, the storage, the transaction log, the compute engine, and the observability layer. If any one of those pieces fails silently, the whole system is compromised.

There is a deeper lesson here about distributed systems. An orchestrator is essentially a state machine. It tracks the state of every task: queued, running, success, failed, retrying. The state is stored in a metadata database, usually Postgres. Every task transition is a write to that database. When you have thousands of tasks running concurrently, the database becomes a bottleneck. This is why Airflow struggles at scale. It is not the scheduler that is slow. It is the database that is slow.

To fix this, you need to reduce the number of state transitions. You do that by making tasks smaller. If a task does one thing and exits, it transitions twice: from queued to running, and from running to success. If a task does ten things, it takes ten times as long, and it holds the worker for ten times as long. The database load does not change. But the wall clock time does. And wall clock time is what matters when you have an SLA.

You also need to separate the control plane from the data plane. The control plane is Airflow. The data plane is Spark, or dbt, or whatever compute engine you use. The control plane should never touch the data. It should only tell the data plane what to do. When the data plane finishes, it reports back to the control plane. This separation is what allows you to scale the data plane independently of the control plane. If you need more compute, you add more workers. If you need more scheduling capacity, you scale the Airflow cluster. They are decoupled.

This is the architecture that every mature data platform eventually converges on. It is not because it is trendy. It is because it is the only way to handle the complexity of a modern data stack.

In the next chapter we will go deeper into the transaction log. We will look at what happens when a protocol mutation corrupts a table and how to fix it at the kernel level.
