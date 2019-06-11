## Java Pub/Sub - Example 6

This example is a reimplementation of example 2 in Java.  

### The Publisher (JavaProtobufDataProductPublisher)

The Java publisher looks very similar to the C++ version:  

```java
GravityNode gn = new GravityNode();
//Initialize gravity, giving this node a componentID.
gn.init("JavaProtobufDataProductPublisher");

gn.registerDataProduct(
  //This identifies the Data Product to the service directory so that others can 
  // subscribe to it.  (See BasicDataProductSubscriber.cpp).  
  "BasicCounterDataProduct", 
  //Assign a transport type to the socket (almost always tcp, unless you are only 
  //using the gravity data product between two processes on the same computer).  
  GravityTransportType.TCP);


boolean quit = false; //TODO: set this when you want the program to quit if you need to clean up before exiting.  
int count = 1;
while(!quit)
{
  //Create a data product to send across the network of type "BasicCounterDataProduct".   

  //In order to publish, the DataProductID must match one of the registered types.  
  GravityDataProduct counterDataProduct = new GravityDataProduct("BasicCounterDataProduct");

  //Initialize our message
  BasicCounterDataProduct.BasicCounterDataProductPB.Builder counterDataPB = 
      BasicCounterDataProduct.BasicCounterDataProductPB.newBuilder();
  counterDataPB.setCount(count);

  //Put message into data product
  counterDataProduct.setData(counterDataPB);

  //Publish the data product.  
  gn.publish(counterDataProduct);
  
  //Increment count
  count++;
  if(count > 50)
    count = 1;
  
  //Sleep for 1 second.
  Thread.sleep(1000); 
}		
```

The only real difference is the way protobufs are managed in Java.  You'll need
to create a Builder object rather than a protobuf instance.  Here, we don't use
an actual protobuf object at all - we just use a protobuf Builder class (a
class generated specifically for building BasicCounterDataProductPB objects).
We use the Builder object the same way we used the protobuf object in C++.  

### The Subscriber (JavaProtobufDataProductSubscriber)

As in the C++ version, we first need to define a subscription handler class:
```java
class SimpleGravityCounterSubscriber implements GravitySubscriber
{
  @Override
  public void subscriptionFilled(List<GravityDataProduct> dataProducts)
  {
    for (GravityDataProduct dataProduct : dataProducts) {
      //Get the protobuf object from the message
      BasicCounterDataProduct.BasicCounterDataProductPB.Builder counterDataPB = BasicCounterDataProduct.BasicCounterDataProductPB.newBuilder();
      if(!dataProduct.populateMessage(counterDataPB))
        Log.warning("Error Parsing Message");

      //Process the message
      Log.warning(String.format("Current Count: %d", counterDataPB.getCount()));
    }
  }
}
```

Again it's pretty similar to the C++ version, but there are a couple of
differences to note in addition to the protobuf usage we mentioned above.  One
is that GravitySubscriber is an interface, not an actual class, so you can
implement it rather than extend it.  Since Java only supports single
inheritance, this frees you up to extend anything else you'd like.  The other
difference is the way logs are written.  Gravity doesn't currently support
variable length argument lists in the Java API, so you'll need to build the
String yourself, and then pass that to the logger.

