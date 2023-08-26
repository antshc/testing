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
#### Out of process component tests
In a microservices architecture, a component can be a single microservice itself. 
* Use LightBDD testing framework to create human readable scenarios
* Developers should mock external services HTTP clients using WireMock .NET
* Use local database hosted in docker

   

#### Test Doubles & test run framework
* Run tests using Microsoft Test server [integration testing framework](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-7.0).
* Use in-memory Entity framework for database
* Use Moq for other dependencies

#### What to cover
##### Test request validations and Authentication attributes
* Developers should mock everything inside the controller
* Developers should cover authorization attributes and claims
* Developers should cover model validation and binding

##### Test business logic with in-memory mocks
* Write tests for each controller that tests business logic. 

#### When to run
* should run in PR gate pipeline

## E2E tests (integrations+configurations testing)
Test that the application started and run
* Test database health check
* Test External system HttpClient (call health checks) 
* Run a simple smoke test for the main flow in the microservice

## Resources
[microservice-testing](https://martinfowler.com/articles/microservice-testing/)
