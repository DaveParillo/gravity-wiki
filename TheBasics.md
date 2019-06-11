## The Basics - Example 1

Take a look at the first example (tests/examples/1 - BasicDataProduct).  This
is a very simple example, but highlights most of what is involved in using
Gravity's pub/sub functionality.

### Setting up the Publisher (BasicDataProductPublisher)

First you have the initialization code:

```cpp
using namespace gravity;

GravityNode gn;
const std::string dataProductID = "HelloWorldDataProduct";

//Initialize gravity, giving this node a componentID.  
GravityReturnCode ret = gn.init("SimpleGravityComponentID");
if (ret != GravityReturnCodes::SUCCESS)
{
  Log::fatal("Could not initialize GravityNode, return code was %d", ret);
  exit(1);
}
```

`GravityNode::init` primarily sets up the messaging framework and loads
configuration values.  One important configuration value is the location of the
ServiceDirectory.  The example here just assumes the default of
tcp://localhost:5555.  Like most of the GravityNode methods, it returns a
`GravityReturnCode` that will equal `GravityReturnCodes::SUCCESS` if there were
no issues.  Note that this init call may take a few seconds because the
GravityNode will check to see if it is part if a domain.  At this point the
GravityNode is ready for use.

Next we can register a data product description to allow us to publish data:
```cpp
//Register a data product
gn.registerDataProduct(
  //This identifies the Data Product to the service directory so that others can 
  // subscribe to it.  (See BasicDataProductSubscriber.cpp).  
  dataProductID,
  //Assign a transport type to the socket (almost always tcp, unless you are only 
  //using the gravity data product between two processes on the same computer).  							
  GravityTransportTypes::TCP
);
if (ret != GravityReturnCodes::SUCCESS)
{
  Log::fatal("Could not register data product with id %s, return code was %d", dataProductID.c_str(), ret);
  exit(1);
}	
```

The comments describe the arguments used here.  The purpose of this call is to
register a data product with the ServiceDirectory.  Once this is done, a
subscriber will be able to look up this data by using the data product id
("!HelloWorldDataProduct").  Note that none of the data published by this
component will ever be sent to the ServiceDirectory - that just acts as a
lookup mechanism that allows components to find each other.  Once a component
finds the source of the data it is interested in, it connects directly to that
component (using the transport specified in the registerDataProduct call).

Now that we're registered, we can start publishing data:

```cpp
bool quit = false; //TODO: set this when you want the program to quit if you need to clean up before exiting.  
int count = 1;
while(!quit)
{
  //Create a data product to send across the network of type "HelloWorldDataProduct"
  GravityDataProduct helloWorldDataProduct(dataProductID);
  //This is going to be a raw data product (ie not using protobufs).  
  char data[20];
  sprintf(data, "Hello World #%d", count++);
  helloWorldDataProduct.setData((void*)data, strlen(data));
  
  //Publish the  data product.  
  ret = gn.publish(helloWorldDataProduct);
  if (ret != GravityReturnCodes::SUCCESS)
  {
    Log::critical("Could not publish data product with id %s, return code was %d", dataProductID.c_str(), ret);
  }

  // sleep for 1 second.  
  gravity::sleep(1000);
}
```

A key aspect of Gravity is that the publisher has no idea whether anyone is
actually listening for this data.  If no components have subscribed to this
data product, then no data will actually be sent anywhere, so there will not be
any unnecessary network traffic.

### Setting up the Subscriber (BasicDataProductSubscriber)

Now that we have a component publishing data, we can set up another component
to subscribe to it.  First, we declare the subscriber class: 

```cpp
//Declare a class for receiving Published messages.  
class SimpleGravitySubscriber : public GravitySubscriber
{
  public:
    virtual void subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts);
};
```

We just need to have a class that extends the GravitySubscriber class, and
overrides the subscriptionFilled method.  The definition of this method is down
below.

```cpp
GravityNode gn;
const std::string dataProductID = "HelloWorldDataProduct";

//Initialize gravity, giving this node a componentID.
GravityReturnCode ret = gn.init("SimpleGravityComponentID2");
if (ret != GravityReturnCodes::SUCCESS)
{
  Log::fatal("Could not initialize GravityNode, return code is %d", ret);
  exit(1);
}
```

For the most part, the initialization here is identical to that of the
publisher, but it is a bit different.  This component uses a different id
string when it calls GravityNode::init.  The component id is used in a few
ways:
* The Gravity code will look for a configuration (.ini) file using this name
  (i.e. SimpleGravityComponentID2.ini) if it cannot find a file named
Gravity.ini.  
* If file logging is enabled (it is by default), the logs will be written to a
  file named <component id>.log.
* If the ConfigServer is used, the configuration for this component will be
  found by using its component id.

Now we can set up our subscription:

```cpp
//Subscribe a SimpleGravityHelloWorldSubscriber to the counter.  
SimpleGravitySubscriber hwSubscriber;
ret = gn.subscribe(dataProductID, hwSubscriber);
if (ret != GravityReturnCodes::SUCCESS)
{
  Log::critical("Could not subscribe to data product with id %s, return code was %d", dataProductID.c_str(), ret);
  exit(1);
}
```

We create an instance of SimpleGravitySubscriber, and use that in the call to
GravityNode::subscribe.  The dataProductID string here must exactly match the
string used to register the data product in the subscriber.

The end of our main function just cleans up (to the extent it can) what we're
doing:

```cpp
//Wait for us to exit (Ctrl-C or being killed).  
gn.waitForExit();

//Currently this will never be hit because we will have been killed (unfortunately).  
//But this shouldn't make a difference because the OS should close the socket and free all resources.  
gn.unsubscribe("HelloWorldDataProduct", hwSubscriber);
```

You don't need to use this waitForExit method, but if there's nothing else for
your main function to do (as is the case here), you use this to just have it
sit and wait until it's time to exit.  It's good to unsubscribe when possible,
but in many cases, the process will exit before it has a chance to. 

Lastly, we need to define the subscriptionFilled method:

```cpp
void SimpleGravitySubscriber::subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts)
{
  for (const auto& dp: dataProducts)
  {
    //Get a raw message
    int size = dp->getDataSize();
    char* message = new char[size+1];
    dp->getData(message, size);
    message[size] = 0; // null terminate

    //Output the message
    Log::warning("Got message: %s", message);
    //Don't forget to free the memory we allocated.
    delete message;
  }
}
```

Here we check the size of the data so that we can allocate space, and then we
read the string data.  Every string that is published by the publisher will be
received here in order.  Note that some strings (e.g. Hello World #0) may not
be received because the subscription has not been established yet.  In later
examples we'll look at more complex data types.

