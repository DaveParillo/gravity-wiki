## Using Gravity Domains - Example 9 ##

In all of the previous examples we've started and accessed the ServiceDirectory using its default URL, "tcp://localhost:5555".  This works fine, but there is another mechanism for finding the ServiceDirectory which allows Gravity to be much more robust: Gravity Domains.  When using a domain, the ServiceDirectory will broadcast its availability on the local network using a configured domain name.  The following shows an example Gravity.ini file configured this way.

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

In this example both the ServiceDirectory and the components (ProtobufDataProductPublisher and ProtobufDataProductSubscriber) read their configuration from this Gravity.ini file.  Because they don't have their own sections, the two components use the values in the general section.  The new parameter in this section, Domain, tells the components the name of the ServiceDirectory domain that they'll be using.  Because the Domain value isn't overridden in the ServiceDirectory section, this also tells the ServiceDirectory the name of the domain name that it will use to advertise itself.  You can use multiple domains on the same network as long as their names are unique.

In the ServiceDirectory section, we still need to set the URL that will ultimately be used.  Only the ServiceDirectory will get this value from the configuration file.  If this URL were set in the general section, the components would complain that they were given both a domain and a URL, and then would ignore the domain.  The other important setting here is BroadcastEnabled.  This tells the ServiceDirectory to send out broadcast messages advertising its availability.  For this example, we've also turned the ServiceDirectory console log level down.

### Domain Related Code Updates ###

There is very little code that needs to change here.  We're basically using the same code from example 2, except that the initialization code was updated:

	// It's possible that the GravityNode fails to initialize when using Domains because
	// it hasn't heard from the ServiceDirectory.  Looping as we've done here ensures that
	// the component has connected to the ServiceDirectory before it continues to other tasks.
	while (grc != GravityReturnCodes::SUCCESS)
	{
	    Log::warning("Unable to connect to ServiceDirectory, will try again in 1 second...");
	    gravity::sleep(1000);
	    grc = gn.init("ProtobufGravityComponentID");
	}

In previous examples we haven't checked the return value of GravityNode::init, but real applications should use the above paradigm instead.  Because the component needs to locate the ServiceDirectory dynamically, it's possible that they won't find it on the first try.  This code ensures that you've established a connection to the ServiceDirectory before moving on to other tasks (e.g. registering and subscribing).  

### Running with a Domain ###

The main point of this example is to demonstrate the ways in which you can recover from various failures when you are using a domain.  The run script provided with this example starts the ServiceDirectory, publisher and subscriber, lets them run for a few seconds, and then kills the ServiceDirectory and publisher.  After a few seconds it restarts both of these.  When the subscriber sees that the ServiceDirectory has restarted, it resubscribes to the data product that it wants to receive ("BasicCounterDataProduct").  The ServiceDirectory is then able to send it an updated list of publishers that are currently available.  If the restarted publisher hasn't yet hooked up to the ServiceDirectory, then the ServiceDirectory will send another update to the subscriber once it has.

Below is the output from running this script.  In this case, the run was killed a few seconds after the components reconnected.  

	[03/18/16 11:39:32.915095 ProtobufGravityComponentID-WARNING] Unable to connect to ServiceDirectory, will try again in 1 second...
	[03/18/16 11:39:33.898637 ProtobufGravityComponentID-DEBUG] Domain listener found update to our domain, orig SD start time = 0, new SD start time = 1458315573, SD url is now tcp://127.0.0.1:5555
	[03/18/16 11:39:33.916643 SimpleGravityComponentID2-DEBUG] Domain listener found update to our domain, orig SD start time = 0, new SD start time = 1458315573, SD url is now tcp://127.0.0.1:5555
	[03/18/16 11:39:33.921406 ProtobufGravityComponentID-DEBUG] Registered publisher at address: tcp://127.0.0.1:24004
	[03/18/16 11:39:33.950214 SimpleGravityComponentID2-WARNING] Unable to connect to ServiceDirectory, will try again in 1 second...
	[03/18/16 11:39:34.958364 SimpleGravityComponentID2-DEBUG] Found update to publishers list for data product BasicCounterDataProduct (domain:Example9)
	[03/18/16 11:39:34.958415 SimpleGravityComponentID2-DEBUG] subscriptionMap.count(key) = 1
	[03/18/16 11:39:34.958492 SimpleGravityComponentID2-WARNING] Current Count: 2
	[03/18/16 11:39:35.922838 SimpleGravityComponentID2-WARNING] Current Count: 3
	[03/18/16 11:39:36.924065 SimpleGravityComponentID2-WARNING] Current Count: 4
	[03/18/16 11:39:37.925284 SimpleGravityComponentID2-WARNING] Current Count: 5
	[03/18/16 11:39:38.925776 SimpleGravityComponentID2-WARNING] Current Count: 6
	[03/18/16 11:39:39.926643 SimpleGravityComponentID2-WARNING] Current Count: 7
	[03/18/16 11:39:40.927042 SimpleGravityComponentID2-WARNING] Current Count: 8
	[03/18/16 11:39:41.927436 SimpleGravityComponentID2-WARNING] Current Count: 9
	Killed ServiceDirectory and ProtobufDataProductPublisher
	./run.sh: line 55:  5374 Terminated              ./ProtobufDataProductPublisher
	./run.sh: line 55:  5393 Terminated              ServiceDirectory
	Restarting ServiceDirectory and ProtobufDataProductPublisher
	[03/18/16 11:39:52.899511 SimpleGravityComponentID2-DEBUG] Domain listener found update to our domain, orig SD start time = 1458315573, new SD start time = 1458315592, SD url is now tcp://127.0.0.1:5555
	[03/18/16 11:39:52.899544 SimpleGravityComponentID2-WARNING] ServiceDirectory restart detected, attempting to re-register...
	[03/18/16 11:39:53.403292 SimpleGravityComponentID2-MESSAGE] Successfully re-subscribed BasicCounterDataProduct
	[03/18/16 11:39:53.403346 SimpleGravityComponentID2-WARNING] Successfully re-registered with the ServiceDirectory
	[03/18/16 11:40:02.900915 ProtobufGravityComponentID-DEBUG] Domain listener found update to our domain, orig SD start time = 0, new SD start time = 1458315592, SD url is now tcp://127.0.0.1:5555
	[03/18/16 11:40:02.940492 SimpleGravityComponentID2-DEBUG] Found update to publishers list for data product BasicCounterDataProduct (domain:Example9)
	[03/18/16 11:40:02.940552 SimpleGravityComponentID2-DEBUG] subscriptionMap.count(key) = 1
	[03/18/16 11:40:02.940808 SimpleGravityComponentID2-DEBUG] Found update to publishers list for data product BasicCounterDataProduct (domain:Example9)
	[03/18/16 11:40:02.940908 SimpleGravityComponentID2-DEBUG] subscriptionMap.count(key) = 1
	[03/18/16 11:40:02.945761 ProtobufGravityComponentID-DEBUG] Registered publisher at address: tcp://127.0.0.1:24004
	[03/18/16 11:40:02.946247 SimpleGravityComponentID2-WARNING] Current Count: 1
	[03/18/16 11:40:03.947283 SimpleGravityComponentID2-WARNING] Current Count: 2
	[03/18/16 11:40:04.947875 SimpleGravityComponentID2-WARNING] Current Count: 3
	[03/18/16 11:40:05.948182 SimpleGravityComponentID2-WARNING] Current Count: 4
	[03/18/16 11:40:06.949531 SimpleGravityComponentID2-WARNING] Current Count: 5
