
## The Gravity Config File - Example 8 ##

This example focuses on how to configure Gravity nodes using an .ini file.

### The Gravity.ini file ###

When a Gravity node starts, it will try to find an .ini file with the same name as the given ComponentID, e.g. ConfigFileExample.ini.  If that isn't found it will look for a Gravity.ini file.  In this example we'll use this Gravity.ini file:

	#
	# This section is common to all components
	#
	[general]
	NoConfigServer=true
	LocalLogLevel=debug
	ConsoleLogLevel=debug
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
