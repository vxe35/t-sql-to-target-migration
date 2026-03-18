# Migration Progress Tracking

## Overview

Track the progress of each migration step here. Update checkboxes as tasks are completed.

**Last Updated**: 2026-03-19 00:13

## Progress Summary


### Step 1: Inventory & Assessment

**Status**: 🔴 Not Started
**Branch**: `migration/step-1-inventory-and-assessment`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] List every stored procedure, view, SSIS package, and SQL Agent job
- [ ] Tag each as: read-only query, ETL transform, aggregation, upsert/merge, or streaming
- [ ] Identify external dependencies (linked servers, CLR procs, full-text search)
- [ ] Estimate row volumes and execution frequency per object
- [ ] Choose target runner: Dataflow (managed) vs Flink / Spark (self-hosted)

**Notes**:
- 

---

### Step 2: Set Up Apache Beam Java Project

**Status**: 🔴 Not Started
**Branch**: `migration/step-2-set-up-apache-beam-java-project`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Add Apache Beam Java SDK BOM (beam-sdks-java-io-jdbc, beam-runners-direct-java)
- [ ] Add runner dependency: beam-runners-google-cloud-dataflow-java OR beam-runners-flink-1.17
- [ ] Configure PipelineOptions interface (project, region, tempLocation for Dataflow)
- [ ] Set up JDBC connection pool (HikariCP) for SQL Server source reads
- [ ] Add SQL Server JDBC driver (mssql-jdbc) to dependencies
- [ ] Establish local Direct Runner smoke test with a trivial Read→Write pipeline

**Notes**:
- 

---

### Step 3: Model T-SQL Schemas as Java Row / POJO

**Status**: 🔴 Not Started
**Branch**: `migration/step-3-model-t-sql-schemas-as-java-row-/-pojo`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Create @SerializableRecord / POJO for each key table or result-set shape
- [ ] Implement Beam Coder registration for each POJO (SerializableCoder or AvroCoder)
- [ ] Map SQL Server types: NVARCHAR→String, DECIMAL→BigDecimal, DATETIME2→Instant, BIT→Boolean
- [ ] Handle nullable columns with Optional<T> or explicit null-check in DoFn

**Notes**:
- 

---

### Step 4: Port SELECT / Filter / Projection Logic

**Status**: 🔴 Not Started
**Branch**: `migration/step-4-port-select-/-filter-/-projection-logic`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Implement JdbcIO.read() with parametrized query for each source table
- [ ] Replace WHERE predicates with Filter.by(SerializableFunction<Row,Boolean>)
- [ ] Replace SELECT column list with MapElements.via(row -> new OutputType(row.col1, row.col2))
- [ ] Replace computed columns (CASE, CONVERT, COALESCE) with DoFn @ProcessElement logic
- [ ] Validate output row counts against SQL Server baseline with JUnit assertions

**Notes**:
- 

---

### Step 5: Port GROUP BY Aggregations

**Status**: 🔴 Not Started
**Branch**: `migration/step-5-port-group-by-aggregations`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Replace GROUP BY col + SUM/COUNT/AVG with GroupByKey + Combine.perKey()
- [ ] Implement custom CombineFn for weighted averages or multi-field aggregations
- [ ] Replace HAVING clause with Filter.by() applied after Combine
- [ ] Replace window functions (ROW_NUMBER, RANK OVER) with stateful DoFn + state API
- [ ] For streaming: wrap GroupByKey in Window.into(FixedWindows.of(Duration.standardMinutes(5)))

**Notes**:
- 

---

### Step 6: Port JOIN Logic

**Status**: 🔴 Not Started
**Branch**: `migration/step-6-port-join-logic`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Large-to-large join: use CoGroupByKey on matching key PCollections
- [ ] Large fact + small dimension: use side-input (View.asMap) broadcast join
- [ ] LEFT OUTER JOIN: handle missing tags in CoGroupByKey output DoFn
- [ ] CROSS APPLY / OUTER APPLY: implement as ParDo emitting multiple elements per input
- [ ] Verify join cardinality matches SQL Server output in integration tests

**Notes**:
- 

---

### Step 7: Port MERGE / Upsert and Write Logic

**Status**: 🔴 Not Started
**Branch**: `migration/step-7-port-merge-/-upsert-and-write-logic`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Simple INSERT: use JdbcIO.write() or BigQueryIO.write()
- [ ] MERGE (upsert): use JdbcIO.write() with custom PreparedStatementSetter using ON CONFLICT / MERGE SQL in the sink
- [ ] Multi-destination writes: use tagged outputs (TupleTag) to fan out to multiple sinks
- [ ] Dead-letter queue: route failed records to a separate TextIO / BigQuery error table

**Notes**:
- 

---

### Step 8: Port Streaming / CDC Sources (if applicable)

**Status**: 🔴 Not Started
**Branch**: `migration/step-8-port-streaming-/-cdc-sources-(if-applicable)`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Enable SQL Server CDC on source tables
- [ ] Use Debezium SQL Server connector to publish CDC events to Kafka or Pub/Sub
- [ ] Replace polling batch job with KafkaIO.read() or PubSubIO.read() Beam source
- [ ] Define watermark strategy and late-data handling policy
- [ ] Set up Dataflow Streaming Engine or Flink checkpoint config

**Notes**:
- 

---

### Step 9: Testing & Validation

**Status**: 🔴 Not Started
**Branch**: `migration/step-9-testing-and-validation`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Unit test each DoFn with TestPipeline and PAssert
- [ ] Integration test end-to-end pipeline with Direct Runner against a test DB
- [ ] Row-count and checksum comparison between SQL Server output and Beam output
- [ ] Performance test on representative data volume; tune parallelism
- [ ] Add monitoring: Dataflow metrics, custom Beam metrics (Metrics.counter)

**Notes**:
- 

---

### Step 10: Deployment & Cut-over

**Status**: 🔴 Not Started
**Branch**: `migration/step-10-deployment-and-cut-over`
**Assigned To**: _Unassigned_
**Start Date**: _Not started_
**Completion Date**: _Not completed_

**Tasks**:
- [ ] Package pipeline as fat JAR or container image
- [ ] Set up CI/CD pipeline (GitHub Actions / Azure DevOps) to build and deploy
- [ ] Run Beam pipeline in shadow mode (parallel with SQL Server) to verify parity
- [ ] Schedule via Cloud Scheduler / Airflow / ADF trigger
- [ ] Disable SQL Server jobs after successful parallel-run period
- [ ] Monitor Dataflow/Flink dashboard: throughput, lag, error rates

**Notes**:
- 

---

## Status Legend

- 🔴 Not Started
- 🟡 In Progress
- 🟢 Completed
- 🔵 In Review
- ⚪ Blocked

## How to Update

1. Change status emoji
2. Check off completed tasks
3. Add notes about progress, blockers, or decisions
4. Update dates as work progresses
5. Commit changes to main branch

## Team Members

| Name | Role | Steps Assigned |
|------|------|----------------|
| TBD  | TBD  | TBD            |

---

**Project Started**: 2026-03-19
