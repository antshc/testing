# Problem
* We have 4 teams and plan to grow, it's already hard to control what changes were made by other teams, what can be broken because of this
* When the contract changes, How to you understand if other microservice continue to work as expected?
* NuGet package/submodules don't solve the problem, because you need to manually install/update it to understand if it breaks something
* Integration tests slow, not stable, hard to support

# Contract tests  
![Message-contract-tests drawio](https://user-images.githubusercontent.com/14298158/172701493-2ebf0d2c-0afc-4a57-95d4-c9c7ce72c9d4.png)

* Whenever some consumer couples to the interface of a component to make use of its behavior, a contract is formed between them. This contract consists of expectations of input and output data structures.
* Each consumer of the component forms a different contract based on its requirements. If the component is subject to change over time, it is important that the contracts of each of the consumers continue to be satisfied.
* Contract test suites written by each consuming team are packaged and runnable in the build pipelines for the producing services. In this way, the maintainers of the producing service know the impact of their changes on their consumers.

# Pros & Cons

## Pros:
* Consumer knows only about fields what it required (id, name)
* Verify contract among all microservices
* Don't need to check manually if contract change breaks something
* Runs automatically in pipeline
* Tests can be used as a documentation and describe different scenarios
* Easy to implement

## Cons:
* Require additional time for implementation
* Is in the early stage of development, some verification doesn't work (decimal not verified and can be used integer), can be fixed with regex
* Use a small number of restrictions.

## What could be validated
* Required fields
* Length of the input string (256 characters) in case Microservice DB has a constraint
* Type of the field (string, number)
* Email, phone number format using regex.
* Enum options in the message (Paid, Sold ...) if one of them removed test fails


# PactNet Nuget
## Message contract tests
Messaging pacts work similarly to synchronous request/response pacts in that there is still a consumer and a producer. The consumer defines the messages it expects to receive (including metadata and the message contents) and the producer is verified to ensure that the messages it produces meet those expectations. [Messaging Pacts](https://github.com/pact-foundation/pact-net/blob/master/docs/messaging-pacts.md)


## Api contract tests
The consumer defines the HTTP message payload (URL, headers, request, response ...) contract it expects to receive and the producer is verified to ensure that the HTTP payload it produces meets those expectations.
[Api Pacts](https://github.com/pact-foundation/pact-net)


# Run automatically
* Consumer1 and Consumer2 build pipeline generates schemas and publish them as an artifact
* Producer download messages schemas from pipeline artifact and run tests using them
![Contract tests pipelines drawio](https://user-images.githubusercontent.com/14298158/173375359-35c75304-fc86-4b95-abec-0b88a961c7af.png)


# Resources
* [Pact Learning](https://github.com/DiUS/pact-workshop-dotnet-core-v3/blob/master/LEARNING.md)
* [pact-workshop-dotnet-core-v3](https://github.com/DiUS/pact-workshop-dotnet-core-v3/)
* [martinfowler Contract testing ](https://martinfowler.com/articles/microservice-testing/#testing-contract-introduction)
