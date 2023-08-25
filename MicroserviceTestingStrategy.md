# Microservice test strategy
* Unit, Integration, Component, and Contract tests should be written by developers
* E2E tests should be written by Automation test engineers
* Developers should support  Automation test engineers with tools for mocking and test data setup

<img src="https://github.com/khdevnet/testing/blob/main/docs/test-pyramid.png" width="200">

## Unit tests
* Developers should write unit tests for every newly added class
* Developers should write tests for existing classes, covering only newly added logic
* All interactions and collaborations between an object and its dependencies, should be replaced by test doubles (**Solitary unit testing**)
* Using code coverage extension [FineCodeCoverage](https://github.com/FortuneN/FineCodeCoverage) in Visual studio to validate coverage 
* Unit tests should be part of the story
* If unit tests are not possible to write because of business pressure, a tech dept task should be created with a list of classes that don't cover with tests, PM should notified that tests should be written after releases as first priority

#### What to cover
Cover everything that is possible with unit tests
* Classes with business logic
* Controllers
* Attributes
* Model Validators
* Filters
* etc ..

#### When to run
* Run manually by developers locally using [Fine code coverage](https://marketplace.visualstudio.com/items?itemName=FortuneNgwenya.FineCodeCoverage) extension
* Run in PR before the merge, all tests should pass
* Measure code coverage using [Visual studio test task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/vstest?view=azure-devops)
* Setup baseline for test coverage and revision condition each month

## Integration tests (test only integration)
* Developers should write integration tests for every newly added repository class
* Developers should write integration tests for existing classes, covering only newly added logic
* Setup test data using SQL or EF queries
* Each test should set and clean test data
* Tests should support running in parallel

#### What to cover
* Repositories
* External clients (optional)

#### When to run
* Integration tests should run in PR gate before the merge
* It should test migrations when merging to develop environment

#### Database test double
* Make a backup of the latest copy of the database and remove redundant data
* Run database in docker container using tmpfs (in memory storage)
* Before starting an integration test run docker inside test using docker client and stop it after

## Component tests
#### In-process component tests
In a microservices architecture, a component can be a single microservice itself. 
* Developers should mock all external dependencies (Database, External API)
 
#### What to cover
* Cover Responses of the component from API perspective

Note: In process controllers component tests
* Developers should mock everything inside the controller
* Developers should cover authorization attributes and claims
* Developers should cover model validation and binding
* Developers should cover everything that is impossible to cover using Unit Tests

#### Out-of-process component tests
* Developers should mock all external dependencies except databases

#### What to cover
* Developers should cover one successful test case from API to database for each new endpoint

#### When to run
* Should run in PR gate before the merge

## Contract tests
* Developers should cover internal microservices http clients [pact](https://github.com/pact-foundation/pact-net)
* Developers should cover events contracts, [McVerifier](https://github.com/khdevnet/mc-verifier) can be used 

#### When to run
* should run in an independent pipeline when contract tests are updated, all microservices should check to verify compatibility


## E2E tests (integrations+configurations testing)
Test that the application started and run
* Test database health check
* Test External system HttpClient (call health checks) 
Test short smoke
* Test a simple short smoke


## Resources
[microservice-testing](https://martinfowler.com/articles/microservice-testing/)
