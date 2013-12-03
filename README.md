# Unit Testing With Mockito

Draft

## Table Of Contents

* [Test Case Naming] (#test-case-naming)
* [Mock Declaration and Injection] (#mock-declaration-and-injection)
* [Mockito Matchers] (#mockito-matchers)
* [Mockito Verify] (#mockito-verify)
* [Equals Builder] (#equals-builder)
* [Test Data Builders] (#test-data-builders)

## Test Case Naming

* Use consistent naming across the project
* Don't use names starting with `test`
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
  // You have to use MockitoAnnotaion.init(this) to instantiate mock objects
  ```
  Use @InjectMock to do constructor or field based injection for the class under test
  ```Java
  @Mock
  FeedsProvider feedsProvider;
  
  @InjectMock
  ClassThatUsesFeedsProvider classThatUsesFeedsProvider;
  
  //This will set mocked feedsProvider instance for the classThatUsesFeedsProvider
  ```

## Mockito Matchers

* Try to avoid `any` matchers instead use mockito provided matchers or custom argument matchers.

  ```Java
  public List<Feed> fetchFeeds(String userId){
   FeedRequest feedRequest = new FeedRequest();
   feedRequest.setUserId(userId);
   return feedProvider.fetch(feedRequest);
  }
  ```
  
  In order to create a stub, the <i>fetch</i> method above you can use `refEq` matcher
  
  ```Java
  @Mock
  feedProvider
  String userId = 'userId';
  List<Feed> feeds = new ArrayList<Feed>();
  FeedRequest feedRequest = new FeedRequest();
  feedRequest.setUserId(userId);
  
  when(feedProvider.fetch(refEq(feedRequest))).thenReturn(feeds);
  
  // Don't use any(Feedrequest.class), refEq can be used when equals method is not implemented.
  ```



