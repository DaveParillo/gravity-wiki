
# Getting Started #

This page discusses getting started with the Gravity software by using the examples provided in test/examples.  Before starting in with the code, make sure you've built all the binaries required to use the system.  See the [[GravitySetup]] page for info on doing that.

## Building and running the examples ##

Each of the examples can be built and run (not currently supported in Windows).  Here's a quick rundown of how to do that (again, this assumes you've gone through the steps in [[GravitySetup]]):
* open a terminal and cd to "<gravity root>/tests/examples/1 - BasicDataProduct"
* run the command "./run.sh"
* hit ctrl-c to kill the run

## The Examples ##
* [[TheBasics]]
* [[UsingProtobufs]]
* [[MultipleDataProducts]]
* [[GravityServices]]
* [[C++OddsAndEnds]]
* [[JavaPubSub]]
* [[JavaServices]]
* [[GravityConfigFile]]
* [[GravityDomains]]

## Using Protobufs - Example 2##

The second example is very similar to the first, with the only real difference being that we're using a Protobuf to hold the data that we're transmitting.  

### The Protobuf (BasicCounterDataProduct.proto) ###

Protobufs are data classes generated from a simple definition file.  The file for this example just contains:

	option optimize_for = SPEED;
	
	message BasicCounterDataProductPB
	{
		required int32 count = 1;
	}


The first line tells the code generator to optimize the generated code for speed rather than size.  Then we have the protobuf definition - a class named "BasicCounterDataProductPB" that contains a single integer field named "count".  the "= 1" at the end of the field definition is a field id (not a default value).  In most cases you'll just assign increasing values to these id's (i.e. the first will be set to 1, the second to 2, and so on).  For more detailed documentation on protobufs, read Google's full documentation here: https://developers.google.com/protocol-buffers/docs/proto.

Before it can be used, this .proto file must be turned into code (c++ in this case, later we'll turn it into java).  The Protobuf framework provides a protoc executable that performs the code generation.  Take a look at the Makefile for more details on how that's done.

### The Publisher (ProtobufDataProductPublisher) ###

The setup code here is identical to that used in the first example.  The only difference is when we create and set a value on the object we're sending:


		//Create a data product to send across the network of type "BasicCounterDataProduct".  
		GravityDataProduct counterDataProduct("BasicCounterDataProduct"); //In order to publish, the DataProductID must match one of the registered types.  

		//Initialize our message
		BasicCounterDataProductPB counterDataPB;
		counterDataPB.set_count(count);
		
		//Put message into data product
		counterDataProduct.setData(counterDataPB);


We create an instance of BasicCounterDataProductPB just as we would any normal class (because it is a normal class, just an automatically generated one).  We then use the set_count method to set the current count value.  Lastly we call GravityNode::setData.  Because we're using a protobuf here, we don't need to specify the data size - the Gravity framework can get the size from the protobuf.  

### The Subscriber (ProtobufDataProductSubscriber) ###

In the subscriber, we also see that there are very few changes.  The only differences occur in the definition of SimpleGravityCounterSubscriber::subscriptionFilled:

	void SimpleGravityCounterSubscriber::subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts)
	{
		for(std::vector< shared_ptr<GravityDataProduct> >::const_iterator i = dataProducts.begin();
				i != dataProducts.end(); i++)
		{
			//Get the protobuf object from the message
			BasicCounterDataProductPB counterDataPB;
			(*i)->populateMessage(counterDataPB);
	
			//Process the message
			Log::warning("Current Count: %d", counterDataPB.count());
		}
	}


Gravity doesn't know what kind of protobuf class you're working with, so it can't create an instance of the protobuf object itself.  You'll need to create the instance of the right type, and then Gravity will fill it in with data it's received from the publisher.  Once that's done, we can access any of the fields, in this case there's just "count".

## Dealing With Multiple Data Products - Example 3 ##

This example doesn't really break any new ground, but it does demonstrate a couple of ways that you can handle receiving different kinds of data.  The publisher doesn't do anything different here, so we'll skip that one.  The main changes lie in the subscriber.

### The Subscriber (MultipleDataProductSubscriber) ###

