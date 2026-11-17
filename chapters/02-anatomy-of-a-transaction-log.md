Chapter 2

The Anatomy of a Transaction Log

A Delta table is not a database. It is a directory on object storage. That directory contains two things: the data files in Parquet format, and a _delta_log directory containing the transaction log. The transaction log is the source of truth. If you understand the transaction log, you understand the table.

When you write to a Delta table, the engine does not modify the existing Parquet files. Instead, it writes new Parquet files and appends a new JSON commit to the _delta_log. Each commit is a single JSON file, named with a zero padded version number. The first commit is 00000000000000000000.json. The second is 00000000000000000001.json. And so on.

Each commit contains one or more actions. An action is a single instruction that describes a change to the state of the table. There are several types of actions, but the most important ones are add, remove, metaData, and protocol.

The add action tells the engine that a new Parquet file is part of the table. It contains the path to the file, the size of the file, the partition values, and some statistics. The remove action marks a file as logically deleted. It does not delete the file from storage. It just tells the engine to ignore it when constructing the current state of the table.

The metaData action describes the schema of the table and its configuration. It includes things like the table name, the description, the partition columns, and the table properties. When you call set_table_properties in Python or Scala, you are appending a metaData action to the log.

The protocol action defines the rules for reading and writing the table. It contains the minReaderVersion, the minWriterVersion, and the readerFeatures and writerFeatures arrays. A reader that does not support the features listed in the protocol will refuse to read the table. That is why a protocol mutation can break a query engine even if the data itself is fine.

To see what this actually looks like, open a terminal and create a simple table using the delta-rs Python bindings. Then look at the first commit.

```python
import pyarrow as pa
from deltalake import write_deltalake
write_deltalake("/tmp/example_table", pa.table({"id": [1, 2, 3]}))



Now read the JSON file at /tmp/example_table/_delta_log/00000000000000000000.json. It will contain a sequence of newline-delimited JSON objects. The first object is the commit info, which records who made the change and when.
json

{"commitInfo":{"timestamp":1718473923000,"operation":"WRITE","operationParameters":{"mode":"ErrorIfExists"},"engineInfo":"deltalake/1.6.5"}}

The second object is the protocol. In this case, the table was created at reader version 1 and writer version 2, which is the default for a table that does not use advanced features.
json

{"protocol":{"minReaderVersion":1,"minWriterVersion":2}}

The third object is the metadata. It defines the schema and the configuration for the table.
json

{"metaData":{"id":"f0e7b3c2-4b2d-4b4e-9a5f-8e8f7e2a1b3c","format":{"provider":"parquet"},"schemaString":"{\"type\":\"struct\",\"fields\":[{\"name\":\"id\",\"type\":\"long\",\"nullable\":true,\"metadata\":{}}]}","partitionColumns":[],"configuration":{}}}

The final object is the add action. It points to the Parquet file that was written to disk.
json

{"add":{"path":"part-00000-2c3d4e5f-6a7b-8c9d-0e1f-2a3b4c5d6e7f-c000.snappy.parquet","size":478,"partitionValues":{},"modificationTime":1718473923000,"dataChange":true}}

This is the entire state of the table at version zero. Four JSON lines. No hidden database, no proprietary binary format. Just text files on disk. If you understand these four actions, you understand the entire mechanism of the table.

To construct the current state of the table, the engine reads the transaction log from the most recent commit backwards. It applies the actions in reverse order, building a list of active files. It stops when it reaches a checkpoint. A checkpoint is a Parquet file that contains the entire state of the table at a specific version. Checkpoints exist because reading thousands of JSON files is slow. By writing a checkpoint every ten commits, the engine can skip the early commits and start from the checkpoint.

This architecture is what allows Delta Lake to support concurrent reads and writes. When a writer commits, it appends a new JSON file. When a reader queries, it reads the latest JSON files. The reader never needs to lock the table. The writer never blocks the reader. The cost of this consistency is that you need to have a mechanism for optimistic concurrency control. If two writers try to commit at the same time, one will succeed and the other will fail with a conflict error. The failed writer has to retry.

When you are debugging a Delta table, the first thing you should do is open the _delta_log directory and look at the latest commit. You can read the JSON file directly. You will see the exact actions that were applied. If a reader is failing, look at the protocol action. If a query is returning stale data, look at the add and remove actions. The log tells you everything.

In the next chapter we will look at what happens when the orchestration layer that schedules these commits fails. We will build a system for detecting broken pipelines before they impact downstream consumers.
