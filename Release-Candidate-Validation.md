- [Validate Binary Distribution](#validate-the-binary-distribution)
  * [Download Binary Distributions](#download-the-binary-distributions)
  * [Validate Pub/Sub and Java Functions](#validate-pubsub-and-java-functions)
  * [Validate Connectors](#validate-connectors)
  * [Validate Stateful Functions](#validate-stateful-functions)
- [Validate RPM and DEB packages](#validate-rpm-and-deb-packages)
  * [Prepare](#prepare)
  * [Validate RPM](#validate-rpm)
  * [Validate DEB](#validate-deb)

Here are some manual instructions for reviewing and validating a release candidate.
These steps can be automated. Contributions are welcome!

### Validate the binary distribution

#### Download the binary distributions

Since Pulsar 2.1, we are shipping two binary distributions. one is server component, containing all the binaries for running Pulsar Service, the other one is a built-in connectors distribution, containing all the built-in connectors.

Unzip the server distribution `apache-pulsar-<release>-bin.tar.gz`. The unzipped binary should be in a directory called `apache-pulsar-<release>`. All the operations below happen within that directory.

```shell
$ cd apache-pulsar-<release>
```

In the apache pulsar directory, unzip the io package and copy the connectors as `connectors` in the pulsar distribution.

```shell
$ tar xf /path/to/apache-pulsar-io-connectors-<release>-bin.tar.gz
// you will find a directory named `apache-pulsar-io-connectors-<release>`
// move the connectors
$ mv apache-pulsar-io-connectors-<release>/connectors connectors
$ ls connectors
pulsar-io-aerospike-<release>.nar
pulsar-io-cassandra-<release>.nar 
pulsar-io-kafka-<release>.nar     
pulsar-io-kinesis-<release>.nar   
pulsar-io-rabbitmq-<release>.nar  
pulsar-io-twitter-<release>.nar
```

#### Validate Pub/Sub and Java Functions

1. Open one terminal to start a standalone cluster.

```shell
$ bin/pulsar standalone
```

when you started the standalone cluster, there are a few things to example:

a) the standalone cluster should be able to locate all the connectors. you should be able to see following logging information.

```shell
Found connector ConnectorDefinition(name=kinesis, description=Kinesis sink connector, sourceClass=null, sinkClass=org.apache.pulsar.io.kinesis.KinesisSink) from /Users/sijie/tmp/apache-pulsar-2.1.0-incubating/./connectors/pulsar-io-kinesis-2.1.0-incubating.nar
...
Found connector ConnectorDefinition(name=cassandra, description=Writes data into Cassandra, sourceClass=null, sinkClass=org.apache.pulsar.io.cassandra.CassandraStringSink) from /Users/sijie/tmp/apache-pulsar-2.1.0-incubating/./connectors/pulsar-io-cassandra-2.1.0-incubating.nar
...
Found connector ConnectorDefinition(name=aerospike, description=Aerospike database sink, sourceClass=null, sinkClass=org.apache.pulsar.io.aerospike.AerospikeStringSink) from /Users/sijie/tmp/apache-pulsar-2.1.0-incubating/./connectors/pulsar-io-aerospike-2.1.0-incubating.nar
```

b) (since pulsar 2.1) the standalone should start bookkeeper table service as well.

example output:

```shell
12:12:26.099 [main] INFO  org.apache.pulsar.zookeeper.LocalBookkeeperEnsemble - 'default' namespace for table service : namespace_name: "default"
default_stream_conf {
  key_type: HASH
  min_num_ranges: 24
  initial_num_ranges: 24
  split_policy {
    fixed_range_policy {
      num_ranges: 2
    }
  }
  rolling_policy {
    size_policy {
      max_segment_size: 134217728
    }
  }
  retention_policy {
    time_policy {
      retention_minutes: -1
    }
  }
}
```

c) functions worker is started correctly.

example output:

```shell
12:12:31.647 [pulsar-external-listener-70-1] INFO  org.apache.pulsar.functions.worker.MembershipManager - Worker c-standalone-fw-localhost-6750:localhost:6750 became the leader.
```

b) sanity check before moving to next steps:

```shell
// check pulsar binary port is listened correctly
$ telnet localhost 6650

// check function cluster
$ curl -s http://localhost:8080/admin/v2/worker/cluster
// example output
[{"workerId":"c-standalone-fw-localhost-6750","workerHostname":"localhost","port":6750}]

// check brokers 
$ curl -s http://localhost:8080/admin/v2/namespaces/public
// example outoupt
["public/default","public/functions"]

// check connectors
$ curl -s http://localhost:8080/admin/v2/functions/connectors
// example output
[{"name":"aerospike","description":"Aerospike database sink","sinkClass":"org.apache.pulsar.io.aerospike.AerospikeStringSink"},{"name":"cassandra","description":"Writes data into Cassandra","sinkClass":"org.apache.pulsar.io.cassandra.CassandraStringSink"},{"name":"kafka","description":"Kafka source and sink connector","sourceClass":"org.apache.pulsar.io.kafka.KafkaStringSource","sinkClass":"org.apache.pulsar.io.kafka.KafkaStringSink"},{"name":"kinesis","description":"Kinesis sink connector","sinkClass":"org.apache.pulsar.io.kinesis.KinesisSink"},{"name":"rabbitmq","description":"RabbitMQ source connector","sourceClass":"org.apache.pulsar.io.rabbitmq.RabbitMQSource"},{"name":"twitter","description":"Ingest data from Twitter firehose","sourceClass":"org.apache.pulsar.io.twitter.TwitterFireHose"}]

// check table services
$ telnet localhost 4181
```

2. Open another terminal to submit a Java Exclamation function.

```shell
$ bin/pulsar-admin functions create --functionConfigFile examples/example-function-config.yaml --jar examples/api-examples.jar
```

You should see `Created Successfully`.

3. At the same terminal as step 2, retrieve the function config:

```shell
$ bin/pulsar-admin functions get --tenant test --namespace test-namespace --name example
```

example output:

```shell
{
  "tenant": "test",
  "namespace": "test-namespace",
  "name": "example",
  "className": "org.apache.pulsar.functions.api.examples.ExclamationFunction",
  "userConfig": "{\"PublishTopic\":\"test_result\"}",
  "autoAck": true,
  "parallelism": 1,
  "source": {
    "topicsToSerDeClassName": {
      "test_src": ""
    },
    "typeClassName": "java.lang.String"
  },
  "sink": {
    "topic": "test_result",
    "typeClassName": "java.lang.String"
  },
  "resources": {}
}
```

4. At the same terminal as step 3, retrieve the function status:

```shell
$ bin/pulsar-admin functions getstatus --tenant test --namespace test-namespace --name example
```

example output:

```shell
{
  "functionStatusList": [
    {
      "running": true,
      "instanceId": "0",
      "metrics": {
        "metrics": {
          "__total_processed__": {},
          "__total_successfully_processed__": {},
          "__total_system_exceptions__": {},
          "__total_user_exceptions__": {},
          "__total_serialization_exceptions__": {},
          "__avg_latency_ms__": {}
        }
      },
      "workerId": "c-standalone-fw-localhost-6750"
    }
  ]
}
```

5. At the same terminal as step 4, subscribe the output topic `test_result`.

```shell
$ bin/pulsar-client consume -s test-sub -n 0 test_result
```

6. Open a new terminal to produce messages into the input topic `test_src`.

```shell
$ bin/pulsar-client produce -m "test-messages-`date`" -n 10 test_src
```

7. At the terminal of step 5, you will see the messages produced by the Exclamation function.

example output:

```shell
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
----- got message -----
test-messages-Thu Jul 19 11:59:15 PDT 2018!
```

#### Validate Connectors

> Make sure you have docker available at your laptop. If you don't have docker installed, you can skip this section.

1. Setup a cassandra cluster.

```shell
$ docker run -d --rm  --name=cassandra -p 9042:9042 cassandra
```

Make sure the cassandra cluster is up running.

```shell
// run docker ps to find the docker process for cassandra
$ docker ps
```

```shell
// check if the cassandra is running as expected
$ docker logs cassandra
```

```shell
// check the cluster status
$ docker exec cassandra nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  103.67 KiB  256          100.0%            af0e4b2f-84e0-4f0b-bb14-bd5f9070ff26  rack1
```

```shell

```

2. Create keyspace and table

Run cqlsh:
```shell
$ docker exec -ti cassandra cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.11.2 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh>
```

In the cqlsh, create keyspace `pulsar_test_keyspace` and table `pulsar_test_table`.

```shell
cqlsh> CREATE KEYSPACE pulsar_test_keyspace WITH replication = {'class':'SimpleStrategy', 'replication_factor':1}; 
cqlsh> USE pulsar_test_keyspace;
cqlsh:pulsar_test_keyspace> CREATE TABLE pulsar_test_table (key text PRIMARY KEY, col text);

```

3. Prepare a cassandra sink yaml file and put it under examples directory as `cassandra-sink.yml`.

```shell
$ cat examples/cassandra-sink.yml
configs:
    roots: "localhost:9042"
    keyspace: "pulsar_test_keyspace"
    columnFamily: "pulsar_test_table"
    keyname: "key"
    columnName: "col"
```

4. Submit a cassandra sink

```shell
$ bin/pulsar-admin sink create --tenant public --namespace default --name cassandra-test-sink --sink-type cassandra --sinkConfigFile examples/cassandra-sink.yml --inputs test_cassandra
"Created successfully"
```

```shell
// get the sink info
$ bin/pulsar-admin functions get --tenant public --namespace default --name cassandra-test-sink
{
  "tenant": "public",
  "namespace": "default",
  "name": "cassandra-test-sink",
  "className": "org.apache.pulsar.functions.api.utils.IdentityFunction",
  "autoAck": true,
  "parallelism": 1,
  "source": {
    "topicsToSerDeClassName": {
      "test_cassandra": ""
    }
  },
  "sink": {
    "configs": "{\"roots\":\"cassandra\",\"keyspace\":\"pulsar_test_keyspace\",\"columnFamily\":\"pulsar_test_table\",\"keyname\":\"key\",\"columnName\":\"col\"}",
    "builtin": "cassandra"
  },
  "resources": {}
}
```

```shell
// get the running status
$ bin/pulsar-admin functions getstatus --tenant public --namespace default --name cassandra-test-sink
{
  "functionStatusList": [
    {
      "running": true,
      "instanceId": "0",
      "metrics": {
        "metrics": {
          "__total_processed__": {},
          "__total_successfully_processed__": {},
          "__total_system_exceptions__": {},
          "__total_user_exceptions__": {},
          "__total_serialization_exceptions__": {},
          "__avg_latency_ms__": {}
        }
      },
      "workerId": "c-standalone-fw-localhost-6750"
    }
  ]
}
```

5. Produce messages to the source topic
```shell
$ for i in {0..10}; do bin/pulsar-client produce -m "key-$i" -n 1 test_cassandra; done
```

6. Check the sink status. It should show 11 messages are processed.

```shell
$ bin/pulsar-admin functions getstatus --tenant public --namespace default --name cassandra-test-sink
{
  "functionStatusList": [
    {
      "running": true,
      "numProcessed": "11",
      "numSuccessfullyProcessed": "11",
      "lastInvocationTime": "1532031040117",
      "instanceId": "0",
      "metrics": {
        "metrics": {
          "__total_processed__": {
            "count": 5.0,
            "sum": 5.0,
            "max": 5.0
          },
          "__total_successfully_processed__": {
            "count": 5.0,
            "sum": 5.0,
            "max": 5.0
          },
          "__total_system_exceptions__": {},
          "__total_user_exceptions__": {},
          "__total_serialization_exceptions__": {},
          "__avg_latency_ms__": {}
        }
      },
      "workerId": "c-standalone-fw-localhost-6750"
    }
  ]
}
```

7. Check results in cassandra

```shell
$ docker exec -ti cassandra cqlsh localhost
Connected to Test Cluster at localhost:9042.
[cqlsh 5.0.1 | Cassandra 3.11.2 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> use pulsar_test_keyspace;
cqlsh:pulsar_test_keyspace> select * from pulsar_test_table;

 key    | col
--------+--------
  key-5 |  key-5
  key-0 |  key-0
  key-9 |  key-9
  key-2 |  key-2
  key-1 |  key-1
  key-3 |  key-3
  key-6 |  key-6
  key-7 |  key-7
  key-4 |  key-4
  key-8 |  key-8
 key-10 | key-10

(11 rows)
```
8. Delete the Sink

```shell
$ bin/pulsar-admin sink delete --tenant public --namespace default --name cassandra-test-sink
"Deleted successfully"
```

#### Validate Stateful Functions

Since Pulsar 2.1, Pulsar enables bookkeeper table service for stateful pulsar functions (as a developer preview).

Here are the instructions to validate counter functions:

1. Create a wordcount function

```shell
$ bin/pulsar-admin functions create --functionConfigFile examples/example-function-config.yaml --jar examples/api-examples.jar --name word_count --className org.apache.pulsar.functions.api.examples.WordCountFunction --inputs test_wordcount_src --output test_wordcount_dest
"Created successfully"
```

2. Get function info and status

```shell
$ bin/pulsar-admin functions get --tenant test --namespace test-namespace --name word_count
{
  "tenant": "test",
  "namespace": "test-namespace",
  "name": "word_count",
  "className": "org.apache.pulsar.functions.api.examples.WordCountFunction",
  "userConfig": "{\"PublishTopic\":\"test_result\"}",
  "autoAck": true,
  "parallelism": 1,
  "source": {
    "topicsToSerDeClassName": {
      "test_wordcount_src": ""
    },
    "typeClassName": "java.lang.String"
  },
  "sink": {
    "topic": "test_wordcount_dest",
    "typeClassName": "java.lang.Void"
  },
  "resources": {}
}
```

```shell
$ bin/pulsar-admin functions getstatus --tenant test --namespace test-namespace --name word_count
{
  "functionStatusList": [
    {
      "running": true,
      "instanceId": "0",
      "metrics": {
        "metrics": {
          "__total_processed__": {},
          "__total_successfully_processed__": {},
          "__total_system_exceptions__": {},
          "__total_user_exceptions__": {},
          "__total_serialization_exceptions__": {},
          "__avg_latency_ms__": {}
        }
      },
      "workerId": "c-standalone-fw-localhost-6750"
    }
  ]
}
```

3. Query the state table for the function: watching on a key called "hello"

```shell
$ bin/pulsar-admin functions querystate --tenant test --namespace test-namespace --name word_count -u bk://localhost:4181 -k hello -w
key 'hello' doesn't exist.
key 'hello' doesn't exist.
key 'hello' doesn't exist
```

4. Produce the messages to source topic `test_wordcount_src`.

Produce 10 messages "hello" to topic `test_wordcount_src`. The value of "hello" should be updated to 10.

```shell
$ bin/pulsar-client produce -m "hello" -n 10 test_wordcount_src
```

Checkout the result in the terminal open at step 3.

```shell
value = 10
```

Now produce another 10 messages "hello". You will see the result is updated to 20.

```shell
$ bin/pulsar-client produce -m "hello" -n 10 test_wordcount_src
```

The result in the terminal open at step 3 is updated to `20`.

```shell
value = 10
value = 20
```

### Validate RPM and DEB packages

#### Prepare

Download the RPM and DEP packages to a directory. Let's name this directory as `pulsar_validation`. We will mount this directory to a docker container so that we can validate RPM packages.

#### Validate RPM

1. Start a centos docker to install RPM

```shell
$ docker run -it --rm -v `pwd`:/pulsar centos:7
```

in centos container, go to directory `/pulsar`

```shell
[root@e6c29e2b70a9 /]# cd pulsar/
[root@e6c29e2b70a9 pulsar]# ls
```

2. Install RPM in the centos container

```shell
# rpm -ivh apache-pulsar-client-devel-2.1.0-1_incubating.x86_64.rpm apache-pulsar-client-2.1.0-1_incubating.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:apache-pulsar-client-2.1.0-1_incu################################# [ 50%]
   2:apache-pulsar-client-devel-2.1.0-################################# [100%]
```

3. Build the go client to validate the RPM package.

a) install go

```shell
# yum install -y go git
```

b) get pulsar go client

```shell
# go get -u github.com/apache/pulsar/pulsar-client-go/pulsar
```

c) build an example

```shell
# cd ~/go/src/github.com/apache/pulsar/pulsar-client-go/examples/producer/
# go build .
# ls
producer  producer.go
# ldconfig
# ./producer
```

#### Validate DEB

1. Start an ubuntu docker to install DEB

```shell
$ docker run -it --rm -v `pwd`:/pulsar ubuntu:16.04
```

in ubuntu container, go to directory `/pulsar`

```shell
[root@e6c29e2b70a9 /]# cd pulsar/
[root@e6c29e2b70a9 pulsar]# ls
```

2. Install DEB in the centos container

```shell
# apt install ./apache-pulsar-client.deb
# apt install ./apache-pulsar-client-dev.deb
```

3. Build the go client to validate the DEB package.

a) install go

```shell
# apt-get update
# apt-get -y upgrade
# apt-get install -y git curl gcc
# curl -O https://storage.googleapis.com/golang/go1.9.1.linux-amd64.tar.gz
# tar -xvf go1.9.1.linux-amd64.tar.gz
# mv go /usr/local
# export GOROOT=/usr/local/go
# export PATH=$PATH:$GOROOT/bin
```

b) get pulsar go client

```shell
# go get -u github.com/apache/pulsar/pulsar-client-go/pulsar
```

c) build an example

```shell
# cd ~/go/src/github.com/apache/pulsar/pulsar-client-go/examples/producer/
# go build .
# ls
producer  producer.go
# ./producer
```