Instead of a single subscription handle, we'll set up a few:

	//Declare a class for receiving Published messages.  
	class SimpleGravitySubscriber : public GravitySubscriber
	{
	public:
		virtual void subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts);
	};
	
	//Declare another class for receiving Published messages.  
	class SimpleGravityCounterSubscriber : public GravitySubscriber
	{
	public:
		SimpleGravityCounterSubscriber();
		virtual void subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts);
	private:
		int countTotals; //Declare a variable to keep track of the sum of all counts received.  
	};
	
	//Define the constructor to SimpleGravityCounterSubscriber (called when a SimpleGravityCounterSubscriber object is created).  
	SimpleGravityCounterSubscriber::SimpleGravityCounterSubscriber()
	{
		countTotals = 0;
	}
	
	//Declare another class for receiving Published messages.  
	class SimpleGravityHelloWorldSubscriber : public GravitySubscriber
	{
	public:
		virtual void subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts);
	};


There's not really much different going on here other than the fact that we have multiple subscriber classes.  Now we can create instances of these classes to handle our subscriptions:

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


Here we can see that we're using the same subscriber class to handle multiple subscriptions, and that we subscribe multiple times to the same data product with different subscription handlers.  Now to define the handlers:

	void SimpleGravityCounterSubscriber::subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts)
	{
		for(std::vector< shared_ptr<GravityDataProduct> >::const_iterator i = dataProducts.begin();
				i != dataProducts.end(); i++)
		{
			//Get the protobuf object from the message
			BasicCounterDataProductPB counterDataPB;
			(*i)->populateMessage(counterDataPB);
	
			//Process the message
			countTotals = countTotals + counterDataPB.count();
	
			Log::warning("Subscriber 1: Sum of All Counts Received: %d", countTotals);
		}
	}
	
	void SimpleGravityHelloWorldSubscriber::subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts)
	{
		for(std::vector< shared_ptr<GravityDataProduct> >::const_iterator i = dataProducts.begin();
				i != dataProducts.end(); i++)
		{
			//Get a raw message
			int size = (*i)->getDataSize();
			char* message = new char[size + 1];
			(*i)->getData(message, size);
			message[size] = 0; //Add NULL terminator
	
			//Output the message
			Log::warning("Subscriber 2: Got message: %s", message);
			//Don't forget to free the memory we allocated.
			delete message;
		}
	}
	
	void SimpleGravitySubscriber::subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts)
	{
		for(std::vector< shared_ptr<GravityDataProduct> >::const_iterator i = dataProducts.begin();
				i != dataProducts.end(); i++)
		{
			//Process Message
			Log::warning("Subscriber 3: Got a %s data product", (*i)->getDataProductID().c_str());
	
			//Take different actions based on the message type.
			if((*i)->getDataProductID() == "BasicCounterDataProduct")
			{
				BasicCounterDataProductPB counterDataPB;
				(*i)->populateMessage(counterDataPB);
				Log::warning("Subscriber 3: Current Count: %d", counterDataPB.count());
			}
		}
	}


The first two handlers are pretty much cut and paste from the previous examples.  The last one, which is used to subscribe to multiple data products, has to check which kind of data product it's currently working with.  It does this by checking the data product id since that's all it knows at this point.  Once it knows what type of data it has received, it can instantiate the right kind of protobuf object, and populate it with the contents of the message.

## Using Services - Example 4 ##

The previous three examples all dealt with publishing and subscribing to data products.  Now we'll look at the service request and response functionality that Gravity provides.

### The Service Provider (BasicServiceProvider) ###

