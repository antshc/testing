[[_TOC_]]

# Prequesities
* [Martin Fowler Component tests](https://martinfowler.com/articles/microservice-testing/#testing-component-introduction)
* [Web Test Server](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-6.0)

# In process component tests concept
* Mock or replace external dependencies
* Run tests using an in-memory database or EF Core In-memory provider
* Write tests for Controllers, IntegrationMessageHandlers
* Test should be intended and run in parallel 

# Conventions

## Project structure

```
|- Clients # RestEase http Clients
|- Configurations
|-- TestingConfiguration.cs # App configuration settings
|-- TestingHostStartup.cs # Override for WebHost configuration
|- Controllers
|-- ControllerName # Controller under test
|--- GetProducts.cs # Action under test HttpMethod + UrlPart in pascal Case
|--- PostProducts.cs # Action under test HttpMethod + UrlPart in pascal Case 
|- IntegrationMessageHandlers # External message handlers
|-- CreateCustomerMessageHandler
|- BaseComponentTest.cs # base class for component
```

## Controller action test class structure
```
public class GetProducts : BaseComponentTest
{
    public GetVehiclesOnSale(WebApplicationFactory<Program> builder, ITestOutputHelper testOutputHelper)
        : base(builder, testOutputHelper)
    {
    }

    [Fact]
    public async Task When_has_subscriptions_Expect_subscriptions()
    {
        # Arrange/Actual/Assert
    }
```

## BaseComponentTest
* use IClassFixture<WebApplicationFactory<Program>>
* use extensions from ServicesTestFramework.WebAppTools.Extensions if possible

```
public abstract class BaseComponentTest : IClassFixture<WebApplicationFactory<Program>>
{
    private static InMemoryDatabaseRoot _inMemoryDatabaseRoot { get; } = new InMemoryDatabaseRoot();

    protected WebApplicationFactory<Program> ApiFactory { get; }

    protected readonly CustomerContext Db;

    protected BaseComponentTest(WebApplicationFactory<Program> webAppFactory, ITestOutputHelper testOutputHelper = null!)
    {
        ApiFactory = webAppFactory
            .WithWebHostBuilder(builder =>
            {
                builder.UseContentRoot(AppContext.BaseDirectory);
                builder.ConfigureTestServices(services =>
                {
                    // Replace IndetyServer Authentication with mock auth
                    services.AddMockAuthentication(JwtBearerDefaults.AuthenticationScheme);
                    // Replace Azure service bus send requests classes
                    services.Swap(new Mock<ServiceBusClient>().Object);
                    services.Swap(new Mock<IBus>().Object);
                    services.Swap(new Mock<IMessageEntityCreationService>().Object);
                    services.SwapDbContext<CustomerContext>(builder => builder.UseInMemoryDatabase(nameof(CustomerContext), _inMemoryDatabaseRoot));
                    if (testOutputHelper is not null)
                    {
                        services.AddLogging(logBuilder => logBuilder.AddXUnit(testOutputHelper));
                    }
                });
            });

        Db = ApiFactory.Services.GetScopedService<CustomerContext>();
    }

    protected TController CreateClient<TController>() where TController : class
    {
        var httpClient = ApiFactory.CreateClient();
        return httpClient.ClientFor<TController>();
    }
}
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
services.SwapDbContext<CustomerContext>(builder => builder.UseInMemoryDatabase(nameof(CustomerContext), _inMemoryDatabaseRoot));
```
### Log to test output
```csharp
if (testOutputHelper is not null)
{
   services.AddLogging(logBuilder => logBuilder.AddXUnit(testOutputHelper));
}
```

  
# Out of process component tests
* Use [WireMock.Net](https://github.com/WireMock-Net/WireMock.Net) to stub remote server
* Use Database in docker conteiner using [Test conteiners](https://github.com/HofmeisterAn/dotnet-testcontainers)
