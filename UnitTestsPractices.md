# Testing
## Configure tools
### Install [FineCodeCoverage](https://github.com/FortuneN/FineCodeCoverage) extension to visual studio
### Add tool from view
<img src="https://user-images.githubusercontent.com/14298158/152655447-a501a796-1991-4b5a-8b7c-2d5e00ad311c.png"  width="300px"/>

### Validate code coverage of your tests
<img src="https://user-images.githubusercontent.com/14298158/152655574-7c151f01-3e6d-49aa-bac5-8d02933f3fb9.png" width="300px"/>

## Libraries
* Fake data generator [Bogus](https://github.com/bchavez/Bogus)
* Create mocks [Moq](https://github.com/moq/moq4)
* Clone objects [DeepCloner](https://github.com/force-net/DeepCloner)
* Comparer objects [Compare net objects](https://github.com/GregFinzer/Compare-Net-Objects/wiki/Getting-Started)

## Unit Tests
Unit tests is a type of software testing where individual units or components of a software are tested.
### Unit Testing Best Practices
* Unit Test cases should be independent. In case of any enhancements or change in requirements, unit test cases should not be affected.
* Test only one code at a time.
* Follow clear and consistent naming conventions for your unit tests
* In case of a change in code in any module, ensure there is a corresponding unit Test Case for the module, and the module passes the tests before changing the implementation
* Bugs identified during unit testing must be fixed before proceeding to the next phase in SDLC
* Adopt a “test as your code” approach. The more code you write without testing, the more paths you have to check for errors. Test should be develop in scope of Task
* In one test case only one aspect of the unit should be tested (try to avoid too many asserts).
* Use InternalsVisibleTo attribute to test internal classes.
* Consider to use various XUnit Inline Data attributes, group test cases using tests method name

### Test Project
* Test project should be closer to the project with logic
* Folder structure of your tests should reflect structure of projects and their classes under tests.

$$$$$Test Project structure image$$$$$$

### Unit Test Class
Use the following rules when creating or modifying unit test class.
#### Test Class Signature
* Name your newly created test class in format NameOfClassToBeTestedTests.
* Derive your unit test class from **BaseUnitTests**
```csharp
public class NameOfClassToBeTestedTests : BaseUnitTests
```

#### Constructor
* Use a Constructor to setup a state each time before any of your test in a fixture run
* Use constructor to initialize Mock Objects, use MockRepository from BaseUnitTests class
```csharp
public class GlassRoofPriceEngineTests : BaseUnitTests
{
    private Mock<IGlassRoofPriceEngineOptions> _glassRoofPriceEngineOptionsMock;

    public GlassRoofPriceEngineTests()
    {
        _glassRoofPriceEngineOptionsMock = _mockRepository.Create<IGlassRoofPriceEngineOptions>();
    }
    ...
```
#### Use FactAttribute to define single test.
```csharp 
[Fact]
public void NameOfMethodTest()
```
#### Use TheoryAttribute and InlineDataAttribute or MemberData to define different test data sets for your test method instead of creation several methods with same logic but different data
```csharp  
[Theory]
[InlineData("New Version","Old Version")]
[InlineData("Test Version","Old Version")]
public void NameOfMethodTest(string version, string deviceVersion)
```

### Test Method Signature
#### Name of the test method should follow convention:
<NameOfMethodToTest>_When_StateUnderTest_Expect_ExpectedBehavior
using this technique:
<NameOfMethodToTest>_When_AgeLessThan18_Expect_isAdultAsFalse
<NameOfMethodToTest>_When_InvalidAccount_Expect_WithdrawMoneyToFail
<NameOfMethodToTest>_When_MandatoryFieldsAreMissing_Expect_StudentAdmissionToFail

#### Use the AAA (Arrange, Act, Assert) pattern to writing unit tests for a method under test.
* The Arrange section of a unit test method initializes objects and sets the value of the data that is passed to the method under test.
* The Act section invokes the method under test with the arranged parameters.
* The Assert section verifies that the action of the method under test behaves as expected. For .NET, methods in the Assert class are often used for verification.

```csharp
public async Task GetInitialOffer_When_Enabled_And_Car_Without_GlassRoof_Expect_ApplyInternalDiscount()
{
    // Arrange
    _glassRoofPriceEngineOptionsMock.Setup(m => m.Enabled).Returns(true);

    // Actual
    var actual = await CreatePriceEngineSUT().GetInitialOffer(priceRequest);

    // Assert
    actual.Should().BeEquivalentTo(expected);
}

```
#### Use method to init test class
```csharp
// Use method to initialize Mock with setup
private GlassRoofPriceEngine CreatePriceEngineSUT()
    => new GlassRoofPriceEngine(_vcVarianceInfrastructureMock.Object, _glassRoofPriceEngineOptionsMock.Object);
```

#### Use test data generator
AutoFaker should be part of test case to make test more readable, this helps to remove global dependency and make it independed. Create test data method can be created inside test class
Set only data which require to setup behaviour
```csharp
var priceRequest = new AutoFaker<VcDiscountedPriceRequest>()
        .RuleFor(d => d.GlassRoof, false)
        .Generate();
```
#### Do not use It.IsAny(), instead use It.Is with equals
```csharp
_vcVarianceInfrastructureMock
    .Setup(m => m.getDiscountedPrice(It.Is<VcDiscountedPriceRequest>(it => AreEqual(it, getDiscountedPriceArg))))
    .ReturnsAsync(getDiscountedPriceReturn);
```
#### Make a Deep copy of the input argument and return argument, it will avoid compare by reference and test will fail when data changed
```csharp
var priceRequest = new AutoFaker<VcDiscountedPriceRequest>()
    .RuleFor(d => d.GlassRoof, false)
    .Generate();

var getDiscountedPriceArg = priceRequest.DeepClone();
var getDiscountedPriceReturn = new AutoFaker<VcDiscountedPriceResponse>().Generate();
var expected = getDiscountedPriceReturn.DeepClone();
_vcVarianceInfrastructureMock
    .Setup(m => m.getDiscountedPrice(It.Is<VcDiscountedPriceRequest>(it => AreEqual(it, getDiscountedPriceArg))))
    .ReturnsAsync(getDiscountedPriceReturn);
```

#### Full example
```csharp
public class GlassRoofPriceEngineTests : BaseUnitTests
{
    private const string MakeName = "MakeName";
    private const int GlassRoofDiscountPercentage = 5;
    private Mock<IGlassRoofPriceEngineOptions> _glassRoofPriceEngineOptionsMock;

    public GlassRoofPriceEngineTests()
    {
        _glassRoofPriceEngineOptionsMock = _mockRepository.Create<IGlassRoofPriceEngineOptions>();
    }

    [Fact]
    public async Task GetInitialOffer_When_Enabled_And_Request_Not_Has_GlassRoof_Expect_InternalOffer()
    {
        _glassRoofPriceEngineOptionsMock.Setup(m => m.Enabled).Returns(true);

        var priceRequest = new AutoFaker<VcDiscountedPriceRequest>()
            .RuleFor(d => d.GlassRoof, false)
            .Generate();

        var getDiscountedPriceArg= priceRequest.DeepClone();
        var getDiscountedPriceReturn = new AutoFaker<VcDiscountedPriceResponse>().Generate();

        var expected = getDiscountedPriceReturn.DeepClone();

        _vcVarianceInfrastructureMock
            .Setup(m => m.getDiscountedPrice(It.Is<VcDiscountedPriceRequest>(it => AreEqual(it, getDiscountedPriceArg))))
            .ReturnsAsync(getDiscountedPriceReturn);

        var actual = await CreatePriceEngineSUT().GetInitialOffer(priceRequest);

        actual.Should().BeEquivalentTo(expected);
    }

    private GlassRoofPriceEngine CreatePriceEngineSUT()
    {
        return new GlassRoofPriceEngine(_vcVarianceInfrastructureMock.Object, _glassRoofPriceEngineOptionsMock.Object);
    }
```




