[[_TOC_]]

# Prequesities
* [What to test](https://www.sisense.com/blog/rest-api-testing-strategy-what-exactly-should-you-test/)
* [Martin Fowler Component tests](https://martinfowler.com/articles/microservice-testing/#testing-component-introduction)
* [Web Test Server](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-6.0)
* [Wiremock.net](https://github.com/WireMock-Net/WireMock.Net/wiki/Request-Matching)
* [Bogus](https://github.com/bchavez/Bogus)
* [gherkin cucumber Given When Then](https://cucumber.io/docs/gherkin/reference/)

# Main concepts
* Mock or replace external API dependencies using wiremock.net
* Run tests using  mocks in the docker container (database, blob storage, etc..)
* Run tests using mocks in the docker container
* Write tests for Controllers, IntegrationMessageHandlers
* Each test should create a unique entity with a unique id, we do operations on it, but do not clean it after the test is finished (will be removed during docker disposal)
* Test should be intended and run in parallel 
* Use CollectionFixture for databases (should limit the amount of simultaneously run containers, because of issues in azure pipelines)

# Conventions

## Project structure

```
|- Core # fremowork extensions
|-- TestWebApplicationFactory
|- Clients # RestEase HTTP Clients
|- IForgotPasswordClient
|- IRegistrationClient
|- Configurations
|-- AzureAppConfiguration.cs # App configuration settings
|-- TestingHostStartup.cs # Override for WebHost configuration
|- Features
|-- Api # Controller under test
|--- Registration.cs # Action under test HttpMethod + UrlPart in pascal Case
|--- Registration_steps.cs # Action under test HttpMethod + UrlPart in pascal Case 
|- IntegrationMessageHandlers # External message handlers
|-- CreateCustomerMessageHandler
|- BaseComponentTest.cs # base class for component
|- TestData # should be use to create reusable fake data
```

## Feature scenarios class
* Add external system mocks setup Given steps first in the scenario, after add customer data mocks steps
* Add external system mocks verify Then steps at the end of the scenario after the check of the response using the same order that was use in the Given setup mocks list
```csharp
[FeatureDescription(@"Customer can register in our system after phone verification")]
[Collection(nameof(MsSqlDbContainer))]
public class Registration : BaseComponentTest
{
    private readonly DbContainer _dbContainer;
    private readonly TestCustomer _testCustomer
    = new AutoFaker<TestCustomer>()
        .RuleFor(c => c.Id, f => f.Random.Guid())
        .RuleFor(c => c.Name, f => f.Name.FirstName())
        .Generate();

    public Registration(DbContainer dbContainer, ITestOutputHelper testOutputHelper)
        : base(dbContainer, testOutputHelper) => _dbContainer = dbContainer;

    [Scenario]
    public async Task Customer_registered_successfully()
    {
        await RunScenarioAsync<Registration_steps>(
             _ => _.Given_crm_proxy_api_add_endpoint_mock_exists(_testCustomer),
             _ => _.Given_CustomerRegisteredEvent_topic_mock_exists(),
             _ => _.Given_customer_with_id_ID_not_exist(_testCustomer.Id),
             _ => _.When_customer_start_registration(_testCustomer),
             _ => _.Then_response_is_ok(),
             _ => _.When_customer_start_phone_verification(_testCustomer.Phone.Value),
             _ => _.Then_response_is_ok(),
             _ => _.When_customer_verify_phone_number_sucessfully(_testCustomer.Phone.Value),
             _ => _.Then_response_is_ok(),
             _ => _.Then_CustomerRegisteredEvent_is_sent(_testCustomer),
             _ => _.Then_customer_created_in_crm(_testCustomer),
             _ => _.Then_user_created_in_identity_server(_testCustomer),
             _ => _.Then_customer_fields_in_database_matched(_testCustomer)
          );
    }

    protected override object CreateFeatureContext(IDependencyResolver x)
               => new Registration_steps(App, RegistrationClient);
}
```
## Reuse steps in the scenario
* should be used for sharing steps for similar scenarios
* all steps will be displayed in the output result and report
```csharp
    public Task<CompositeStep> When_customer_do_registration_with_kyc_verification_scenario(TestCustomer testCustomer)
    {
        return Task.FromResult(CompositeStep.DefineNew()
            .WithContext(ctx => this)
            .AddAsyncSteps(
                 _ => _.Given_crm_proxy_api_add_endpoint_mock_exists(testCustomer),
                 _ => _.Given_CustomerRegisteredEvent_topic_mock_exists(),
                 _ => _.Given_sms_provider_api_send_endpoint_mock_exists(testCustomer),
                 _ => _.Given_customer_with_id_ID_not_exist(testCustomer.Id),
                 _ => _.When_kyc_customer_start_registration(testCustomer),
                 _ => _.Then_response_is_ok(),
                 _ => _.When_customer_start_phone_verification(testCustomer.Phone.Value),
                 _ => _.Then_response_is_ok(),
                 _ => _.When_customer_verify_phone_number_sucessfully(testCustomer.Phone.Value),
                 _ => _.Then_response_is_ok())
            .Build());
    }
```
## Precondition steps in the scenario
* should be used to set up precondition for the scenario
* parent step will be displayed in the output result and report
```csharp
    public async Task Given_customer_without_kyc_verification_precondition(TestCustomer testCustomer)
    {
        await Given_crm_proxy_api_add_endpoint_mock_exists(testCustomer);
        await Given_CustomerRegisteredEvent_topic_mock_exists();
        await Given_sms_provider_api_send_endpoint_mock_exists(testCustomer);
        await Given_customer_with_id_ID_not_exist(testCustomer.Id);
        await When_customer_start_registration(testCustomer);
        await When_customer_start_phone_verification(testCustomer.Phone.Value);
        await When_customer_verify_phone_number_sucessfully(testCustomer.Phone.Value);
    }
```

## BaseComponentTest
* should be used to initialize TestWebApplicationFactory
* TestWebApplicationFactory will be initialized for each test (will reset mock setups)
* should be used to create HTTP clients
* should be used to set up behavior for global mocks
* scenario-specific setup should be done in steps

```csharp
public abstract class BaseComponentTest :
    FeatureFixture,
    IDisposable
{
    public readonly Guid CustomerId = Guid.Parse("1d925d66-e115-4d54-84cb-91396d8fb781");

    protected BaseComponentTest(DbContainer dbContainer, ITestOutputHelper? testOutputHelper = null)
    {
        App = new TestWebApplicationFactory(dbContainer, testOutputHelper);

        App.ContactRepositoryMock
            .Setup(u => u.Get(It.IsAny<ContactSpecification>()))
            .Returns(Task.FromResult(new ContactEntity { CustomerId = CustomerId }));

        App.ContactVerificationMock
            .Setup(c => c.Verify(It.IsAny<VerificationPhoneWithCode>(), It.IsAny<CancellationToken>()))
            .ReturnsAsync(new VerificationResult(true, 0));

        App.IdentityServerClientMock
            .Setup(i => i.LoginOnBehalfOfUser(It.IsAny<UserCredentials>(), null, null, It.IsAny<CancellationToken>()))
            .Returns(Task.FromResult(
                          new AccessAndRefreshToken(FakeToken.WithJwtId(CustomerId).ToString().Split("Bearer ")[1],
                             "refreshToken")));

        Client = App.CreateClientWithLogger();

        CustomersController = ControllerFor<ICustomersController>();
        DbContainer = dbContainer;
    }

    public DbContainer DbContainer { get; }

    protected TestWebApplicationFactory App { get; init; }

    protected HttpClient Client { get; init; }

    protected ICustomersController CustomersController { get; init; }
...
}
```
## TestWebApplicationFactory
* should be used to create a test web application
* Mocks initialization should be there (setup behavior in tests)
```csharp
public class TestWebApplicationFactory : WebApplicationFactory<Program>
{
    private const string EnvironmentName = "Local";
    private readonly DbContainer _dbContainer;
    private readonly ITestOutputHelper? _testOutputHelper;

    public TestWebApplicationFactory(
        DbContainer dbContainer,
        ITestOutputHelper? testOutputHelper = null)
    {
        WireMockServer = WireMockServer.Start();
        _dbContainer = dbContainer;
        _testOutputHelper = testOutputHelper;
    }

    public WireMockServer WireMockServer { get; init; }

    public Mock<IBus> BusMock { get; init; } = new Mock<IBus>();

    public Mock<IGuidProvider> GuidProviderMock { get; init; } = new Mock<IGuidProvider>();
    // ...

    public HttpClient CreateClientWithLogger() => CreateDefaultClient(new StepHttpLoggingHandler(new TestLogger<StepHttpLoggingHandler>()));

    protected override void Dispose(bool disposing)
    {
        base.Dispose(disposing);
        WireMockServer.Dispose();
    }

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        base.ConfigureWebHost(builder);
        SetTestOutputLogger(builder, _testOutputHelper);

        builder.UseEnvironment(EnvironmentName);
        builder.UseContentRoot(Directory.GetCurrentDirectory());
        builder.ConfigureAppConfiguration(app =>
        {
            var testAppSettings = new List<KeyValuePair<string, string>>() {
                        new("SqlDb:Customer:ConnectionString", _dbContainer.DbConnectionString),
                        new("CustomerApi:CrmClient:BaseUrl", WireMockServer.Url),
                        new("CustomerApi:CrmClient:Enabled", "true"),
                    };

            app.AddInMemoryCollection(new Dictionary<string, string>(testAppSettings));
        });
        builder.ConfigureTestServices(services =>
        {
            services.RemoveAll<IDistributedCache>();
            services.AddDistributedMemoryCache();

            services.Swap(CustomerMetadataEnricherMock.Object);
            services.Swap(ContactVerificationMock.Object);
            // ... other Swap
            services.AddMockAuthentication();
        });
    }
```
## Mocked class verification
### Setup
```csharp
 IdentityServerClientMock.Setup(i => i.LoginOnBehalfOfUser(It.IsAny<UserCredentials>()).Returns(Task.Completed)
```  
### Verify
* use It.Is<CustomerRegisteredEvent>() with strict conditions, for random data use conditions (<>) CusotmerId > 0, id != Guid.Empty 
```csharp
 IdentityServerClientMock.Verify(i => i.LoginOnBehalfOfUser(It.Is<UserCredentials>()).Returns(Task.Completed)
```  

## Service bus message verification
### Setup

```csharp
    public Task Given_CustomerRegisteredEvent_topic_mock_exists()
    {
        _capturedRegisteredEventMessages = new List<CustomerRegisteredEvent>();
        _busMock.Setup(x => x.Publish(Capture.In(_capturedRegisteredEventMessages), It.IsAny<string>(), It.IsAny<CancellationToken>()));

        return Task.CompletedTask;
    } 
```
### Verify
* use It.Is<CustomerRegisteredEvent>() with strict conditions, for random data use conditions (<>) CusotmerId > 0, id != Guid.Empty 
* log captured events in light bdd comments
```csharp
    public Task Then_CustomerRegisteredEvent_is_sent(TestCustomer testCustomer)
    {
        _capturedRegisteredEventMessages.Log(); // add light bdd comments

        _busMock.Verify(
            x => x.Publish(
                ValidateCustomerRegisteredEvent(testCustomer),
                It.IsAny<string>(),
                It.IsAny<CancellationToken>()),
            Times.Once());

        return Task.CompletedTask;
    }
```

## External API request verification (Wiremock.net)
### Setup
* setup in scenario step
* setup full request, make it strict as possible
```csharp
    public Task Given_crm_proxy_api_update_endpoint_mock_exists(TestCustomer testCustomer)
    {
        _wireMockServer.Given(Request.Create()
                               .UsingPut()
                               .WithPath(CrmCustomerUpdateEndpoint)
                               .WithBody((body) => VerifyCrmRequestBody(testCustomer, body)))
                       .RespondWith(WireMock.ResponseBuilders.Response.Create().WithStatusCode(HttpStatusCode.OK));

        return Task.CompletedTask;
    }

```

### Verify
* verify that request was sent
* reuse condition what was use on setup
```csharp
  public Task Then_customer_updated_in_crm(TestCustomer testCustomer)
    {
        _wireMockServer.FindLogEntries(Request.Create()
                               .UsingPut()
                               .WithPath(CrmCustomerUpdateEndpoint)
                               .WithBody((body) => VerifyCrmRequestBody(testCustomer, body)))
                       .Should()
                       .NotBeEmpty();

        return Task.CompletedTask;
    }
```

## TestData
* Customers with suffix **Registered** will be existing customers who were created during the database initialization and could be used to test read logic or scenarios where customers predefined, will contain static data
* customers with suffix **Random** will be use to generate random customer, will contain random data

```csharp
internal static class TestCustomers
{
    public static TestCustomer RegisteredJohn{ get; } = GenerateRandomCustomer();

    public static TestCustomer RegisteredDoe { get; } = GenerateRandomCustomer();

    public static TestCustomer RandomCustomer1 { get; } = GenerateRandomCustomer();

    public static TestCustomer RandomCustomer2 { get; } = GenerateRandomCustomer();

    public static TestCustomer CustomerEmailNotVerified { get; } = GenerateRandomCustomer(emailVerified: false);

    private static TestCustomer GenerateRandomCustomer(bool phoneVerified = true, bool emailVerified = true)
        => new Faker<TestCustomer>()
            .CustomInstantiator(f => new TestCustomer(
                f.Random.Guid(),
                f.PickRandom<CustomerType>(),
                f.Name.FirstName(),
                new TestCustomerContact(phoneVerified, RandomHelper.RandomPhoneNumber()),
                new TestCustomerContact(emailVerified, f.Internet.Email()),
                f.Address.FullAddress(),
                1,
                true))
            .Generate();
}

public record TestCustomer(
    Guid Id,
    CustomerType CustomerType,
    string Name,
    TestCustomerContact Phone,
    TestCustomerContact Email,
    string Location,
    int? TrafficSourceId,
    bool PersonalDataProcessing);

public record TestCustomerContact(bool IsVerified, string Value);
```

## Faker
### Option Use RuleFor
the purpose is to init properties
* better visibility about which field setup
* fields could be setup in random order

```csharp
new Faker<TestCustomer>()
            .RuleFor(fake => fake.Id, fake => fake.Random.Guid())
            .RuleFor(fake => fake.Name, fake => fake.Name.FirstName())
            .RuleFor(fake => fake.Email, fake => new TestCustomerContact(emailVerified, fake.Internet.Email()))
            .RuleFor(fake => fake.Location, fake => fake.Address.FullAddress())
            .RuleFor(fake => fake.TrafficSourceId, fake => 1)
            .RuleFor(fake => fake.PersonalDataProcessing, fake => true)
            .RuleFor(fake => fake.Phone, fake => new TestCustomerContact(phoneVerified, RandomHelper.RandomPhoneNumber()))
            .Generate();
```

### Option use CustomInstantiator
the purpose of this method is to allow you to configure how the instance is created (when init only through constructor or partially from the constructor)
cons of using it everywhere:
* less readable code, not clear to what fields setup
* should follow the order in the constructor 
* optional parameters should be moved to the end
```csharp
new Faker<TestCustomer>()
            .CustomInstantiator(f => new TestCustomer(
                f.Random.Guid(),
                f.PickRandom<CustomerType>(),
                f.Name.FirstName(),
                new TestCustomerContact(phoneVerified, RandomHelper.RandomPhoneNumber()),
                new TestCustomerContact(emailVerified, f.Internet.Email()),
                f.Address.FullAddress(),
                1,
                true))
            .Generate();

// Alternative, to improve readability is to use field names
new Faker<TestCustomer>()
            .CustomInstantiator(f =>
                new TestCustomer(
                    Id: f.Random.Guid(),
                    CustomerType: f.PickRandom<CustomerType>(),
                    Name: f.Name.FirstName(),
                    Phone: new TestCustomerContact(phoneVerified, RandomHelper.RandomPhoneNumber()),
                    Email: new TestCustomerContact(emailVerified, f.Internet.Email()),
                    Location: f.Address.FullAddress(),
                    TrafficSourceId: 1,
                    PersonalDataProcessing: true))
            .Generate();
```
## Light Bdd Table.For() TBDD
Note: do not display in azure devops test case, need extension of Test case importer 
```csharp
 public Task Given_crm_proxy_api_add_endpoint_mock_exists(InputTable<TestCustomer> testCustomer)
    {}
```
# Mocks
### Mock identity server
```csharp
// Replace IndetyServer Authentication with mock auth
services.AddMockAuthentication(JwtBearerDefaults.AuthenticationScheme);
// Mock [FromNameIdentifier] attribute
services.Swap<IBoundActionParametersService>(new BoundActionParametersService(typeof(Startup).Assembly));
```
### Mock Azure Service bus
```csharp
// Replace Azure service bus send requests classes
services.Swap(new Mock<ServiceBusClient>().Object);
services.Swap(new Mock<IBus>().Object);
services.Swap(new Mock<IMessageEntityCreationService>().Object);
```
### Mock data provider
```csharp
// Use In memory entity framework provider
services.SwapDbContext<PersonalAccountContext>(builder => builder.UseInMemoryDatabase(nameof(PersonalAccountContext), _inMemoryDatabaseRoot));
```
### Log to test output
```csharp
if (testOutputHelper is not null)
{
   services.AddLogging(logBuilder => logBuilder.AddXUnit(testOutputHelper));
}
```
