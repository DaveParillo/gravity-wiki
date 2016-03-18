
## The Gravity Config File - Example 8 ##

This example focuses on how to configure Gravity nodes using an .ini file.

### The Gravity.ini file ###

When a Gravity node starts, it will look in the current working directory for an .ini file with the same name as the given ComponentID, e.g. ConfigFileExample.ini.  If that isn't found it will look for a Gravity.ini file in the same location.  In this example we'll use this Gravity.ini file:

	#
	# This section is common to all components
	#
	[general]
	# Set the log level for logs written to the log file
	LocalLogLevel=debug

	# Set the log level for logs written to the console that started the component
	ConsoleLogLevel=debug

	# Without this, Gravity will spend a few seconds trying to retrieve
	# parameters from a remote ConfigServer.  
	NoConfigServer=true

	# The URL where the ServiceDirectory will be found.  This value is the same as the default.
	ServiceDirectoryURL="tcp://localhost:5555"

	#
	# This section is specific to the named component.  Any values set here will override values
	# set in the general section for this component.
	#
	[ConfigFileExample]
	LocalLogLevel=message
	Fs = 65000
	bin_ms = ( 1 / $Fs ) * 1000                              #bin size in ms
	bin_us = $bin_ms * 1000
	win_ms = 1000
	na_samples = 100

	#Parentheses are necessary for order of operations
	nsamps = $na_samples+(($Fs/1000)*$win_ms)
	nsamps_minus = $nsamps-$na_samples 
	operatorstr = "plus + minus - asterisk * divison / "    #quotes turn off arithmetic

We can see that this file has two sections (denoted by a name between [ ] brackets): general and ConfigFileExample.  The general section is applicable to any node that uses this .ini file.  This is a convenient location to set some common parameters, like log level and the ServiceDirectory URL.

The other sections in the file are component specific - the name of the section should always match a Component ID (note that this can be used for ServiceDirectory specific settings as well, as we'll see in the next example).  In this example, our component name is ConfigFileExample.  Any settings in the general section will be overridden by settings in this section for the ConfigFileExample component only.

Now we can look at how to retrieve those values in our code.

### The Configured code (ConfigFileExample) ###

Here we have a very simple component that reads and prints several configured values:

	//Initialize gravity, giving this node a componentID.
	GravityReturnCode ret = gn.init("ConfigFileExample");
	if (ret != GravityReturnCodes::SUCCESS)
	{
		Log::fatal("Could not initialize GravityNode, return code was %d", ret);
		exit(1);
	}

	Log::message("ServiceDirectoryURL = %s\n", gn.getStringParam( "ServiceDirectoryURL", "Not found" ).c_str() );
	Log::message("LocalLogLevel = %s\n", gn.getStringParam( "LocalLogLevel", "Not Found" ).c_str() );
	Log::message("bin_ms = %f\n", gn.getFloatParam( "bin_ms", 0. ) );
	Log::message("bin_us = %f\n", gn.getFloatParam( "bin_us", 0. ) );
	Log::message("win_ms = %d\n", gn.getIntParam( "win_ms", 0 ) );
	Log::message("na_samples = %d\n", gn.getIntParam( "na_samples", 0 ) );
	Log::message("nsamps = %d\n", gn.getIntParam( "nsamps", 0 ) );
	Log::message("nsamps_minus = %d\n", gn.getIntParam( "nsamps_minus", 0 ) );
	Log::message("operatorstr = %s\n", gn.getStringParam( "operatorstr", "Not found" ).c_str() );
	Log::message("default_value = %s\n", gn.getStringParam( "default_value", "Not found" ).c_str() );
	return 0;

The first thing to note here is that the GravityNode must be initialized before you can begin reading the configuration values.  This is how it knows which file and component section(s) to load.

If you run this example, you'll see the values that were loaded for each of the parameters printed to the console.  In some cases, no value is found, so the provided default value is used instead (e.g. default_value shows a value of "Not found").  In the case of LocalLogLevel, you can see that the printed value is "message" which matches the value set in the ConfigFileExample section, not the general section.

One last thing to note here is that parameter order does matter if you're using values set in this file.  For example, if you moved the "Fs = 65000" line to the end of the file, you would get an error saying that it could not find Fs when it tries to evaluate the line "bin_ms = ( 1 / $Fs ) * 1000"

