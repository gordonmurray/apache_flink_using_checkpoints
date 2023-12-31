# Apache Flink using Checkpoints

I tried out [Checkpointing](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/) in some Flink Jobs to see if it would help with Job restarts

Some details in a blog post here https://gordonmurray.com/aws/2023/10/25/using-checkpoints-in-apache-flink-jobs.html

```docker-compose.yml

version: '3.1'

services:

  jobmanager:
    image: flink:1.17.0
    restart: always
    ports:
      - "8081:8081" # Flink UI port
    command: jobmanager
    volumes:
      - ./jars/flink-sql-connector-mysql-cdc-2.4.1.jar:/opt/flink/lib/flink-sql-connector-mysql-cdc-2.4.1.jar
      - ./jars/flink-connector-jdbc-3.1.0-1.17.jar:/opt/flink/lib/flink-connector-jdbc-3.1.0-1.17.jar
      - ./jars/flink-s3-fs-hadoop-1.17.1.jar:/opt/flink/plugins/s3-fs-hadoop/flink-s3-fs-hadoop-1.17.1.jar
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        execution.checkpointing.mode: AT_LEAST_ONCE
        execution.checkpointing.interval: 60min
        execution.checkpointing.timeout: 10min
        state.backend: rocksdb
        state.backend.incremental: true
        state.checkpoints.dir: s3://example-s3-bucket/checkpoints/
        s3.access.key: {{ s3_key }}
        s3.secret.key: {{ s3_secret }}
        s3.fs.impl: org.apache.hadoop.fs.s3a.S3AFileSystem
        s3.endpoint: s3.amazonaws.com

  taskmanager:
    image: flink:1.17.0
    restart: always
    depends_on:
      - jobmanager
    command: taskmanager
    volumes:
      - ./jars/flink-sql-connector-mysql-cdc-2.4.1.jar:/opt/flink/lib/flink-sql-connector-mysql-cdc-2.4.1.jar
      - ./jars/flink-connector-jdbc-3.1.0-1.17.jar:/opt/flink/lib/flink-connector-jdbc-3.1.0-1.17.jar
      - ./jars/flink-s3-fs-hadoop-1.17.1.jar:/opt/flink/plugins/s3-fs-hadoop/flink-s3-fs-hadoop-1.17.1.jar
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        s3.access.key: {{ s3_key }}
        s3.secret.key: {{ s3_secret }}
        s3.fs.impl: org.apache.hadoop.fs.s3a.S3AFileSystem
        s3.endpoint: s3.amazonaws.com
    deploy:
      replicas:2

```
