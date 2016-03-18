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