When a service provider registers with its GravityNode, it must provide a reference to an object that can handle service requests.  To set this, you create a class the extends GravityServiceProvider, and implement its request method:

	class MultiplicationServiceProvider : public GravityServiceProvider
	{
	public:
		virtual shared_ptr<GravityDataProduct> request(const std::string serviceID, const GravityDataProduct& dataProduct);
	};
	
	
	shared_ptr<GravityDataProduct> MultiplicationServiceProvider::request(const std::string serviceID, const GravityDataProduct& dataProduct)
	{
		//Just to be safe.  In theory this can never happen unless this class is registered with more than one serviceID types.  
		if(dataProduct.getDataProductID() != "Multiplication") {
			Log::critical("Request is not for multiplication!");
			return shared_ptr<GravityDataProduct>(new GravityDataProduct("BadRequest"));
		}
	
		//Get the parameters for this request.  
		MultiplicationOperandsPB params;
		dataProduct.populateMessage(params);
	
		Log::message("Request received: %d x %d", params.multiplicand_a(), params.multiplicand_b());
		
		//Do the calculation
		int result = params.multiplicand_a() * params.multiplicand_b();
		
		//Return the results to the requestor
		MultiplicationResultPB resultPB;
		resultPB.set_result(result);
		
		shared_ptr<GravityDataProduct> resultDP(new GravityDataProduct("MultiplicationResult"));
		resultDP->setData(resultPB);
	
		return resultDP;
	}


Similar to the subscriptionFilled method for a data subscriber, the request method provides a reference to a single GravityDataProduct that contains the information necessary to carry out the request.  In the example code, the service provider retrieves the MultiplicationOperandsPB from the request GravityDataProduct, and uses the information provided there to create the response.  It then bundles this up into a new MultiplicationResultPB protobuf object, wraps that in a new GravityDataProduct and returns it.  The Gravity framework then sends this response back to the requester.

Once the implementation for the service provider class is ready, it just needs to be registered:

	MultiplicationServiceProvider msp;
	gn.registerService(
						//This identifies the Service to the service directory so that others can 
						// make a request to it.  
						"Multiplication", 
						//Assign a transport type to the socket (almost always tcp, unless you are only 
						//using the gravity data product between two processes on the same computer).  
						GravityTransportTypes::TCP, 
						//Give an instance of the multiplication service class to be called when a request is made for multiplication.  
						msp);


### The Requester (BasicServiceRequestor) ###

There are two ways to make a service request via Gravity: asynchronously and synchronously.  This has no impact on the service provider - the code above works fine in both cases.  For asynchronous calls, we'll need to set up a class to handle the response to the request:

	//After multiplication is requested, this class may be called with the result.  
	class MultiplicationRequestor : public GravityRequestor
	{
	public:
		virtual void requestFilled(string serviceID, string requestID, const GravityDataProduct& response);
	};
	
	void MultiplicationRequestor::requestFilled(string serviceID, string requestID, const GravityDataProduct& response)
	{
		//Parse the message into a protobuf.  
		MultiplicationResultPB result;
		response.populateMessage(result);
		
		//Write the answer
		Log::message("Asynchronous response received: %s = %d", requestID.c_str(), result.result());
		
		gotAsyncMessage = true;
	}


In addition to the GravityDataProduct containing the response, requestFilled also provides the service and request id's.  The service id is simply the string used when making the request that identifies the service the request is made against.  Because the service provider may set the data product id on this GravityDataProduct to something unexpected, having the service id ensures that you can determine which type of request this call is in response to.  The request id is an arbitrary (and optional) string provided when the request is made to help client code tie a response to a particular request.  If you choose not to set it the request id in the request call, then it will be an empty string here.  

Now that we have the response handler ready, we can make the request:

	/////////////////////////////
	// Set up the first multiplication request
	MultiplicationRequestor requestor;

	GravityDataProduct multRequest1("Multiplication");
	MultiplicationOperandsPB params1;
	params1.set_multiplicand_a(8);
	params1.set_multiplicand_b(2);
	multRequest1.setData(params1);
	
	// Make an Asynchronous request for multiplication
	gn.request("Multiplication", //Service Name
				multRequest1, //Request
				requestor, //Object containing callback that will get the result.
				"8 x 2"); //A string that identifies which request this is.


When requestFilled is called, the request id will be set to "8 x 2".  

The other type of request is synchronous.  These calls don't need a response handler since the GravityDataProduct response is returned directly from the request call.  Of course, these calls need to be made with care since there may be no response, or it may be a long time until the response is sent back.

	//Set up the second multiplication request
	GravityDataProduct multRequest2("Multiplication");
	MultiplicationOperandsPB params2;
	params2.set_multiplicand_a(5);
	params2.set_multiplicand_b(7);
	multRequest2.setData(params2);
	
	//Make a Synchronous request for multiplication
	shared_ptr<GravityDataProduct> multSync = gn.request("Multiplication", //Service Name
								multRequest2, //Request
								1000); //Timeout in milliseconds


