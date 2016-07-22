Kafka Connect FTP
=================

[![Build Status](https://travis-ci.org/Eneco/kafka-connect-ftp.svg?branch=master)](https://travis-ci.org/Eneco/kafka-connect-ftp)

Monitors files on an FTP server and feeds changes into Kafka.

Remote directories of interest are to be provided. On a specified interval, the list of files in the directories is refreshed. Files are downloaded when they were not known before, or when their timestamp or size are changed. Only files with a timestamp younger than the specified maximum age are considered. Hashes of the files are maintained and used to check for content changes. Changed files are then fed into Kafka, either as a whole (update) or only the appended part (tail), depending on the configuration.

Data Types
----------

Each Kafka record represents a file, and has the following types.

-   The format of the keys is configurable through `ftp.keystyle=string|struct`. It can be a `string` with the file name, or a structure with `filename: string` and `offset: long`. The offset is always `0` for files that are updated as a whole, and hence only relevant for tailed files.
-   The values of the records contain the body of the file as `bytes`.

Setup
-----

### Properties

In addition to the general configuration for Kafka connectors (e.g. name, connector.class, etc.) the following options are available.

| name                 | data type | required | default | description                                   |
|:---------------------|:----------|:---------|:--------|:----------------------------------------------|
| `ftp.address`        | string    | yes      | -       | host\[:port\] of the ftp server               |
| `ftp.user`           | string    | yes      | -       | username                                      |
| `ftp.password`       | string    | yes      | -       | password                                      |
| `ftp.refresh`        | string    | yes      | -       | iso8601 duration the server is polled         |
| `ftp.file.maxage`    | string    | yes      | -       | iso8601 duration how old files can be         |
| `ftp.keystyle`       | string    | yes      | -       | `string` or `struct`, see above               |
| `ftp.monitor.tail`   | list      | no       | -       | comma separated list of path:destinationtopic |
| `ftp.monitor.update` | list      | no       | -       | comma separated list of path:destinationtopic |

An example file is [here](./example.properties).

### Tailing Versus Update as a Whole

The following rules are used.

-   *Tailed* files are *only* allowed to grow. Bytes that have been appended to it since a last inspection are yielded. Preceding bytes are not allowed to change;
-   *Updated* files can grow, shrink and change anywhere. The entire contents are yielded.

Usage
-----

Build.

    mvn clean package

Put jar into `CLASSPATH`.

    export CLASSPATH=`realpath ./target/kafka-connect-ftp-0.1-jar-with-dependencies.jar` 

With `$CONFLUENT_HOME` pointing to the root of your Confluent Platform installation, start.

    $CONFLUENT_HOME/bin/connect-standalone $CONFLUENT_HOME/etc/schema-registry/connect-avro-standalone.properties your.specific.properties
