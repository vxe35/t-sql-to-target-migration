# Migration Plan: T-SQL to Apache Beam (Java SDK)

## Overview

This repository contains the migration plan and structure for converting a T-SQL codebase to Apache Beam (Java SDK).

**Migration Strategy**: Incremental, step-by-step approach
**Target Timeline**: To be determined based on codebase complexity
**Branches**: Each step has its own branch for isolated development

## Migration Steps


### Step 1: Inventory & Assessment

**Description**: Catalogue all T-SQL objects and classify them by Beam transformation pattern.

**Branch**: `migration/step-1-inventory-and-assessment`

**Tasks**:
- [ ] List every stored procedure, view, SSIS package, and SQL Agent job
- [ ] Tag each as: read-only query, ETL transform, aggregation, upsert/merge, or streaming
- [ ] Identify external dependencies (linked servers, CLR procs, full-text search)
- [ ] Estimate row volumes and execution frequency per object
- [ ] Choose target runner: Dataflow (managed) vs Flink / Spark (self-hosted)

**Estimated Duration**: TBD


### Step 2: Set Up Apache Beam Java Project

**Description**: Bootstrap Maven/Gradle project with Beam SDK and chosen runner dependencies.

**Branch**: `migration/step-2-set-up-apache-beam-java-project`

**Tasks**:
- [ ] Add Apache Beam Java SDK BOM (beam-sdks-java-io-jdbc, beam-runners-direct-java)
- [ ] Add runner dependency: beam-runners-google-cloud-dataflow-java OR beam-runners-flink-1.17
- [ ] Configure PipelineOptions interface (project, region, tempLocation for Dataflow)
- [ ] Set up JDBC connection pool (HikariCP) for SQL Server source reads
- [ ] Add SQL Server JDBC driver (mssql-jdbc) to dependencies
- [ ] Establish local Direct Runner smoke test with a trivial Read→Write pipeline

**Estimated Duration**: TBD


### Step 3: Model T-SQL Schemas as Java Row / POJO

**Description**: Create Java data classes that mirror T-SQL table and result-set schemas.

**Branch**: `migration/step-3-model-t-sql-schemas-as-java-row-/-pojo`

**Tasks**:
- [ ] Create @SerializableRecord / POJO for each key table or result-set shape
- [ ] Implement Beam Coder registration for each POJO (SerializableCoder or AvroCoder)
- [ ] Map SQL Server types: NVARCHAR→String, DECIMAL→BigDecimal, DATETIME2→Instant, BIT→Boolean
- [ ] Handle nullable columns with Optional<T> or explicit null-check in DoFn

**Estimated Duration**: TBD


### Step 4: Port SELECT / Filter / Projection Logic

**Description**: Replace T-SQL SELECT and WHERE clauses with JdbcIO sources and Filter / MapElements transforms.

**Branch**: `migration/step-4-port-select-/-filter-/-projection-logic`

**Tasks**:
- [ ] Implement JdbcIO.read() with parametrized query for each source table
- [ ] Replace WHERE predicates with Filter.by(SerializableFunction<Row,Boolean>)
- [ ] Replace SELECT column list with MapElements.via(row -> new OutputType(row.col1, row.col2))
- [ ] Replace computed columns (CASE, CONVERT, COALESCE) with DoFn @ProcessElement logic
- [ ] Validate output row counts against SQL Server baseline with JUnit assertions

**Estimated Duration**: TBD


### Step 5: Port GROUP BY Aggregations

**Description**: Map T-SQL aggregation patterns to Beam Combine and GroupByKey transforms.

**Branch**: `migration/step-5-port-group-by-aggregations`

**Tasks**:
- [ ] Replace GROUP BY col + SUM/COUNT/AVG with GroupByKey + Combine.perKey()
- [ ] Implement custom CombineFn for weighted averages or multi-field aggregations
- [ ] Replace HAVING clause with Filter.by() applied after Combine
- [ ] Replace window functions (ROW_NUMBER, RANK OVER) with stateful DoFn + state API
- [ ] For streaming: wrap GroupByKey in Window.into(FixedWindows.of(Duration.standardMinutes(5)))

**Estimated Duration**: TBD


### Step 6: Port JOIN Logic

**Description**: Replace T-SQL JOINs with Beam CoGroupByKey or broadcast side-input patterns.