There's no need for a request id here, since there's no question as to which request the response is tied to.  Instead, a timeout can be specified so that the call won't block forever.  If no timeout is specified (or a value of -1 is used), then the call will only return when a response is received.

## C++ Odds and Ends - Example 5 ##

This example highlights a few other features available in Gravity.  One is support for retrieving configuration values from some configuration source.  Another feature is the ability to monitor whether a component is still running by sending and listening for heartbeats.  The other feature is the ability to send messages to other processes on the same host via ZeroMQ's IPC (Inter-Process Communication) transport mechanism.  ZeroMQ doesn't currently support this functionality on Windows, so we've disabled that part of the example code on Windows platforms.  

### Retrieving Configuration Values ###

!GravityNode provides a few methods for retrieving configuration values of different types.  In MiscComponent2 we have:

	//Get a parameter from either the Gravity.ini config file, the MiscGravityComponentID.ini config file, or the config service.  
	int interval = gn.getIntParam("HeartbeatInterval", //Param Name
					500000); //Default value.  
	
	//Get another parameter from gravity.  
	bool listen_for_heartbeat = gn.getBoolParam("HeartbeatListener", true);


All the configuration retrieval methods have this same argument structure, i.e. a string that identities the name of the parameter, and an optional default parameter.  If the parameter isn't found, then the default value is returned.  The order of priority of lookups (i.e. values from a higher priority source overwrite values from a lower priority source) are:
* <component name>.ini (in this case "MiscGravityComponentID.ini") has highest priority
* Gravity.ini
* Config Service values listed specifically for this component (i.e. values in a [MiscGravityComponentID] section).
* Config Service values listed for general use (i.e. values in the [General] section) have lowest priority

This setup supports central control of all parameters, but allows for local overrides.

### Working with Heartbeats ###

Initiating the heartbeat mechanism (MiscComponent1) is fairly trivial:

	GravityNode gn;
	//Initialize gravity, giving this node a componentID.  
	gn.init("MiscGravityComponentID1");

	//Get a parameter from either the gravity.ini config file, the MiscGravityComponentID.ini config file, or the config service.  
	int interval = gn.getIntParam("HeartbeatInterval", 500000);
	
	//Start a heartbeat that other components can listen to, telling them we're alive.  
	gn.startHeartbeat(interval);


After initializing the GravityNode and determining the frequency that heartbeats should be sent, we just tell the GravityNode to start the heartbeats.  Now anyone can listen for these as long as they know the ID of this component ("MiscGravityComponentID1" which is set in the call to init).  First we need to define the GravityHeartbeatListener class:

	class MiscHBListener : public GravityHeartbeatListener
	{
	public:
		virtual void MissedHeartbeat(std::string dataProductID, int microsecond_to_last_heartbeat, std::string status);
        	virtual void ReceivedHeartbeat(std::string dataProductID, std::string status);
	};
	
	void MiscHBListener::MissedHeartbeat(std::string dataProductID, int microsecond_to_last_heartbeat, std::string status)
	{
		Log::warning("Missed Heartbeat.  Last heartbeat %d microseconds ago.  ", microsecond_to_last_heartbeat);
	}
	
	void MiscHBListener::ReceivedHeartbeat(std::string dataProductID, std::string status)
	{
		Log::warning("Received Heartbeat.");
	}
	


Then we can register our listener:

	//Get a parameter from either the gravity.ini config file, the MiscGravityComponentID.ini config file, or the config service.  
	int interval = gn.getIntParam("HeartbeatInterval", //Param Name
					500000); //Default value.  
	
	//Get another parameter from gravity.  
	bool listen_for_heartbeat = gn.getBoolParam("HeartbeatListener", true);
	MiscHBListener hbl;
	
	if(listen_for_heartbeat)
		gn.registerHeartbeatListener("MiscGravityComponentID1", interval, hbl);


