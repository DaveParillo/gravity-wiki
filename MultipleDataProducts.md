## Dealing With Multiple Data Products - Example 3

This example doesn't really break any new ground, but it does demonstrate a
couple of ways that you can handle receiving different kinds of data.  The
publisher doesn't do anything different here, so we'll skip that one.  The main
changes lie in the subscriber.

### The Subscriber (MultipleDataProductSubscriber)

Instead of a single subscription handle, we'll set up a few:

```cpp
//Declare a class for receiving Published messages.  
class SimpleGravitySubscriber : public GravitySubscriber
{
  public:
    virtual void subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts);
};

//Declare another class for receiving Published messages.  
class SimpleGravityCounterSubscriber : public GravitySubscriber
{
  public:
    SimpleGravityCounterSubscriber();
    virtual void subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts);
  private:
    int countTotals = 0;   //Declare a variable to keep track of the sum of all counts received.  
};

//Declare another class for receiving Published messages.  
class SimpleGravityHelloWorldSubscriber : public GravitySubscriber
{
  public:
    virtual void subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts);
};
```


There's not really much different going on here other than the fact that we
have multiple subscriber classes.  Now we can create instances of these classes
to handle our subscriptions:

```cpp
//Declare an object of type SimpleGravityCounterSubscriber (this also initilizes the total count to 0).  
SimpleGravityCounterSubscriber counterSubscriber;
//Subscribe a SimpleGravityCounterSubscriber to the counter data product.  
gn.subscribe("BasicCounterDataProduct", counterSubscriber); 

//Subscribe a SimpleGravityHelloWorldSubscriber to the counter.  
SimpleGravityHelloWorldSubscriber hwSubscriber;
gn.subscribe("HelloWorldDataProduct", hwSubscriber);

//Subscribe this guy to both data products.  
SimpleGravitySubscriber	allSubscriber;
gn.subscribe("BasicCounterDataProduct", allSubscriber);
gn.subscribe("HelloWorldDataProduct", allSubscriber);
```

Here we can see that we're using the same subscriber class to handle multiple
subscriptions, and that we subscribe multiple times to the same data product
with different subscription handlers.  Now to define the handlers:

```cpp
void SimpleGravityCounterSubscriber::subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts)
{
  for (const auto& dp: dataProducts)
  {
    //Get the protobuf object from the message
    BasicCounterDataProductPB counterDataPB;
    dp->populateMessage(counterDataPB);

    //Process the message
    countTotals = countTotals + counterDataPB.count();

    Log::warning("Subscriber 1: Sum of All Counts Received: %d", countTotals);
  }
}
	
void SimpleGravityHelloWorldSubscriber::subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts)
{
  for (const auto& dp: dataProducts)
  {
    //Get a raw message
    int size = dp->getDataSize();
    char* message = new char[size + 1];
    dp->getData(message, size);
    message[size] = 0; //Add NULL terminator

    //Output the message
    Log::warning("Subscriber 2: Got message: %s", message);
    //Don't forget to free the memory we allocated.
    delete message;
  }
}
	
void SimpleGravitySubscriber::subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts)
{
  for (const auto& dp: dataProducts)
  {
    //Process Message
    Log::warning("Subscriber 3: Got a %s data product", dp->getDataProductID().c_str());

    //Take different actions based on the message type.
    if(dp->getDataProductID() == "BasicCounterDataProduct")
    {
      BasicCounterDataProductPB counterDataPB;
      dp->populateMessage(counterDataPB);
      Log::warning("Subscriber 3: Current Count: %d", counterDataPB.count());
    }
  }
}
```

The first two handlers are pretty much cut and paste from the previous
examples.  The last one, which is used to subscribe to multiple data products,
has to check which kind of data product it's currently working with.  It does
this by checking the data product id since that's all it knows at this point.
Once it knows what type of data it has received, it can instantiate the right
kind of protobuf object, and populate it with the contents of the message.

