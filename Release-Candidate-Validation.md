Here are some instructions for reviewing and validating a release candidate.

### Validate the binary distribution

Unzip the binary distribution. The unzipped binary should be in a directory called `apache-pulsar-<release>`. All the operations below happen within that directory.

1. Open one terminal to start a standalone cluster.

```shell
$ bin/pulsar standalone
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