We just need to know the name of the component we want to monitor ("MiscGravityComponentID1") and the expected interval of the heartbeats.  If we stop receiving the heartbeats, then our GravityHeartbeatListener::!MissedHeartbeat will be called. When a heartbeat is received, then our GravityHeartbeatListener::!ReceivedHeartbeat will be called.

### Using the IPC Transport ###

Using IPC in Gravity is almost identical to using the normal TCP mechanism.  There are a couple caveats though:
* IPC isn't supported under Windows, so we've ifdef'd that functionality out of the examples for Windows platforms.
* IPC, of course, will only work between components on the same platform.  So be sure that the components will be collocated before using IPC.

We register and publish the data in MiscComponent1:

	#ifndef WIN32
		//Register a data product that is very fast only for components on this same machine.  
		gn.registerDataProduct("IPCDataProduct", GravityTransportTypes::IPC);
	#endif
		
		//Let this exit after a few seconds so the heartbeat listener in MiscComponent2 will be notified when it goes away.
		int count = 5;
		while(count-- > 0)
		{
	#ifndef WIN32
			//Publish the inter process communication data product.  
			GravityDataProduct ipcDataProduct("IPCDataProduct");
			
			std::string data = "hey!";
			ipcDataProduct.setData(data.c_str(), data.length());
			
			gn.publish(ipcDataProduct);
	#endif
			gravity::sleep(1000);
		}


The only difference here is the "icp" string we pass to registerDataProduct - the code looks exactly the same as it did in earlier examples.  

Subscribing to data also looks pretty much the same.  First we define a listener:

	//Declare a class for receiving Published messages.  
	class MiscGravitySubscriber : public GravitySubscriber
	{
	public:
		virtual void subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts);
	};
	
	void MiscGravitySubscriber::subscriptionFilled(const std::vector< shared_ptr<GravityDataProduct> >& dataProducts)
	{
		for(std::vector< shared_ptr<GravityDataProduct> >::const_iterator i = dataProducts.begin();
				i != dataProducts.end(); i++)
		{
			//Get a raw message
			int size = (*i)->getDataSize();
			char* message = new char[size+1];
			(*i)->getData(message, size);
			message[size] = 0; // null terminate
	
			//Output the message
			Log::warning("Got message: %s", message);
			//Don't forget to free the memory we allocated.
			delete message;
		}
	}


There are no differences here at all.  And subscribing to the data:

		// IPC isn't supported in Windows.
	#ifndef WIN32
		//Subscribe to the IPC data product.  
		MiscGravitySubscriber ipcSubscriber;
		gn.subscribe("IPCDataProduct", ipcSubscriber);
	#endif

also looks identical to what we have in the tcp case.

## Java Pub/Sub - Example 6 ##

This example is a reimplementation of example 2 in Java.  

### The Publisher (JavaProtobufDataProductPublisher) ###

The Java publisher looks very similar to the C++ version:  

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
			GravityDataProduct counterDataProduct = new GravityDataProduct("BasicCounterDataProduct"); //In order to publish, the DataProductID must match one of the registered types.  

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


The only real difference is the way protobufs are managed in Java.  You'll need to create a Builder object rather than a protobuf instance.  Here, we don't use an actual protobuf object at all - we just use a protobuf Builder class (a class generated specifically for building BasicCounterDataProductPB objects).  We use the Builder object the same way we used the protobuf object in C++.  

### The Subscriber (JavaProtobufDataProductSubscriber) ###

As in the C++ version, we first need to define a subscription handler class:

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


Again it's pretty similar to the C++ version, but there are a couple of differences to note in addition to the protobuf usage we mentioned above.  One is that GravitySubscriber is an interface, not an actual class, so you can implement it rather than extend it.  Since Java only supports single inheritance, this frees you up to extend anything else you'd like.  The other difference is the way logs are written.  Gravity doesn't currently support variable length argument lists in the Java API, so you'll need to build the String yourself, and then pass that to the logger.

## Using Services in Java - Example 7 ##

This example is a reimplementation of example 4 in Java.  There's really not much to say here, as in the previous example the code is very similar to the code in the C++ version (with the minor exception of protobuf usage).

### The Service Provider (BasicServiceProvider) ###

