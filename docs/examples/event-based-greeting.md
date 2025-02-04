

# Event-based Greeting Example

This example demonstrates a workflow that waits for a `greetingcloudevent` event. When the event is received, a state will be triggered using the data provided by the event. 

The `generate-greeting` workflow generates the `greetingcloudevent` that the `eventbased-greeting` workflow is waiting for.

## Event Listener Workflow YAML 

```yaml
id: eventbased-greeting
functions:
- id: greeter
  image: direktiv/greeting:v1
  type: reusable
start:
  type: event
  state: greeter
  event:
    type: greetingcloudevent
description: "A simple action that greets you" 
states:
- id: greeter
  type: action
  action: 
    function: greeter
    input: jq(.greetingcloudevent)
  transform: 'jq({ "greeting": .return.greeting })'
```

## GenerateGreeting Workflow YAML
```yaml
id: generate-greeting
description: "Generate greeting event" 
states:
- id: gen
  type: generateEvent
  event:
    type: greetingcloudevent
    source: Direktiv
    data:
      name: "Trent"
```
