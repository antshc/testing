[[_TOC_]]
# Prerequisites
* [Theory](https://github.com/khdevnet/testing/blob/main/ContractTestsTheory.MD)
* [PactNet Github](https://github.com/pact-foundation/pact-net)
* [PactNet Videos](https://www.youtube.com/playlist?list=PLwy9Bnco-IpfZ72VQ7hce8GicVZs7nm0i)

# Consumer tests
[Full example](https://github.com/pact-foundation/pact-net/blob/master/samples/Messaging/Consumer.Tests/StockEventProcessorTests.cs)
## Message convention
* Microservice Should have its own copy of the message
* Message should contain only fields that it uses in the consumer microservice

## Contract schema name definition

```
MessagePact.V3("stockevent-appnameapi-consumer", "stockevent-appname2-producer", new PactConfig () {...})
```
### Consumer name structure
**stockevent** - event class name in lowercase
**appnameapi** - consumer application name
**consumer** - suffix

### Producer name structure
**stockevent** - event class name in lowercase
**appname2** - producer application name
**producer** - suffix

##Contracts definition folder
* Folder in the same level as we have src
```
 IMessagePactV3 v3 = MessagePact.V3("stockevent-appnameapi-consumer", "stockevent-appname2-producer", new PactConfig
            {
                PactDir = "../../../pacts/", // folder in the same level as we have src
            })
```
## Contract message schema test
### Test case name
```
.ExpectsToReceive("Receive a Stock Event")
```
**Receive a Stock Event** - will be used in the Producer contract test as scenario name ```scenarios.Add("Receive a Stock Event")```

### Message metadata
#### Topic/queue name
```
somestockqueue
```
#### Example
```
 .WithMetadata("topic", "somestockqueue")
```

### Message body content
#### Message type
```csharp
    public class StockEvent
    {
        public string Name { get; init; }

        public decimal Price { get; init; }

        public DateTimeOffset Timestamp { get; init; }
    }
```
#### Example
```csharp
 .WithJsonContent(Match.Type(new
    {
        Name = Match.Type("AAPL"),
        Price = Match.Decimal(1.23m),
        Timestamp = Match.Type(14.February(2022).At(13, 14, 15, 678))
     }))
```
## Test class naming conventions
```csharp
namespace Consumer.ContractTests.Messages
{
    public class StockEventContract
    {
        [Fact]
        public void When_new_stock_was_created_Expect_receive_stock_event()
        {
        }
    }
}
```
# Producer tests
[Full example](https://github.com/pact-foundation/pact-net/blob/master/samples/Messaging/Provider.Tests/StockEventGeneratorTests.cs)

## Message convention
* Microservice Should have its own copy of the message

## Consumer contracts location folder
```csharp
string pactPath = Path.Combine("..","etc..", "pacts"); // path to root of the project + pacts folder
this.verifier.WithDirectorySource(new DirectoryInfo(pactPath))
```

## Filter consumer contracts schema related to current producer
```csharp
 .MessagingProvider("stockevent-appname2-producer", defaultSettings)
```
**"stockevent-appname2-producer"** - name of the producer that was used in the consumer contract test
## Verify consumer test case
```csharp
.WithProviderMessages(scenarios =>
                 {
                     scenarios.Add("Receive a Stock Event",
                                   () => new StockEvent
                                   {
                                       Name = "AAPL",
                                       Price = 1.23m,
                                       Timestamp = new DateTimeOffset(2022, 2, 14, 13, 14, 15, TimeSpan.Zero)
                                   })
```
**Receive a Stock Event** - name of the test case that was used in the consumer test

## Test class naming convention
```csharp
namespace Provider.ProducerTests.Messages
{
    public class StockEventContract
    {
        [Fact]
        public void When_new_stock_was_created_Expect_sends_stock_event()
        {
        }
    }
}

```
# Match helpers
Use helpers to make validation of fields more strict
```csharp
public static class MatchCustom
{
    public static IMatcher NonEmptyString(dynamic example)
       => Match.Regex(example, @"^(?!\s*$).+");

    public static IMatcher Guid(dynamic example)
        => Match.Regex(example, @"^[{]?[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}[}]?$");

    public static IMatcher PhoneNumber(dynamic example)
       => Match.Regex(example, @"^\+[1-9]\d{1,14}$");

    public static IMatcher Email(dynamic example)
       => Match.Regex(example, @"^[a-zA-Z0-9][a-zA-Z0-9.!#$%&'*+-/=?^_`{|}~]*?[a-zA-Z0-9._-]?@[a-zA-Z0-9][a-zA-Z0-9._-]*?[a-zA-Z0-9]?\\.[a-zA-Z]{2,63}$");
}
```