For the provider, we first set up the request listener:

	class MultiplicationServiceProvider implements GravityServiceProvider
	{
		public  MultiplicationServiceProvider()
		{
		}
	
		@Override
		public GravityDataProduct request(String serviceID, GravityDataProduct dataProduct)
		{
			//Just to be safe.  In theory this can never happen unless this class is registered with more than one serviceID types.
			if(!dataProduct.getDataProductID().equals("Multiplication")) {
				Log.critical(String.format("Request is not for %s, not Multiplication!", dataProduct.getDataProductID()));
				return new GravityDataProduct("BadRequest");
			}
	
			//Get the parameters for this request.
			Multiplication.MultiplicationOperandsPB.Builder params = Multiplication.MultiplicationOperandsPB.newBuilder();
			dataProduct.populateMessage(params);
	
			Log.warning(String.format("%d x %d", params.getMultiplicandA(), params.getMultiplicandB()));
	
			//Do the calculation
			int result = params.getMultiplicandA() * params.getMultiplicandB();
	
			//Return the results to the requestor
			Multiplication.MultiplicationResultPB.Builder resultPB = Multiplication.MultiplicationResultPB.newBuilder();
			resultPB.setResult(result);
	
			GravityDataProduct resultDP = new GravityDataProduct("MultiplicationResult");
			resultDP.setData(resultPB);
	
			return resultDP;
		}
	}


Once that's done, we can register it:

		MultiplicationServiceProvider msp = new MultiplicationServiceProvider();
		gn.registerService(
				//This identifies the Service to the service directory so that others can
				// make a request to it.
				"Multiplication",
				//Assign a transport type to the socket (almost always tcp, unless you are only
				//using the gravity data product between two processes on the same computer).
				GravityTransportType.TCP,
				//Give an instance of the multiplication service class to be called when a request is made for multiplication.
				msp);



### The Requester (BasicServiceRequestor) ###

And for the requester, we setup the response listener:

	//After multiplication is requested, this class may be called with the result.  
	class MultiplicationRequestor implements GravityRequestor
	{
		public void requestFilled(String serviceID, String requestID, GravityDataProduct response)
		{
			//Parse the message into a protobuf.  
			Multiplication.MultiplicationResultPB.Builder result = Multiplication.MultiplicationResultPB.newBuilder();
			response.populateMessage(result);
			
			//Write the answer
			Log.message(String.format("%s: %d", requestID, result.getResult()));
			
			gotAsyncMessage = true;
		}
	
		boolean gotAsyncMessage = false;
		public boolean gotMessage() { return gotAsyncMessage; }
	}

	
And then we make the asynchronous request that uses the listener:

		/////////////////////////////
		// Set up the first multiplication request
		MultiplicationRequestor requestor = new MultiplicationRequestor();
		
		GravityDataProduct multRequest1 = new GravityDataProduct("Multiplication");
		Multiplication.MultiplicationOperandsPB.Builder params1 = Multiplication.MultiplicationOperandsPB.newBuilder();
		params1.setMultiplicandA(8);
		params1.setMultiplicandB(2);
		multRequest1.setData(params1);
		
		// Make an Asyncronous request for multiplication
		gn.request("Multiplication", //Service Name
					multRequest1, //Request
					requestor, //Object containing callback that will get the result.  
					"8 x 2"); //A string that identifies which request this is.  


And lastly, we make the synchronous request against the same service:

		/////////////////////////////////////////
		//Set up the second multiplication request
		GravityDataProduct multRequest2 = new GravityDataProduct("Multiplication");
		Multiplication.MultiplicationOperandsPB.Builder params2 = Multiplication.MultiplicationOperandsPB.newBuilder();
		params2.setMultiplicandA(5);
		params2.setMultiplicandB(7);
		multRequest2.setData(params2);

		//Make a Synchronous request for multiplication
		GravityDataProduct multSync = gn.request("Multiplication", //Service Name
								multRequest2, //Request
								1000); //Timeout in milliseconds

		if(multSync == null)
		{
			Log.critical("Request Returned NULL!");
		}
		else
		{
			Multiplication.MultiplicationResultPB.Builder result = Multiplication.MultiplicationResultPB.newBuilder();
			multSync.populateMessage(result);
			
			Log.message(String.format("5 x 7 = %d", result.getResult()));
		}
