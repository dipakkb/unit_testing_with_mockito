# Tools & Techniques for effective unit testing

## Table Of Contents

* [Test Case Naming] (#test-case-naming)
* [Mock Declaration and Injection] (#mock-declaration-and-injection)
* [Mockito Matchers] (#mockito-matchers)
* [Mockito Verify] (#mockito-verify)
* [Mockito Spy] (#mockito-spy)
* [Equals Builder] (#equals-builder)
* [Test Data Builders] (#test-data-builders)
* [Building Mocks] (#building-mocks)

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
  YouTubeFeedsProvider youTubeFeedsProvider;
  @Mock
  InventoryDAO inventoryDAO;
  // One way to intialize the mock objects is to use MockitoAnnotaion.initMocks(this) before every test runs
  
  ```
  Use @InjectMock to do constructor or field based injection for the class under test
  ```Java
  @Mock
  YouTubeFeedsProvider youTubeFeedsProvider;
  
  @InjectMock
  ClassThatUsesYouTubeFeedsProvider classThatUsesYouTubeFeedsProvider;
  
  @Before public void initMocks(){MockitoAnnotaion.init(this);}
  //This will set mocked youTubeFeedsProvider instance for the classThatUsesYouTubeFeedsProvider
  ```

  Before proceeding to next please read [this](http://docs.mockito.googlecode.com/hg/latest/org/mockito/InjectMocks.html)

## Mockito Matchers

* Use appropriate mockito provided matchers or custom argument matchers that are very closely related to the arguments instead of using `any()` matchers. Using `any()` will lead to ineffective mock behaviour as it gives a more generic fault tolerant behaviour, what we need is fail fast.

  ```Java
  public List<Feed> fetchFeeds(String userId){
    FeedRequest feedRequest = new FeedRequest();
    feedRequest.setUserId(userId);
    return feedProvider.fetch(feedRequest);
  }
  ```
  
  In order to create a stub for the <i>fetch</i> method above, you can use `refEq` matcher
  
  ```Java
  @Mock
  feedProvider
  String userId = "userId";
  List<Feed> feeds = new ArrayList<Feed>();
  FeedRequest feedRequest = new FeedRequest();
  feedRequest.setUserId(userId);
  
  when(feedProvider.fetch(refEq(feedRequest))).thenReturn(feeds);
  
  // Don't use any(Feedrequest.class), refEq can be used when equals method is not implemented. It uses reflection to match the contents of the objects.
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
* instead of creating items like this in every tests,

  ```Java
  Item = new Item();
  item.setQuantity(5);
  item.setSKU("123");
  ```
  
 Create a separate TestItemBuilder class. This just implements the Builder pattern to create an item.

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
  Item item = new TestItemBuilder().withQuantity(5).withSKU("sku").build();
  ```
 
  
  [See More...](http://martinfowler.com/bliki/FluentInterface.html)


## Building Mocks
   
  Builders may also be used to hide the complexity of constructing mocks and setting expectations on them.
  This is especially useful when the object you're trying to build is not a POJO but something like an EJB.
  
  Instead of having the mocks and expectations in your test class (which is quite distracting):
  
  ```Java
  Item item = mock(Item.class);
  when(item.getQuantity()).thenReturn(5);
  when(item.getSKU()).thenReturn("sku");
  ```
  
  you could flesh out a builder for the same, and use:
  
  ```Java
  Item item = new ItemBuilder()
             .withQuantity(5)
             .withSKU("sku").build();
  ```
  
  The mocking and expectation-setting would be abstracted into a bukder:
  
  ```Java
  public class ItemBuilder{
    //default values for the item object
    private int quantity = 2;
    private String sku = "100";
    
    public ItemBuilder(){
    }
    
    public ItemBuilder withQuantity(int quantity){
      this.quantity = quantity;
      return this;
    }
    
    public ItemBuilder withSKU(String sku){
      this.sku = sku;
      return this;
    }
    
    public Item build(){
      Item item = mock(Item.class);
      when(item.getQuantity()).thenReturn(quantity);
      when(item.getSKU()).thenReturn(sku);
      return item;
    }
  }
  ```





