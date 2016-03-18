
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