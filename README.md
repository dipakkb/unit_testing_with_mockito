# Unit Testing With Mockito

Draft

## Table Of Contents

* [Test Case Naming] (#test-case-naming)
* [Mock Declaration and Injection] (#mock-declaration-and-injection)
* [Mockito Matchers] (#mockito-matchers)
* [Mockito Verify] (#mockito-verify)
* [Mockito Spy] (#mockito-spy)
* [Equals Builder] (#equals-builder)
* [Test Data Builders] (#test-data-builders)

## Test Case Naming

* Use consistent naming across the project
* Don't use names starting with <i>test</i>
* Give meaningful names to your test case based on what behavior you are going to test

  ```Java
  //Test on Inventory Class
  @Test
  public void shouldBeOutOfStockWhenQuantityIsZero(){
  }
  ```

## Mock Declaration and Injection

* Use `@Mock` annotation to instantiate your mock objects.

  ```Java
  @Mock
  FeedsProvider feedsProvider;
  @Mock
  InventoryDAO inventoryDAO;
  // One way to intialize the mock objects is to use MockitoAnnotaion.init(this) before every test runs
  
  ```
  Use @InjectMock to do constructor or field based injection for the class under test
  ```Java
  @Mock
  FeedsProvider feedsProvider;
  
  @InjectMock
  ClassThatUsesFeedsProvider classThatUsesFeedsProvider;
  
  @Before public void initMocks(){MockitoAnnotaion.init(this);}
  //This will set mocked feedsProvider instance for the classThatUsesFeedsProvider
  ```

  [See More ...](http://docs.mockito.googlecode.com/hg/latest/org/mockito/InjectMocks.html)

## Mockito Matchers

* Don't use `any` matchers blindly instead use mockito provided matchers or custom argument matchers.

  ```Java
  public List<Feed> fetchFeeds(String userId){
   FeedRequest feedRequest = new FeedRequest();
   feedRequest.setUserId(userId);
   return feedProvider.fetch(feedRequest);
  }
  ```
  
  In order to create a stub the <i>fetch</i> method above, you can use `refEq` matcher
  
  ```Java
  @Mock
  feedProvider
  String userId = "userId";
  List<Feed> feeds = new ArrayList<Feed>();
  FeedRequest feedRequest = new FeedRequest();
  feedRequest.setUserId(userId);
  
  when(feedProvider.fetch(refEq(feedRequest))).thenReturn(feeds);
  
  // Don't use any(Feedrequest.class), refEq can be used when equals method is not implemented.
  ```
  
## Mockito Verify

* There is no need to do `verify` a method invocation if the returned value from the stub is calling another method. In this case verify is redundant. Because if the stub is not correct default return value is `null` and method call on that will fail which will fail the test case.

  ```Java
  stateMap = mapService.fetchDataByState("UP");
  stateMap.locate("ABC");
  ```
  There is no need to `verify` that fetchDataByState is invoked. If that does not happen then locate call will fail anyway.

* Always verify void method invocation
* If the returned value of a method is `null` and the subsequent code flow is dependent on `null` check then you should verify that method invocation happened. Because that will help in finding if the test is failing for the incorrect stub.

  ```Java
  stateMap = mapService.fetchDataByState("UP");
  if (null != stateMap){
    stateMap.locate("ABC");
  }
  else{
    //some other thing
  }
  ```
* In summary if you think only stub based approach is not sufficient for the test case then do a `verify`.
* Instead of using `verify` with `times(0)` use `never()` or use `verifyZeroInteractions`
  [See More ...](http://docs.mockito.googlecode.com/hg/latest/org/mockito/Mockito.html#never_verification)

## Mockito Spy

* [Read documentation here]("http://docs.mockito.googlecode.com/hg/latest/org/mockito/Mockito.html#spy")
* [Spying on superclass]("http://stackoverflow.com/questions/3467801/mockito-how-to-mock-only-the-call-of-a-method-of-the-superclass")

## Equals Builder
* You can use `EqualsBuilder.reflectionEquals` to comapre two objects in test instead of comapring field by field.
* [See More on EqualsBuilder](http://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/builder/EqualsBuilder.html)

## Test Data Builder

* Try to use test data builders as much as possible.
  ```Java
  Item = new Item();
  item.setQuantity(5);
  item.setSKU("123");
  ```
  
instead of creating items like this in every tests create a separate TestItemBuilder class. This just implements the Builder pattern to create an item.

  ```Java
  public class TestItemBuilder{
    //default values for the item object
    private int quantity = 2;
    private String sku = "100";
    
    public TestItemBuilder(){
    }
    
    public TestItemBuilder withQuantity(int quantity){
      this.quantity = quantity;
      return this;
    }
    
    public TestItemBuilder withSKU(String sku){
      this.sku = sku;
      return this;
    }
    
    public Item build(){
      Item item = new Item();
      item.setQuantity(this.quantity);
      item.setSKU(this.sku);
      return item;
    }
  }
  ```
  
  In your test code you can create item by using the above builder.
  
  ```Java
  Item item = new TestItemBuilder().withQuantity(5).withSKU("sku").build()
  ```

  





