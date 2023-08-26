# Microservice test strategy
* Unit, Integration, Component, and Contract tests should be written by developers
* E2E tests should be written by Automation test engineers
* Developers should support Automation test engineers with tools for mocking and test data setup

<img src="https://github.com/khdevnet/testing/blob/main/docs/test-pyramid.png" width="200">

## Unit tests
* unit tests is optional, write to cover complex logic

## Integration tests (test-only integration)
* Write optional one time tests that runs manually to check that integration works

#### What to cover
* Repositories
* HTTP Clients
* Tests should test database migrations, before merge to main branch

#### When to run
* Integration tests runs manually during development and removes after

## Component tests
#### Out-of-process component tests
In a microservices architecture, a component can be a single microservice itself. 
* Use LightBDD testing framework to create human-readable scenarios
* Developers should mock external services HTTP clients using WireMock .NET
* Use local database hosted in docker
* Run tests using Microsoft Test server [integration testing framework](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-7.0).
* Use in-memory mock for message bus. 
#### What to cover
* Cover manual test scenarios with tests
   
#### In-process component tests
* Developers should mock everything inside the controller
* Developers should cover authorization attributes and claims
* Developers should cover model validation and binding
#### What to cover
* Test request validations and Authentication attributes

#### When to run
* should run in the PR gate pipeline

## Contract tests
* Developers should cover internal microservices http clients [pact](https://github.com/pact-foundation/pact-net)
* Developers should cover events contracts, [pact messaging](https://github.com/pact-foundation/pact-net/tree/master/samples/OrdersApi) can be used 


#### When to run
* should run in an independent pipeline, all microservices should check to verify compatibility

## E2E tests (integrations+configurations testing)
Test that the application started and run
* Test database health check
* Test External system HttpClient (call health checks) 
* Run a simple smoke test for the main flow in the microservice

## Resources
[microservice-testing](https://martinfowler.com/articles/microservice-testing/)
