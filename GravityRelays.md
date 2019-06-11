## Using Gravity Relays - Example 13

### The problem Gravity Relays only start to make sense when you take into
account the way the Gravity publishers and subscribers connect to each other.
Gravity uses a peer-to-peer architecture, so a connection is made between each
publisher and subscriber pair (there is no intermediary acting as a data
repository here).  This is a large part of why Gravity is capable of
transmitting large volumes of data quickly between nodes.  In some network
layouts though, this can have a negative impact on performance.  Consider the
case below where NodeA is on Host1, and NodeB and NodeC are on Host2:  

![Without Relay](https://github.com/aphysci/gravity/blob/master/test/examples/13-Relay/doc/WithoutRelay.jpg)

The two subscribers each make a connection to the publisher.  When NodeA
publishes a message, that data is sent directly to NodeB and NodeC.  As you can
see in the image above, this means that the data is sent across the network
twice.  If Host1 and Host2 are connected by a slow network link and/or if the
volume of data being published by NodeA is high, this can lead to performance
issues.

### The solution: Gravity Relay ###

With a Gravity Relay, you can ensure that data is only passed between hosts
once.  Below is the same as the above scenario with the addition of a Gravity
Relay:

![With Relay](https://github.com/aphysci/gravity/blob/master/test/examples/13-Relay/doc/WithRelay.jpg)

In this case, the data is only sent across the network once.  NodeB and NodeC
subscribe to and receive the data in exactly the same way that they normally
would.  They don't need to know or care that the data was relayed through
another component.  If they do care though, there is a flag on the
GravityDataProduct (GDP) that indicates that the data was forwarded via a
Relay.  More on this, and other details on how to use Relays below.

### The details: How to set this up ###

The Gravity Relay component was built so that there is very little work
required to include it in your GDP data flow.  If you look at the code in the
Relay example, you'll see that the code is basically identical to the code in
example 2 [Protobuf Data Products](UsingProtobufs).  The only difference is a
couple of strings, and the use of GravityDataProduct::isRelayedDataproduct() in
the subscriber for the Relay example.  The real difference in this example is
in the Gravity.ini file:

```ini
[general]
ServiceDirectoryURL="tcp://localhost:5555"
NoConfigServer=true
LocalLogLevel=DEBUG
ConsoleLogLevel=DEBUG

[Relay]
DataProductList="BasicCounterDataProduct"
#ProvideLocalOnly=false
```

The general section is the same as normal, although keep in mind there's really
not much advantage to using a Relay if all the components are on the same host
(localhost as configured here).  `localhost` is used here to make the example
easy to run on anyone's machine without changing the example code or config.

In the Relay section you need to specify all the GDP id's that should be
relayed using the DataProductList parameter.  Additional values would be
separated by commas (',').  The Relay will register itself as a relay via it's
own GravityNode.  This will notify the ServiceDirectory that anyone looking for
BasicCounterDataProduct should go to this Relay publisher instead of any other
publishers.  Starting a Gravity Relay and configuring it with this parameter is
all that is required to include a Relay in your data flow.

Optionally you can also use the ProvideLocalOnly flag (commented out here) to
indicate whether you want the output of this Relay to be kept from components
running on other machines (the default is True).  This means that the
ServiceDirectory will only point subscribers to this Relay as a publisher if
they reside on the same host.  Otherwise the ServiceDirectory will point the
subscriber to the original publisher(s) of that data. 

### Caveat

To make the Gravity Relay easy and seamless to use, we designed it to insert
itself into a Gravity data flow.  Because of this, it's important that the
Gravity Relay be shut down gracefully so that it can remove itself from this
data flow (that is, assuming you want to continue running the publishers and
subscribers of the data that the Relay was handling).  As mentioned above, once
a Relay is registered with the ServiceDirectory, the ServiceDirectory
determines whether or not a subscriber should get data from the original
source(s) or from the Relay.  If the Relay were to go away, the
ServiceDirectory needs to know this so that it can point all (new or old)
subscribers to the original source(s) of that data.  The Gravity Relay code is
written to respond to SIGINT and SIGTERM signals.  If it catches either of
these it will unregister itself and then terminate.  This will allow the other
components to function as though it were never there.

### Running the example

You can run this example as you would any of the others.  This run script
starts the ServiceDirectory, publisher, subscriber and Relay, and then kills
and restarts the Relay.  You can see in the output of the subscriber component
when it receives data from the Relay or from the original publisher.

