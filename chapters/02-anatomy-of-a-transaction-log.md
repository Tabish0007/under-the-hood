Chapter 2

# The Commit That Disappeared

Maya had been at the company for two years when she started to trust the platform. The migration was done. The pipelines were stable. The on-call rotation was quiet. She had time to work on the backlog.

Then the ticket came in. It was from the data science team. Their training pipeline was failing. The job was reading from a Delta table, and it was throwing an error that made no sense. The error said the table did not exist at the version they were trying to read.

Maya opened the table in Spark. It read fine. She ran a simple query. It returned the right data. She checked the history. The table had forty-two versions. The latest version was forty-one. Everything looked normal.

She asked the data scientist to share the code. They were using a feature called time travel. They were reading the table at version thirty-eight. They were pinning the version to ensure reproducibility. That was a good practice. But it was failing.

Maya tried to read the table at version thirty-eight. It failed. She tried version thirty-seven. It worked. She tried version thirty-nine. It worked. Version thirty-eight was missing.

She opened the _delta_log directory on S3. She listed the files. She saw commits from 00000000000000000000.json to 00000000000000000041.json. Version thirty-eight was there. The file existed. So why was the engine saying it did not exist?

She opened the file. It was empty. Zero bytes. The commit had been created but never written to. This was a partial write. The writer had crashed after creating the file but before writing the content.

This is a known problem in distributed systems. It is called a torn write. The file system guarantees that the file is created before the write begins, but it does not guarantee that the write completes. If the writer crashes in the middle, you are left with an empty file. The transaction log is now corrupted.

The good news is that Delta Lake is designed to handle this. The engine does not trust the file system. It checks the file size before reading. If the file is empty, it treats the commit as failed and skips it. That is why Spark could still read the table. It was skipping version thirty-eight and reading version thirty-nine instead.

But the data scientist was using a different engine. They were using a Python library that did not implement the same safety checks. It saw the file, tried to read it, and crashed.

Maya had two options. She could tell the data scientist to upgrade their library. That would patch the symptom. Or she could delete the empty commit from the log. That would fix the table.

She chose the second option. She backed up the log directory. She deleted the empty file. She refreshed the table. The table was now readable at version thirty-eight. The data scientist was unblocked.

But Maya was not satisfied. She wanted to understand why the writer had crashed. She looked at the logs from the ingestion job. The job had been running on a spot instance. The instance had been terminated by the cloud provider in the middle of a write. That was the root cause.

The lesson here is that the transaction log is not just a list of commits. It is a contract between the writer and the reader. The writer promises to write a complete commit. The reader promises to read only complete commits. If the writer breaks the promise, the reader has to protect itself. That is why Delta Lake checks the file size. That is why it skips empty files.

But not every engine implements those checks. Some engines are more trusting. They assume that the file system is reliable and that the writer always completes its work. When that assumption is wrong, the engine crashes.

The platform engineer understands this. They know that the transaction log is the source of truth. They know that the log can be corrupted. They know how to read the log, how to identify a torn write, and how to repair it.

The data engineer who does not understand the log is helpless. They see an error message and they open a ticket. They wait for someone else to fix it.

This is the difference between using the platform and owning it.

---

**The Lesson**

The transaction log is a contract. It defines the state of the table at every version. If the log is corrupted, the table is corrupted. The platform engineer knows how to read the log, identify corruption, and repair it. The data engineer does not. That is the gap this book is about.
