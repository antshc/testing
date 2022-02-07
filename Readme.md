# Microservice test theory
<img src="https://github.com/khdevnet/testing/blob/main/docs/test-pyramid.png" width="200">

## Unit tests
A unit test exercises the smallest piece of testable software in the application to determine whether it behaves as expected.

The size of the unit under test is not strictly defined, however unit tests are typically written at the class level or around a small group of related classes. 

**Sociable unit testing** focusses on testing the behaviour of modules by observing changes in their state. This treats the unit under test as a black box tested entirely through its interface.

**Solitary unit testing** looks at the interactions and collaborations between an object and its dependencies, which are replaced by test doubles.

<img src="https://github.com/khdevnet/testing/blob/main/docs/unit-tests.png" width="200">

## Integration tests
Integration tests collect modules together and test them as a subsystem in order to verify that they collaborate as intended to achieve some larger piece of behaviour.

In microservice architectures they are typically used to verify interactions between layers of integration code and the external components to which they are integrating.

Examples of the kinds of external components against which such integration tests can be useful include other microservices, data stores and caches.

<img src="https://github.com/khdevnet/testing/blob/main/docs/integration-test-boundary.png" width="200">

**Gateway integration tests** allow any protocol level errors such as missing HTTP headers, incorrect SSL handling or request/response body mismatches to be flushed out at the finest testing granularity possible.

**Persistence integration tests** provide assurances that the schema assumed by the code matches that available in the data store.

## Component tsts
In a microservice architecture, the components are the microservices themselves. By writing tests at this granularity, the contract of the API is driven through tests from the perspective of a consumer.

**In process component tests**
Replacing an external datastore with an in-memory implementation can provide significant test performance improvements. Whilst this excludes the real datastore from the test boundary, any persistence integration tests will provide sufficient coverage.
<img src="https://github.com/khdevnet/testing/blob/main/docs/in-process-component-tests.png" width="200">

**Out of process component tests**
As a result of the network interactions and use of a real datastore, test execution time is likely to increase. However, if the microservice has complex integration, persistence or startup logic, the out-of-process approach may be more appropriate.

External service stubs are available in a number of different varieties: some are dynamically programmed via an API, some use hand crafted fixture data and some use a record-replay mechanism capturing requests and responses to the real external service.

<img src="https://github.com/khdevnet/testing/blob/main/docs/out-of-process-component-tests.png" width="200">

## Contract tests
Whilst contract tests provide confidence for consumers of external services, they are even more valuable to the maintainers of those services. By receiving contract test suites from all consumers of a service, it is possible to make changes to that service safe in the knowledge that consumers won't be impacted.

<img src="https://github.com/khdevnet/testing/blob/main/docs/contract-tests.png" width="200">

## E2E tests
end-to-end tests is to verify that the system as a whole meets business goals 

If an external service is managed by a third party, it may not be possible to write end-to-end tests in a repeatable and side effect free manner. Similarly, some services may suffer from reliability problems that cause end-to-end tests to fail for reasons outside of the team's control.

In cases such as these, it can be beneficial to stub the external services, losing some end-to-end confidence but gaining stability in the test suite.

It good to have tests which will run using GUI but also support change the contract and run API tests directly to decrease verification speed.

<img src="https://github.com/khdevnet/testing/blob/main/docs/e2e-tests.png" width="200">

## Summary
<img src="https://github.com/khdevnet/testing/blob/main/docs/all-kinds-of-tests.png" width="200">


## Resources
[microservice-testing](https://martinfowler.com/articles/microservice-testing/)
