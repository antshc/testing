# Microservice test strategy
* Test cases should be prepared by Automation QA but, Component and contract tests should be written by developers and reviewed by QAs.
* Component tests should simulate validation the same way a developer does during debugging, for example checking what data is sent in the event, what data is saved to the database, etc…
* E2E tests should be written by developers.

<img src="https://github.com/khdevnet/testing/blob/main/docs/test-pyramid.png" width="200">

## Unit tests
* Unit tests are optional
#### What to cover
* Cover complex logic
#### When to run
* Run in the pipeline
 
## Integration tests (test-only integration)
* Tests optional, could be written to run manually to check that integration works
#### What to cover
* Integration with external dependencies
* Repositories
* HTTP Clients
#### When to run
* Run locally, run manually during development, and remove after

## Component tests
#### Out-of-process component tests
In a microservices architecture, a component can be a single microservice itself. 
* Use [LightBDD testing framework](https://github.com/LightBDD/LightBDD) to create human-readable scenarios
* Developers should mock external services HTTP clients using WireMock .NET
* Use local database hosted in docker, test database migrations
* Run tests using Microsoft Test server [integration testing framework](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-7.0).
* Use in-memory mock for message bus.
  
#### What to cover
* Automate manual test scenarios

#### When to run
* Run in the pipeline
  
#### In-process component tests
* Developers should mock everything inside the controller
* Developers should cover authorization attributes and claims
* Developers should cover model validation and binding
 
#### What to cover
* Test request validations and Authentication attributes

#### When to run
* should run in the pipeline

## Contract tests
* Developers should cover internal microservices HTTP clients [pact](https://github.com/pact-foundation/pact-net)
* Developers should cover events contracts, [pact messaging](https://github.com/pact-foundation/pact-net/tree/master/samples/OrdersApi) can be used 

#### When to run
* should run in an independent pipeline, all microservices should check to verify compatibility

## E2E tests (integrations+configurations testing)
Test that the application started and run
* Test database health check
* Test External system HttpClient (call health checks) 
* Run a simple smoke test for the main flow in the microservice

## Resources
* [microservice-testing](https://martinfowler.com/articles/microservice-testing/)
* [Practical Event-Driven Microservices Architecture: Building Sustainable and Highly Scalable Event-Driven Microservices](https://www.oreilly.com/library/view/practical-event-driven-microservices/9781484274682/)
