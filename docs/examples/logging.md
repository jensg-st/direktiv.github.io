

# Logging

Though Direktiv provides instance logs viewable on the instance's information page, it won't keep them forever. Sending logs to a third-party logging service can be useful for taking control of your data. Here's how we can do it.

## The 'log' Parameter

Every state in a workflow definition supports an optional `log` parameter. If this parameter is defined, Direktiv will evaluate it as a `jq` query against the instance data before executing the state's logic, and send the results to the instance logs.

## CloudEvents From Logs

Workflows can be configured to generate CloudEvents on their namespace anytime the `log` parameter produces data. Look for a field called "Log To Event" on the workflow definition (YAML) UI page. If this field is set to anything other than an empty string CloudEvents will be generated with an additional extension `logger` set to the value saved here.

Using this it's easy to capture instance logs and do whatever you want with them. Create another workflow that is triggered by matching CloudEvents, then send them to your own Logstash server, for example.

```yaml
id: helloworld
states:
- id: hello
  type: noop
  transform: 'jq({ result: "Hello World!" })'
  log: '"Hello, logger!"'
```

This workflow, configured to Log To Event to `mylog`, will produce a CloudEvent like the following:

```json
{
    "specversion" : "1.0",
    "type" : "direktiv.instanceLog",
    "source" : "4e455e04-dfcc-4d7d-90b3-37e00988335d",
    "id" : "a9a07797-fe2a-490e-ba61-61b86e61d6c0",
    "time" : "2018-04-05T17:31:00Z",
    "logger" : "mylog",
    "datacontenttype" : "text/json",
    "data" : "Hello, logger!"
}
```

## Google Cloud Platform Stackdriver Example

This workflow listens for the logger `gcpLogger` and sends it to gcp:

```yaml
id: gcp-logger
functions:
- id: send-log
  image: direktiv/gcplog:v1
  type: reusable
start:
  type: event
  event:
    type: direktiv.instanceLog
    filters:
      logger: gcpLogger
states:
- id: log
  type: action
  action:
    function: send-log
    secrets: [GCP_SERVICEACCOUNTKEY]
    input: 
      serviceAccountKey: jq(.secrets.GCP_SERVICEACCOUNTKEY)
      "project-id": "direktiv"
      "log-name": "direktiv-log"
      message: jq(."direktiv.instanceLog")
```

## AWS Cloudwatch Example

This workflow listens for the logger `awsLogger` and sends it to aws:

```yaml
id: aws-logger
functions:
- id: send-log
  image: direktiv/awslog:v1
  type: reusable
start:
  type: event
  event:
    type: direktiv.instanceLog
    filters:
      logger: awsLogger
states:
- id: log
  type: action
  action:
    function: send-log
    secrets: [AWS_KEY, AWS_SECRET]
    input:
      key: jq(.secrets.AWS_KEY)
      secret: jq(.secrets.AWS_SECRET)
      region: "us-east-2"
      "log-group": "direktiv"
      "log-stream": "direktiv"
      message: jq(."direktiv.instanceLog")
```

## Azure Log Analytics Example

This workflow listens for the logger `azureLogger` and sends it to azure:

```yaml
id: azure-logger
functions:
- id: send-log
  image: direktiv/azlog:v1
  type: reusable
start:
  type: event
  event:
    type: direktiv.instanceLog
    filters:
      logger: azure-logger
states:
- id: log
  type: action
  action:
    function: send-log
    secrets: [AZURE_WORKSPACE_ID, AZURE_WORKSPACE_KEY]
    input:
      "workspace-id": jq(.secrets.AZURE_WORKSPACE_ID)
      key: jq(.secrets.AZURE_WORKSPACE_KEY)
      type: "direktiv-log"
      message: jq(."direktiv.instanceLog")
```

## Other Providers

These three cloud examples serve to demonstrate how this all works, but they're not the only options. Following this pattern you can log any way you like.
