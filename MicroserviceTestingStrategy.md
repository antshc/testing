# Microservice test strategy
* Unit, Integration, Component, Contract tests should be written by developes
* E2E tests should be write by Automation test engineers
* Developers should support  Automation test engineer with tools for mocking and test data setup

<img src="https://github.com/khdevnet/testing/blob/main/docs/test-pyramid.png" width="200">

## Unit tests
* Developers should write unit tests for every new added class
* Developers should write tests for existing classes, cover only new added logic
* All interactions and collaborations between an object and its dependencies, should be replaced by test doubles (**Solitary unit testing**)
* Using code coverege tool in Visual studio enterprise all class conditions should be cover
* Unit tests should be part of the story
* if unit tests not possible to write because of business presure, tech dept task should be created with list of classes what doesn't cover with tests, PM should notified that tests should be written after releases as first priority

### What to cover
Cover everithing what possible with unit tests
* Classes with business logic
* Controllers
* Attributes
* Model Validators
* Filters
* etc ..

### When to run
* Run manually by developers locally use [Fine code coverage](https://marketplace.visualstudio.com/items?itemName=FortuneNgwenya.FineCodeCoverage) extension
* Run in PR before merge, all tests should pass
* Measure code coverage using [Visual studio test task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/vstest?view=azure-devops)
* Setup baseline for test coverage and revision condition each month

## Integration tests
* Developers should write integration tests for every new added repository class
* Developers should write integration tests for existing classes, cover only new added logic
* Setup test data using SQL or EF queries
* Each test should setup and cleanup test data
* Tests should support run in paralell

### What to cover
* Repositories
* External clients (optional)

### When to run
* Integration tests should run in PR gate before merge
* It should test migrations when merge to develop env

### Database test double
* Make backup latest copy of database and remove redundant data
* Run database in docker container using tmpfs (in memory storage)
* Before start an integration test run docker inside test using docker client and stop it after

## Component tests
### In process component tests
* Developers should mock everything inside controller

### What to cover
* Developers should cover authorization attributes and claims
* Developers should cover everyhing what impossible to cover using Unit Tests

### Out of process component tests
* Developers should mock all external dependencies except databases

### What to cover
* Developers should cover one successful test case from API to database for each new endpoint

### When to run
* Should run in PR gate before merge

## Contract tests
* Developers should cover internal microservices http clients [pact](https://github.com/pact-foundation/pact-net)
* Developers should cover events contracts (library not found)

### When to run
* should run in independed pipeline when contract tests updated, all microservices should checkout to verify compatibility

## Resources
[microservice-testing](https://martinfowler.com/articles/microservice-testing/)
