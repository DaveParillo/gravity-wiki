## Using Gravity Domains - Example 9 ##

In all of the previous examples, we've started and accessed the ServiceDirectory using it's default URL: "tcp://localhost:5555".  This works fine, but there is another mechanism for finding the ServiceDirectory which allows a Gravity society to be much more robust: Gravity Domains.  When using a domain, the ServiceDirectory will broadcast on its availability local network using a configured domain name.  The following shows an example Gravity.ini file configured this way.

### Gravity.ini Domain Configuration ###

	[general]
	Domain="Example9"
	NoConfigServer=true
	LocalLogLevel=debug
	ConsoleLogLevel=debug

	[ServiceDirectory]
	ServiceDirectoryURL="tcp://localhost:5555"
	BroadcastEnabled=true
	ConsoleLogLevel=warn

In this example both ServiceDirectory and components (ProtobufDataProductPublisher and ProtobufDataProductSubscriber) read their configuration from this Gravity.ini file.  Because they don't have their own sections, the two components use the values in the general section.  The new parameter here, Domain, tells the components the name of the ServiceDirectory domain that they'll be using.  Because the Domain value isn't overridden in the ServiceDirectory section, this also tells the ServiceDirectory the name of the domain that it will advertise.

In the ServiceDirectory section, we still need to set the URL that will ultimately be used.  But only the ServiceDirectory will get this from the configuration file.  If this URL were set in the general section, the components would complain that they were given both a domain and a URL, and then would ignore the domain.  The other important setting here is BroadcastEnabled.  This tells the ServiceDirectory to send out broadcast messages advertising its availability.  For this example, we've also turned the ServiceDirectory console log level down.

### Code Updates ###

There is very little code that needs to change here.  We're basically using the same code from example 2, except that the following code was updated:

	// It's possible that the GravityNode fails to initialize when using Domains because
	// it hasn't heard from the ServiceDirectory.  Looping as we've done here ensures that
	// the component has connected to the ServiceDirectory before it continues to other tasks.
	while (grc != GravityReturnCodes::SUCCESS)
	{
	    Log::warning("Unable to connect to ServiceDirectory, will try again in 1 second...");
	    gravity::sleep(1000);
	    grc = gn.init("ProtobufGravityComponentID");
	}

In previous examples we haven't checked the return value of GravityNode::init, but real applications should use the above paradigm instead.  Because the component needs to locate the ServiceDirectory dynamically, it's possible that this won't occur on the first try.  This code ensures that you've established a connection to the ServiceDirectory before moving on to other tasks (e.g. registering and subscribing).  

