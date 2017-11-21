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
	do
	{
            ret = gn.request("Multiplication", //Service Name
                             multRequest1, //Request
                             requestor, //Object containing callback that will get the result.
                             "8 x 2"); //A string that identifies which request this is.
            // Service may not be registered yet
            if (ret != GravityReturnCodes::SUCCESS)
            {
                Log::warning("request to Multiplication service failed, retrying...");
                gravity::sleep(1000);
            }
	}
    while (ret != GravityReturnCodes::SUCCESS);

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

