## C++ Odds and Ends - Example 5

This example highlights a few other features available in Gravity.  One is
support for retrieving configuration values from some configuration source.
Another feature is the ability to monitor whether a component is still running
by sending and listening for heartbeats.  The other feature is the ability to
send messages to other processes on the same host via ZeroMQ's IPC
(Inter-Process Communication) transport mechanism.  ZeroMQ doesn't currently
support this functionality on Windows, so we've disabled that part of the
example code on Windows platforms.  

### Retrieving Configuration Values

We'll go into more detail on using .ini files a little later, but here's a
quick primer for now.  GravityNode provides a few methods for retrieving
configuration values of different types.  In MiscComponent2 we have:

```cpp
// Get a parameter from either the Gravity.ini config file, 
// the MiscGravityComponentID.ini config file, or the config service.  
int interval = gn.getIntParam("HeartbeatInterval", // Param Name
                               500000);            // Default value.  

//Get another parameter from gravity.  
bool listen_for_heartbeat = gn.getBoolParam("HeartbeatListener", true);
```


All the configuration retrieval methods have this same argument structure, i.e.
a string that identities the name of the parameter, and an optional default
parameter.  If the parameter isn't found, then the default value is returned.
The order of priority of lookups (i.e. values from a higher priority source
overwrite values from a lower priority source) are:
* <component name>.ini (in this case "MiscGravityComponentID.ini") has highest
  priority
* Gravity.ini
* Config Service values listed specifically for this component (i.e. values in
  a [MiscGravityComponentID] section).
* Config Service values listed for general use (i.e. values in the [General]
  section) have lowest priority

This setup supports central control of all parameters, but allows for local
overrides.

### Working with Heartbeats

Initiating the heartbeat mechanism (MiscComponent1) is fairly trivial:

```cpp
GravityNode gn;
// Initialize gravity, giving this node a componentID.  
gn.init("MiscGravityComponentID1");

// Get a parameter from either the gravity.ini config file, 
// the MiscGravityComponentID.ini config file, or the config service.  
int interval = gn.getIntParam("HeartbeatInterval", 500000);

// Start a heartbeat that other components can listen to, telling them we're alive.  
gn.startHeartbeat(interval);
```

After initializing the GravityNode and determining the frequency that
heartbeats should be sent, we just tell the GravityNode to start the
heartbeats.  Now anyone can listen for these as long as they know the ID of
this component ("MiscGravityComponentID1" which is set in the call to init).
First we need to define the GravityHeartbeatListener class:

```cpp
class MiscHBListener : public GravityHeartbeatListener
{
  public:
    virtual void MissedHeartbeat(std::string dataProductID, 
                                 int microsecond_to_last_heartbeat, 
                                 std::string status);
    virtual void ReceivedHeartbeat(std::string dataProductID, 
                                   std::string status);
};

void MiscHBListener::MissedHeartbeat(std::string dataProductID, 
                                     int microsecond_to_last_heartbeat, 
                                     std::string status)
{
  Log::warning("Missed Heartbeat.  Last heartbeat %d microseconds ago.  ", microsecond_to_last_heartbeat);
}

void MiscHBListener::ReceivedHeartbeat(std::string dataProductID, 
                                       std::string status)
{
  Log::warning("Received Heartbeat.");
}

```

Then we can register our listener:

```cpp
// Get a parameter from either the gravity.ini config file, 
// the MiscGravityComponentID.ini config file, or the config service.  
int interval = gn.getIntParam("HeartbeatInterval", // Param Name
                               500000);            // Default value.  

// Get another parameter from gravity.  
bool listen_for_heartbeat = gn.getBoolParam("HeartbeatListener", true);
MiscHBListener hbl;

if(listen_for_heartbeat)
  gn.registerHeartbeatListener("MiscGravityComponentID1", interval, hbl);
```

We just need to know the name of the component we want to monitor
("MiscGravityComponentID1") and the expected interval of the heartbeats.  If we
stop receiving the heartbeats, then our
GravityHeartbeatListener::!MissedHeartbeat will be called. When a heartbeat is
received, then our GravityHeartbeatListener::!ReceivedHeartbeat will be called.

### Using the IPC Transport

Using IPC in Gravity is almost identical to using the normal TCP mechanism.
There are a couple caveats though:
* IPC isn't supported under Windows, so we've ifdef'd that functionality out of
  the examples for Windows platforms.
* IPC, of course, will only work between components on the same platform.  So
  be sure that the components will be collocated before using IPC.

We register and publish the data in MiscComponent1:

```cpp
#ifndef WIN32
  // Register a data product that is very fast only for components on this same machine.  
  gn.registerDataProduct("IPCDataProduct", GravityTransportTypes::IPC);
#endif
  
  // Let this exit after a few seconds so the heartbeat listener in 
  // MiscComponent2 will be notified when it goes away.
  int count = 5;
  while(count-- > 0)
  {
#ifndef WIN32
    // Publish the inter process communication data product.  
    GravityDataProduct ipcDataProduct("IPCDataProduct");
    
    std::string data = "hey!";
    ipcDataProduct.setData(data.c_str(), data.length());
    
    gn.publish(ipcDataProduct);
#endif
    gravity::sleep(1000);
  }
```

The only difference here is the "ipc" string we pass to registerDataProduct - the code looks exactly the same as it did in earlier examples.  

Subscribing to data also looks pretty much the same.  First we define a listener:

```cpp
// Declare a class for receiving Published messages.  
class MiscGravitySubscriber : public GravitySubscriber
{
  public:
    virtual void subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts);
};

void MiscGravitySubscriber::subscriptionFilled(const vector<shared_ptr<GravityDataProduct>>& dataProducts)
{
  for (const auto& dp: dataProducts)
  {
    //Get a raw message
    int size = dp->getDataSize();
    char* message = new char[size+1];
    dp->getData(message, size);
    message[size] = 0; // null terminate

    //Output the message
    Log::warning("Got message: %s", message);
    //Don't forget to free the memory we allocated.
    delete message;
  }
}
```

There are no differences here at all.  And subscribing to the data:

```cpp
// IPC isn't supported in Windows.
#ifndef WIN32
  //Subscribe to the IPC data product.  
  MiscGravitySubscriber ipcSubscriber;
  gn.subscribe("IPCDataProduct", ipcSubscriber);
#endif
```
also looks identical to what we have in the tcp case.

