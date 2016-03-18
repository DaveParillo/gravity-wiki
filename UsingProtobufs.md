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
