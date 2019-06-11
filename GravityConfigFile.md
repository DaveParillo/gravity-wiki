
## The Gravity Config File - Example 8

This example focuses on how to configure Gravity nodes using an .ini file.

### The Gravity.ini file

When a Gravity node starts, it will look in the current working directory for
an .ini file with the same name as the given ComponentID, e.g. if the component
is named ConfigFileExample, then it will look for ConfigFileExample.ini.  If
that isn't found it will look for a file named Gravity.ini in the same
location.  In this example we'll use this Gravity.ini file:

```ini
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
#NoConfigServer=true

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

# This value not set here - value will be retrieved from the ConfigServer
#config_server_value = 1
# This value will override the value retrieved from the ConfigServer
config_server_overridden_value = 5

```


We can see that this file has two sections (denoted by a name between [ ]
brackets): general and ConfigFileExample.  The general section is applicable to
any component that uses this .ini file.  This is a convenient location to set
some common parameters, like log level and the ServiceDirectory URL.

The other sections in the file are component specific - the name of the section
should always match a Component ID (note that this can be used for
ServiceDirectory specific settings as well, as we'll see in the next example).
In this example, our component name is ConfigFileExample.  Any settings in the
general section will be overridden by settings in this section for the
ConfigFileExample component.

### The ConfigServer

Gravity also provides support for remote configuration via the ConfigServer.
The ConfigServer is just another Gravity component, but it is one that provides
a service used directly by each GravityNode.  This functionality can be
disabled by setting NoConfigServer=true in your local Gravity.ini file (the
line is commented out above so that we can use it in this example).  

All remote configuration values provided by the ConfigServer are maintained in
a file called config_file.ini collocated with the ConfigServer (note that,
because the ConfigServer is a normal Gravity component, _its_ configuration
values (ServiceDriectoryURL, logging, etc.) are stored in a Gravity.ini file as
above).  A config_file.ini has sections for components of interest, just like
Gravity.ini.  It can also have a general section as in the Gravity.ini file.
All values provided by the ConfigServer have lower precedence than local config
values.  In this example we'll use this config_file.ini file:

```ini
[ConfigFileExample]

config_server_value = 1
config_server_overridden_value = 2
```

Now we can look at how to retrieve all these values in our code.

### The Configured code (ConfigFileExample)

Here we have a very simple component that reads and prints several configured values:

```cpp

//Initialize gravity, giving this node a componentID.
GravityReturnCode ret = gn.init("ConfigFileExample");
if (ret != GravityReturnCodes::SUCCESS)
{
  Log::fatal("Could not initialize GravityNode, return code was %d", ret);
  exit(1);
}

// Note that all config keys are case insensitive
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

Log::message("config_server_value = %d\n", gn.getIntParam( "CONFIG_server_value", 0 ));
Log::message("config_server_overridden_value = %d\n", gn.getIntParam( "CONFIG_server_overridden_value", 0 ));
return 0;
```

The first thing to note here is that the GravityNode must be initialized before
you can begin reading the configuration values.  This is how it knows which
file and component section(s) to load.

If you run this example, you'll see the values that were loaded for each of the
parameters printed to the console.  In some cases, no value is found, so the
provided default value is used instead (e.g. default_value shows a value of
"Not found").  In the case of LocalLogLevel, you can see that the printed value
is "message" which matches the value set in the ConfigFileExample section, not
the general section.

There are two values provided by the ConfigServer here.  The first
(config_server_value) is printed as provided by the ConfigServer (1) because
there is no local config value.  The second (config_server_overridden_value) is
also in Gravity.ini, so that value (5) is printed instead.

One last thing to note here is that parameter order does matter if you're using
values set in this file.  For example, if you moved the `Fs = 65000` line to
the end of the file, you would get an error saying that it could not find Fs
when it tries to evaluate the line `bin_ms = ( 1 / $Fs ) * 1000`