**Branch**: `migration/step-6-port-join-logic`

**Tasks**:
- [ ] Large-to-large join: use CoGroupByKey on matching key PCollections
- [ ] Large fact + small dimension: use side-input (View.asMap) broadcast join
- [ ] LEFT OUTER JOIN: handle missing tags in CoGroupByKey output DoFn
- [ ] CROSS APPLY / OUTER APPLY: implement as ParDo emitting multiple elements per input
- [ ] Verify join cardinality matches SQL Server output in integration tests

**Estimated Duration**: TBD


### Step 7: Port MERGE / Upsert and Write Logic

**Description**: Replace T-SQL MERGE, INSERT, UPDATE statements with Beam sink writes.

**Branch**: `migration/step-7-port-merge-/-upsert-and-write-logic`

**Tasks**:
- [ ] Simple INSERT: use JdbcIO.write() or BigQueryIO.write()
- [ ] MERGE (upsert): use JdbcIO.write() with custom PreparedStatementSetter using ON CONFLICT / MERGE SQL in the sink
- [ ] Multi-destination writes: use tagged outputs (TupleTag) to fan out to multiple sinks
- [ ] Dead-letter queue: route failed records to a separate TextIO / BigQuery error table

**Estimated Duration**: TBD


### Step 8: Port Streaming / CDC Sources (if applicable)

**Description**: Replace SQL Server CDC / Service Broker with Beam streaming IO connectors.

**Branch**: `migration/step-8-port-streaming-/-cdc-sources-(if-applicable)`

**Tasks**:
- [ ] Enable SQL Server CDC on source tables
- [ ] Use Debezium SQL Server connector to publish CDC events to Kafka or Pub/Sub
- [ ] Replace polling batch job with KafkaIO.read() or PubSubIO.read() Beam source
- [ ] Define watermark strategy and late-data handling policy
- [ ] Set up Dataflow Streaming Engine or Flink checkpoint config

**Estimated Duration**: TBD


### Step 9: Testing & Validation

**Description**: Validate Beam pipeline output against SQL Server baseline.

**Branch**: `migration/step-9-testing-and-validation`

**Tasks**:
- [ ] Unit test each DoFn with TestPipeline and PAssert
- [ ] Integration test end-to-end pipeline with Direct Runner against a test DB
- [ ] Row-count and checksum comparison between SQL Server output and Beam output
- [ ] Performance test on representative data volume; tune parallelism
- [ ] Add monitoring: Dataflow metrics, custom Beam metrics (Metrics.counter)

**Estimated Duration**: TBD


### Step 10: Deployment & Cut-over

**Description**: Deploy to production runner and retire T-SQL jobs.

**Branch**: `migration/step-10-deployment-and-cut-over`

**Tasks**:
- [ ] Package pipeline as fat JAR or container image
- [ ] Set up CI/CD pipeline (GitHub Actions / Azure DevOps) to build and deploy
- [ ] Run Beam pipeline in shadow mode (parallel with SQL Server) to verify parity
- [ ] Schedule via Cloud Scheduler / Airflow / ADF trigger
- [ ] Disable SQL Server jobs after successful parallel-run period
- [ ] Monitor Dataflow/Flink dashboard: throughput, lag, error rates

**Estimated Duration**: TBD


## How to Use This Repository

1. **Review the migration plan**: Read through all steps carefully
2. **Start with Step 1**: Checkout the `migration/step-1-*` branch
3. **Complete tasks**: Follow the guide in each step's README
4. **Create PRs**: Submit pull requests for review after completing each step
5. **Track progress**: Update the MIGRATION_TRACKING.md file
6. **Move to next step**: Once a step is complete and merged, move to the next

## Branch Structure

- `main`: Main branch with documentation and tracking
- `migration/step-X-*`: Individual branches for each migration step
- `staging`: Integration branch for testing combined changes
- `production`: Final production-ready code

## Progress Tracking

Track your progress in `.github/MIGRATION_TRACKING.md`

## Issues

Each migration step has a corresponding GitHub issue for discussion and tracking.

## Resources

- Official {target_lang} documentation
- Framework migration guides
- Community resources and examples

## Notes

- Test thoroughly at each step
- Maintain backward compatibility where possible
- Document any deviations from the plan
- Keep stakeholders informed of progress

---

**Created**: 2026-03-19
